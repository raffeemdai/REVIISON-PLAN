# Docling Shared Worker — Local PDF + LangChain Documents + S3 Assets

This example uses the local source PDF:

```python
from pathlib import Path

file_path = Path(
    "E:/Euron_Super_30_project/practice/source_docs_rag/doclaynet.pdf"
)
```

## Goal

Convert the PDF **once**, keep the native `DoclingDocument` in memory, and reuse it to create:

- Context-aware LangChain text chunks for RAG
- Extracted picture assets uploaded to Amazon S3
- Asset metadata that can later be enriched with a vision model

The same native document is passed to the text and image functions. Neither downstream function reconverts the original PDF.

## 1. Install dependencies

```bash
pip install docling langchain-core boto3
```

## 2. Imports and configuration

```python
from pathlib import Path
from typing import Any
import tempfile


import boto3
from docling.document_converter import DocumentConverter
from docling.chunking import HybridChunker
from langchain_core.documents import Document as LCDocument


# Local source PDF
file_path = Path(
    "E:/Euron_Super_30_project/practice/source_docs_rag/doclaynet.pdf"
)

# S3 configuration (use IAM role/profile in production; don't hardcode keys)
AWS_REGION = "us-east-1"
S3_BUCKET = "my-enterprise-rag-bucket"
S3_PREFIX = "processed/doclaynet"

s3 = boto3.client("s3", region_name=AWS_REGION)
```


## 3. Shared worker: one conversion

```python
class SharedDoclingWorker:
    def __init__(self, s3_client, bucket: str, s3_prefix: str = "processed"):
        self.s3 = s3_client
        self.bucket = bucket
        self.s3_prefix = s3_prefix.rstrip("/")

        # Reusable converter and chunker objects
        self.converter = DocumentConverter()
        self.chunker = HybridChunker()

    def convert_once(self, file_path: Path):
        """Convert the source file once and return the native DoclingDocument."""
        result = self.converter.convert(str(file_path))
        native_doc = result.document
        return native_doc


```


### Explanation

- `DocumentConverter()` creates the Docling conversion engine.
- `convert()` is called once per source file in this worker's `process()` method.
- `result.document` is the native Docling document, retained in memory.
- Text chunking and image extraction receive this object; they do not convert the PDF again.

## 4. Convert native Docling chunks into LangChain Documents

```python
    def create_langchain_chunks(self, native_doc, source: str) -> list[LCDocument]:
        langchain_docs = []

        for index, chunk in enumerate(self.chunker.chunk(native_doc)):
            # Include headings/captions as context in the text sent for embedding.
            text_for_embedding = self.chunker.contextualize(chunk)


            metadata: dict[str, Any] = {
                "source": source,
                "chunk_id": index,
                "content_type": "text",
            }


            # Add heading breadcrumb when available.
            headings = getattr(chunk.meta, "headings", None)
            if headings:
                metadata["headings"] = headings


            # Page number, when available in chunk provenance.
            origin = getattr(chunk.meta, "origin", None)
            page_no = getattr(origin, "page_no", None) if origin else None
            if page_no is not None:
                metadata["page_no"] = page_no


            langchain_docs.append(
                LCDocument(page_content=text_for_embedding, metadata=metadata)
            )


        return langchain_docs
```

### Explanation

1. `self.chunker.chunk(native_doc)` chunks the already-converted native document.
2. `contextualize(chunk)` adds useful structural context such as headings/captions to the chunk text for embedding.
3. Metadata records source, chunk index, headings, and page number if available.
4. Each output is a standard LangChain `Document` that can be sent to an embedding model/vector store.

## 5. Extract images from the same native document and upload to S3

```python
    def extract_images_and_upload(self, native_doc, source: str) -> list[dict[str, Any]]:
        assets = []
        source_stem = Path(source).stem


        for index, picture in enumerate(native_doc.pictures):
            image = picture.get_image(native_doc)
            if image is None:
                continue


            # Use a temporary directory to avoid collisions and clean up local files.
            with tempfile.TemporaryDirectory() as tmp_dir:
                local_image = Path(tmp_dir) / f"picture_{index}.png"
                image.save(local_image, format="PNG")


                s3_key = (
                    f"{self.s3_prefix}/{source_stem}/"
                    f"picture_{index}.png"
                )

                self.s3.upload_file(str(local_image), self.bucket, s3_key)


            assets.append(
                {
                    "asset_id": f"{source_stem}_picture_{index}",
                    "asset_type": "image",
                    "source": source,
                    "s3_uri": f"s3://{self.bucket}/{s3_key}",
                    "content_type": "image/png",
                }
            )

        return assets
```

