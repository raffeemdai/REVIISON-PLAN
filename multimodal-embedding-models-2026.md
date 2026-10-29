# Best Multimodal Embedding Models from Top Providers (as of Oct 29, 2026)

A multimodal embedding model maps text, images (and often video, audio, PDFs) into one shared vector space, so a text query can retrieve images, screenshots, or video clips directly.

## TL;DR

| Need | Pick |
|---|---|
| Widest modality coverage (text, image, video, audio, PDF) | **Google Gemini Embedding 2** |
| Visually rich documents / enterprise RAG with long context | **Cohere Embed 5 Pro** |
| Cheapest storage via dimension compression | **Voyage multimodal-3.5** (or Jina v4) |
| Already on AWS | **Amazon Nova 2 Multimodal Embeddings** |
| Self-hosting / open weights | **Qwen3-VL-Embedding (2B / 8B)** |

OpenAI offers no multimodal embedding API; its `text-embedding-3` models are text-only.

---

## Comparison

| Provider | Model | Modalities | Context | Dimensions | Price (approx.) | Access |
|---|---|---|---|---|---|---|
| Google | Gemini Embedding 2 | Text, image, video, audio, PDF | 8,192 tokens | 3072 default, MRL down to 128 | $0.20 / 1M tokens (text); audio $6.50 / 1M | Gemini API, Vertex AI |
| Cohere | Embed 5 Pro / Fast | Text, image, mixed text+image (PDF pages) | 128K tokens | 256–2048 (MRL); float, int8, binary | Pro: $0.12 text, $0.40 image / 1M. Fast: $0.08 text, $0.40 image / 1M | Cohere API, Azure Foundry, SageMaker, VPC |
| Voyage AI (MongoDB) | voyage-multimodal-3.5 | Text, image, video, interleaved | 32K tokens | 256 / 512 / 1024 / 2048 | $0.12 / 1M text; $0.60 / B pixels | Voyage API, Azure Foundry |
| Amazon | Nova 2 Multimodal Embeddings | Text, image, video, audio, documents | 8K tokens (async segmentation for long inputs) | 256 / 384 / 1024 / 3072 | $0.135 / 1M text; ~$0.0007 / sec video | Amazon Bedrock |
| Jina AI (Elastic) | jina-embeddings-v4 | Text, image, PDF | 32K tokens | 2048 single-vector, plus multi-vector | API; weights under Qwen Research License (non-commercial without a license) | Jina API, Hugging Face |
| Alibaba Qwen | Qwen3-VL-Embedding 2B / 8B | Text, image, screenshots, video | 32K tokens | 2048 (2B) / 4096 (8B), MRL | Free to self-host (check license on Hugging Face) | Hugging Face, vLLM |

---

## Provider Notes

### Google: Gemini Embedding 2
- Launched March 10, 2026 in public preview; later listed as generally available.
- The only commercial API in this list that natively handles all five modalities, with interleaved inputs and 100+ languages.
- Vector space is incompatible with `gemini-embedding-001`, so migrating means re-embedding everything.
- **Weakness:** not optimized for dimension truncation (lowest MRL score in the benchmark below).

### Cohere: Embed 5 (Pro and Fast)
- Released September 30, 2026. Pro and Fast share one embedding space, so you can index with Pro and query with Fast.
- 128K context and image-aware PDF/slide/table handling reduce the need for document parsing.
- Cohere reports ViDoRe V3 averages of 85.8 (Pro) and 84.5 (Fast) versus 77.0 for Embed 4. These are vendor-reported figures using Cohere's own method.
- No video or audio.

### Voyage AI: voyage-multimodal-3.5
- Released January 2026. Handles interleaved text, images, and video through a single backbone.
- Trained with Matryoshka learning and quantization awareness, so quality holds up at small dimensions.
- The Voyage 4 text family shares one embedding space across its models (cheap indexing, premium querying), but that shared space applies to the text models, not to multimodal-3.5.

### Amazon: Nova 2 Multimodal Embeddings
- Text, documents, images, video, and audio in one space; up to 200 languages.
- Best fit if your data already lives in S3 / OpenSearch / S3 Vectors.

### Jina AI: jina-embeddings-v4
- 3.8B parameters, built on a Qwen2.5-VL backbone. Strong on visual document retrieval (84.11 average on ViDoRe, 90.17 in multi-vector mode, per Jina).
- Supports multi-vector (late interaction) output and task-specific LoRA adapters.
- **Licensing caveat:** weights are not free for commercial use.

### Open weights: Qwen3-VL-Embedding
- Released January 2026. The 8B model scored 77.8 on MMEB-V2, ranking first among all models at release (per the paper).
- Supports 30+ languages, instruction-aware prompts, and MRL.
- NVIDIA's Llama Nemotron Embed VL 1B V2 is another compact open option for multimodal document QA retrieval.

---

## Independent Benchmark Snapshot (Zilliz "CCKM", March 2026)

Small sample sizes; tight gaps may be noise. Cohere Embed 5 and Qwen3-VL-8B were not part of this test.

| Model | Cross-modal (R@1) | Cross-lingual | Long-doc (up to 32K) | 256-dim quality retention |
|---|---|---|---|---|
| Gemini Embedding 2 | 0.928 | **0.997** | **1.000** (only one tested at 32K) | Weakest (rho 0.68) |
| Qwen3-VL-Embedding-2B | **0.945** | 0.988 | 1.000 (within 4K tested) | Mid (0.77) |
| Voyage multimodal-3.5 | 0.900 | 0.982 | 1.000 (within 4K tested) | **Best** (0.88) |
| Jina Embeddings v4 | not tested | 0.985 | 1.000 (up to 8K) | Strong (0.83) |
| Jina CLIP v2 | 0.873 | 0.934 | 1.000 (up to 8K) | n/a |
| CLIP ViT-L-14 (2021 baseline) | 0.768 | 0.030 | n/a | n/a |

Takeaways:
- Gemini is the best all-rounder; the open 2B Qwen model beats every closed API on hard cross-modal retrieval because of its much smaller modality gap (0.25 vs 0.73 for Gemini).
- Voyage and Jina v4 lose under 1% quality when truncated to 256 dimensions, a 4x–12x storage saving.

---

## How to Choose

1. **Need audio or video plus text and images?** Gemini Embedding 2 or Amazon Nova 2.
2. **Mostly PDFs, slides, and screenshots?** Cohere Embed 5 Pro, Jina v4, or Voyage multimodal-3.5.
3. **Large corpus, storage-sensitive?** Voyage multimodal-3.5 with 256–512 dims.
4. **Data sovereignty or self-hosting?** Qwen3-VL-Embedding (verify license for your use).
5. **Locked into a cloud?** Google, Vertex/Gemini; AWS, Nova 2; Azure, Cohere or Voyage via Foundry.

## Caveats

- Public leaderboards (MTEB) are mostly text-only, and several 2026 commercial models are barely benchmarked on them. Many headline numbers are vendor-reported.
- Embeddings from different models (or versions) are not interchangeable; switching requires re-embedding.
- Always run 50–200 of your own queries against candidate models before committing.

## Sources

- Zilliz CCKM benchmark: zilliz.com/learn/choose-embedding-model-rag-2026
- Cohere Embed 5 announcement: docs.cohere.com/changelog/embed-v5
- Voyage multimodal docs: mongodb.com/docs/voyageai/models/multimodal-embeddings
- AWS Nova Multimodal Embeddings: aws.amazon.com/blogs/aws/amazon-nova-multimodal-embeddings-now-available-in-amazon-bedrock
- Jina v4: jina.ai/models/jina-embeddings-v4
- Qwen3-VL-Embedding: huggingface.co/Qwen/Qwen3-VL-Embedding-8B
- Pricing aggregators (cloudprice.net, llmreference.com, OpenRouter); verify current rates with each provider.