### Explanation

- `native_doc.pictures` iterates the picture items detected in the converted document.
- `picture.get_image(native_doc)` retrieves a picture image from the native document where image data is available.
- Each image is saved temporarily and uploaded to S3.
- The temporary directory is automatically removed after upload.
- This extracts picture assets; it does **not** guarantee that every chart will be identified as a chart. Chart classification/interpretation can be added as a separate vision-model step.

## 6. Complete processing method

Add the following method inside the `SharedDoclingWorker` class:


```python
    def process(self, file_path: Path) -> dict[str, Any]:
        if not file_path.exists():
            raise FileNotFoundError(f"PDF not found: {file_path}")

        source = str(file_path)
        print(f"Processing: {source}")

        # STEP 1 — Convert ONCE
        native_doc = self.convert_once(file_path)
        print("Docling conversion completed (one conversion).")

        # STEP 2 — Reuse the same native document for text chunking
        chunks = self.create_langchain_chunks(native_doc, source=source)
        print(f"LangChain chunks created: {len(chunks)}")

        # STEP 3 — Reuse the same native document for picture extraction
        assets = self.extract_images_and_upload(native_doc, source=source)
        print(f"Picture assets uploaded: {len(assets)}")

        return {
            "source": source,
            "chunks": chunks,
            "assets": assets,
            "chunk_count": len(chunks),
            "asset_count": len(assets),
        }
```

## 7. Run it against your local PDF

```python
worker = SharedDoclingWorker(
    s3_client=s3,
    bucket=S3_BUCKET,
    s3_prefix=S3_PREFIX,
)


result = worker.process(file_path)

chunks = result["chunks"]
assets = result["assets"]

print("Chunks:", result["chunk_count"])
print("Assets:", result["asset_count"])

# Preview one chunk
if chunks:
    print(chunks[0].page_content)
    print(chunks[0].metadata)

# Preview uploaded asset metadata
for asset in assets[:3]:
    print(asset)
```

## 8. Optional: save the native document for restart/recovery

If you want to avoid reconverting a PDF after a worker restart, save the native Docling document in a reusable format, such as Docling JSON, then reload it according to the API supported by your pinned Docling version. Validate image-reference behavior before relying on a serialized document for asset extraction; local image references may not remain available after moving the JSON to another machine.


## 9. Processing flow

```text
Local PDF
  E:/Euron_Super_30_project/practice/source_docs_rag/doclaynet.pdf
        |
        v
Shared Docling Worker
        |
        +-- DocumentConverter.convert()  <-- exactly one conversion call
        |       |
        |       v
        |   Native DoclingDocument (kept in memory)
        |       |
        |       +--> HybridChunker --> contextualized chunks --> LangChain Documents
        |       |
        |       +--> picture.get_image(native_doc) --> temporary PNG --> S3
        |
        v
Return chunks + asset metadata
```

## 10. Why this is a shared-worker design

| Step | Input | Does it convert the PDF? |
|---|---|---:|
| `convert_once()` | Original local PDF | **Yes — once** |
| `create_langchain_chunks()` | Native `DoclingDocument` | No |
| `extract_images_and_upload()` | Same native `DoclingDocument` | No |
| Upload images to S3 | Extracted PNGs | No |
| LLM image captioning (if added) | Extracted images | No PDF conversion; separate model inference |


## Production cautions

- This is a shared native-Docling-worker pattern. It does not use `DoclingLoader` for the conversion because the worker needs to retain the native `DoclingDocument` and share it across downstream operations.
- Pin and test your Docling versions. The native picture extraction and chunking APIs should be validated in the exact environment you deploy.
- If you later add a LangChain loader path, do not run it separately on the same PDF unless you intentionally accept another conversion. Instead, create LangChain `Document` objects from the shared native document as shown above.
- Use a stable document ID and checksum in S3 object keys in production. The sample keys based on the filename are simplified and may overwrite assets from different versions of a document.
- Ensure picture generation/extraction is enabled in the PDF pipeline options if your selected Docling pipeline requires it for picture data. Validate that `picture.get_image(native_doc)` returns actual images for your PDF.
- The code uploads picture assets to S3 but does not yet embed them, run vision captioning, or write embeddings to a vector database. Those are separate downstream steps.

## References

- Docling chunking concepts: https://docling-project.github.io/docling/concepts/chunking/
- Docling Python SDK: https://github.com/docling-project/docling/blob/main/docling/.agents/skills/docling/references/python-sdk.md
