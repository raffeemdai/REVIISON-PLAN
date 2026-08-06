# LLM Basics — Interview Preparation Guide

**Source basis:** Prepared only from the attached:
1. **AI Engineering Guidebook Full-compressed.pdf** — LLM chapter
2. **Fine Tuning By Me Quality Thought.docx** — LLM, neural-network, Transformer, training, alignment, and related interview notes

**Goal:** Focus on the most important **basic + interview-relevant LLM topics**. Advanced topics are included only where they are useful for interviews.

---

# 1. What is an LLM?

**LLM = Large Language Model.**

An LLM is a **deep-learning / neural-network model**, generally based on the **Transformer architecture**, trained on very large amounts of text.

Its core training objective is simple:

> **Given the tokens that came before, predict the next token.**

Example:

```text
Input:  The capital of France is
Next token: Paris
```

The same next-token-prediction process is repeated again and again to generate a complete answer.

### Interview answer

**Q: What is an LLM?**

An LLM is a Transformer-based neural-network model trained on massive text corpora using next-token prediction. It learns statistical patterns of language such as grammar, syntax, semantic relationships, coding patterns, and statistical world knowledge, and can use those learned patterns to generate text, answer questions, summarize, and write code.

---

# 2. Why are LLMs important?

Before LLMs, NLP systems were often built separately for individual tasks:

- Translation model for translation
- Sentiment model for sentiment analysis
- Summarization model for summarization

LLMs changed this because a **single foundation model can perform many tasks** depending on the prompt.

Examples:

- Question answering
- Summarization
- Text generation
- Code generation
- Translation
- Analysis

### Interview point

**Traditional NLP:** often one model per task.  
**LLM:** one general-purpose model can handle many language tasks.

---

# 3. What makes an LLM “large”?

The word **large** mainly refers to scale in:

1. **Number of parameters**
2. **Amount of training data**
3. **Training compute**

### Parameters

Parameters are learned numerical values inside the model.

Modern LLMs may contain **billions of parameters**.

These parameters are adjusted during training and encode the statistical patterns the model learns.

### Interview answer

**Q: What does “large” mean in Large Language Model?**

It refers mainly to the large number of model parameters, the huge amount of training data, and the large amount of compute used for training.

---

# 4. Basic Neural-Network Concepts

Before understanding an LLM, understand these terms:

- Neuron
- Weight
- Bias
- Activation
- Forward propagation
- Loss
- Backpropagation
- Optimizer

---

## 4.1 Weights

Weights are numerical values that determine how strongly an input influences the output.

During training, weights are changed to improve predictions.

Simple idea:

```text
Input × Weight
      ↓
Weighted value
```

---

## 4.2 Bias

Bias is an additional learned parameter added to the weighted input.

Conceptually:

```text
z = (w1*x1 + w2*x2 + ...) + bias
```

Bias helps shift the neuron's output and gives the model more flexibility.

---

## 4.3 Activation Function

An activation function transforms the weighted input and bias into the neuron's output.

Simple flow:

```text
Inputs
  ↓
Weights + Bias
  ↓
Activation Function
  ↓
Output
```

---

## 4.4 Forward Propagation

Forward propagation means sending input through the neural network to generate a prediction.

For LLM training:

```text
Tokens
  ↓
Embeddings
  ↓
Transformer layers
  ↓
Logits
  ↓
Probabilities
  ↓
Predicted next token
```

---

## 4.5 Loss

Loss tells the model how wrong its prediction was.

For LLM next-token training, **cross-entropy loss** is used.

Simple rule:

```text
Correct token gets high probability  -> Low loss
Correct token gets low probability   -> High loss
```

---

## 4.6 Backpropagation

Backpropagation sends the error backward through the network.

It determines how each weight contributed to the error and calculates gradients.

Then an optimizer updates the weights.

```text
Prediction
   ↓
Compare with actual next token
   ↓
Loss
   ↓
Backpropagation
   ↓
Gradients
   ↓
Update weights
```

The Word document mentions **AdamW** as a commonly used optimizer in LLM training.

### Interview answer

**Q: What is backpropagation?**

Backpropagation is the process of propagating the loss backward through the neural network, calculating gradients for the trainable parameters, and using those gradients to update weights so future predictions improve.

---

# 5. RNN, LSTM and Why Transformers Matter

You do not need deep RNN/LSTM knowledge for a basic LLM interview, but you should know the evolution.

## RNN

RNN = Recurrent Neural Network.

RNNs process sequential data and maintain information from previous steps.

Problem:

- Difficulty remembering information across long sequences
- Vanishing-gradient problem
- Sequential processing makes scaling harder

## LSTM

LSTM = Long Short-Term Memory.

LSTM is a special RNN designed to keep important information for longer using:

- Memory cell
- Input gate
- Forget gate
- Output gate

## Transformer

Transformers process sequences using **attention instead of recurrence**.

This is the architecture used by modern LLMs.

### Simple comparison

| Model | Main Idea | Limitation / Benefit |
|---|---|---|
| RNN | Sequential recurrence | Short memory, difficult long dependencies |
| LSTM | RNN + controlled memory | Better long-term memory, still sequential |
| Transformer | Attention-based processing | Better scaling and context relationships |

### Interview answer

**Q: Why did Transformers become more popular than RNN/LSTM for LLMs?**

Transformers use attention to model relationships between tokens and can process tokens more efficiently than recurrent architectures. This makes them much more suitable for large-scale language-model training.

---

# 6. Tokenization

LLMs cannot directly process raw text.

The text is first divided into **tokens**.

A token can be:

- A full word
- Part of a word
- Punctuation
- Another small text unit depending on the tokenizer

Example:

```text
Text:
"The cat is playing."

Possible token idea:
["The", "cat", "is", "play", "ing", "."]
```

### Why tokenization?

It gives the model a manageable vocabulary and converts text into units that can be mapped to numbers.

### Interview answer

**Q: What is tokenization?**

Tokenization is the process of breaking input text into smaller units called tokens. These tokens are converted to token IDs and then numerical embeddings before being processed by the Transformer.

---

# 7. Embeddings

Neural networks work with numbers, not words.

An **embedding** converts a token into a numerical vector.

Example:

```text
"cat"
   ↓
[0.33, 0.19, 0.72, ...]
```

Embeddings represent learned properties and relationships of tokens in numerical form.

### Interview answer

**Q: What is an embedding?**

An embedding is a dense numerical vector representation of a token that allows the model to process semantic and contextual information mathematically.

---

# 8. Positional Encoding / Positional Information

Attention by itself does not naturally know word order.

Compare:

```text
Dog bites man
Man bites dog
```

The words are similar, but their order changes the meaning.

Positional information is added to token representations so the model understands sequence order.

```text
Token embedding
      +
Position information
      ↓
Position-aware representation
```

### Interview answer

**Q: Why do Transformers need positional encoding?**

Transformers process tokens using attention and do not inherently know token order. Positional information tells the model where each token appears in the sequence.

---

# 9. Transformer Architecture — Most Important Interview Topic

A simplified LLM flow is:

```text
Text
 ↓
Tokenization
 ↓
Token IDs
 ↓
Embeddings
 ↓
Positional Information
 ↓
Transformer Blocks
 ↓
Linear Output Layer
 ↓
Logits
 ↓
Softmax
 ↓
Next-token probabilities
```

The Transformer block mainly contains:

1. Self-Attention
2. Multi-Head Attention
3. Residual connection
4. Layer normalization
5. Feed-Forward Network (FFN)

Modern GPT-style LLMs are generally **decoder-only Transformer models**.

---

# 10. What is Attention?

Attention helps the model determine which tokens are important to each other.

Example:

```text
"The animal didn't cross the road because it was tired."
```

To understand **"it"**, the model must determine whether "it" refers to:

- animal
- road

Attention assigns different importance to the tokens and uses context to build a better representation.

### Interview answer

**Q: What is attention?**

Attention is a mechanism that lets the model assign different importance to tokens when processing another token, allowing it to capture contextual relationships between words.

---

# 11. Self-Attention

Self-attention means tokens in the **same sequence** attend to other tokens in that sequence.

Each token receives a context-aware representation based on other relevant tokens.

### One-line interview answer

> Self-attention allows each token to look at other tokens in the same sequence and determine which ones are most relevant to building its contextual representation.

---

# 12. Query, Key and Value — Q, K, V

Each token representation is projected into three vectors:

- **Query (Q)**
- **Key (K)**
- **Value (V)**

Simple mental model:

- **Query:** What am I looking for?
- **Key:** What information do I represent?
- **Value:** What information should I contribute?

Flow:

```text
Query of current token
       ↓
Compare with Keys of tokens
       ↓
Attention scores
       ↓
Softmax
       ↓
Attention weights
       ↓
Weighted combination of Values
       ↓
Context-aware representation
```

### Interview answer

**Q: Explain Q, K and V.**

Each token is transformed into Query, Key and Value vectors. The Query is compared with Keys to calculate attention scores. Those scores determine how much of each Value should contribute to the output representation.

---

# 13. Scaled Dot-Product Attention

Basic flow:

```text
Q × K
 ↓
Attention scores
 ↓
Scale scores
 ↓
Softmax
 ↓
Attention weights
 ↓
Weighted Values
```

The Word document notes that attention scores are divided by **sqrt(d_k)** before softmax to keep values stable and avoid extremely small gradients.

### Formula to recognize

```text
Attention(Q,K,V) = softmax(QKᵀ / √d_k)V
```

You do not need to derive this formula in a basic interview, but you should know what it represents.

---

# 14. Multi-Head Attention

Instead of doing one attention calculation, Transformers use multiple attention heads.

Different heads may learn different relationships such as:

- Subject relationship
- Verb relationship
- Position
- Semantic similarity
- Long-range relationship

Then the outputs of all heads are combined.

### Interview answer

**Q: Why do we need multi-head attention?**

Multi-head attention allows the model to learn multiple types of relationships between tokens in parallel instead of relying on a single attention pattern.

---

# 15. Residual Connections

A residual or skip connection adds the original input back to a sub-layer's output.

Example:

```text
Output = Attention_Output + Original_Input
```

Purpose:

- Prevent useful information from being lost
- Help deep networks train more effectively

---

# 16. Layer Normalization

Layer normalization stabilizes values flowing through deep Transformer layers.

Purpose:

- Stabilize training
- Improve convergence
- Keep activations in a controlled range

---

# 17. Feed-Forward Network (FFN)

After attention, each token representation is passed through a small neural network called a **Feed-Forward Network**.

Purpose:

> Apply nonlinear transformations and refine the token representation.

A Transformer block repeatedly combines attention and FFN processing.

---

# 18. Stacked Transformer Layers

An LLM contains many Transformer layers.

```text
Input
 ↓
Transformer Layer 1
 ↓
Transformer Layer 2
 ↓
Transformer Layer 3
 ↓
...
 ↓
Transformer Layer N
```

Each layer builds increasingly rich contextual representations.

---

# 19. Logits

The model's final output layer produces raw scores for possible next tokens.

These raw scores are called **logits**.

Example:

```text
Token        Logit
------------------
Paris        8.2
London       4.1
Berlin       2.8
```

Logits are **not probabilities**.

---

# 20. Softmax

Softmax converts logits into probabilities.

```text
Logits
  ↓
Softmax
  ↓
Probability distribution
```

Example:

```text
Paris   0.80
London  0.15
Berlin  0.05
```

Probabilities sum to 1.

### Interview question

**Q: What is the difference between logits and softmax output?**

Logits are raw unnormalized scores. Softmax converts those scores into a normalized probability distribution.

---

# 21. How Does an LLM Generate Text?

LLM generation is **autoregressive**.

At every step:

```text
Previous tokens
      ↓
Transformer
      ↓
Next-token probability distribution
      ↓
Select/sample next token
      ↓
Append token to context
      ↓
Repeat
```

Example:

```text
Prompt: "Once"

Predict: "upon"

New context: "Once upon"

Predict: "a"

New context: "Once upon a"

Predict: "time"

...
```

The process continues until:

- A stop token / EOS is generated
- A stop sequence is reached
- Maximum generation limit is reached

---

# 22. Conditional Probability

LLM next-token prediction can be understood as:

```text
P(next token | previous tokens)
```

The model calculates the probability of possible next tokens conditioned on the existing context.

This is an important conceptual interview point.

---

# 23. Training vs Inference

## Training

During training:

```text
Input
 ↓
Prediction
 ↓
Compare with correct target
 ↓
Loss
 ↓
Backpropagation
 ↓
Update weights
```

**Weights change.**

## Inference

During inference:

```text
Prompt
 ↓
Forward pass
 ↓
Probabilities
 ↓
Generate next token
```

**Weights do not normally change.**

### Interview answer

**Q: What is the difference between training and inference?**

Training updates the model's weights using loss and backpropagation. Inference uses already-trained weights to generate predictions without updating them.

---

# 24. How Are LLMs Trained?

For interview preparation, remember this three-stage high-level model:

```text
Pretraining
    ↓
Supervised Fine-Tuning (SFT)
    ↓
Alignment / Preference Tuning
```

The AI Engineering PDF also discusses **reasoning fine-tuning** as an additional stage for tasks with verifiable answers.

---

# 25. Pretraining

Pretraining is the foundation stage.

Objective:

> Predict the next token from large-scale text data.

Typical data mentioned in the attached documents includes:

- Web pages
- Books
- Wikipedia
- Code
- Academic articles
- Conversations

### Pipeline

```text
1. Collect massive text data
2. Tokenize text
3. Convert tokens to embeddings
4. Forward pass through Transformer
5. Produce logits
6. Convert to probabilities
7. Compare prediction with true next token
8. Calculate cross-entropy loss
9. Backpropagate
10. Update weights using optimizer
11. Repeat at massive scale
```

### Self-supervised learning

Pretraining is described as **self-supervised** because the text itself provides the label.

Example:

```text
Text:
"The sky is blue"

Input:
"The sky is"

Target:
"blue"
```

No human has to manually label "blue".

### Interview answer

**Q: What happens during pretraining?**

During pretraining, the model learns next-token prediction from massive text datasets. It tokenizes text, performs a forward pass through Transformer layers, calculates cross-entropy loss against the actual next token, backpropagates the error, and updates its weights.

---

# 26. What Does the Model Learn During Pretraining?

The Word document lists:

- Grammar
- Syntax
- Semantic relationships
- Reasoning patterns
- Coding patterns
- Statistical world knowledge

Important interview distinction:

> The model does **not** behave like a structured database of retrievable facts or symbolic rules. Its learned patterns are distributed across its weights.

This is one reason LLMs can produce confident but incorrect responses.

---

# 27. Base / Pretrained Model

After pretraining, we have a **base / foundation model**.

It is good at predicting and continuing text, but it may not naturally behave like an assistant.

Example:

```text
User:
"What is the capital of France?"

Base model:
May continue the text in a way that resembles its training corpus instead of directly answering.
```

### Interview answer

**Q: What is a base model?**

A base or pretrained model has learned general language through next-token prediction but has not necessarily been instruction-tuned or aligned to behave like an assistant.

---

# 28. Supervised Fine-Tuning (SFT)

SFT is also called **instruction tuning**.

After pretraining, the model knows language but needs to learn how to follow instructions.

SFT uses curated:

```text
Instruction → Response
```

pairs.

Example:

```text
User:
Explain photosynthesis simply.

Assistant:
...
```

### SFT teaches

- Follow instructions
- Format answers properly
- Be conversational

The Word document emphasizes an important point:

> Pretraining and SFT both use next-token prediction. The main difference is the **training data**.

### Interview answer

**Q: Pretraining vs SFT?**

Pretraining learns general language patterns from massive raw text, while SFT uses smaller curated instruction-response pairs to teach the model how to follow instructions and behave like an assistant.

---

# 29. RLHF — Reinforcement Learning from Human Feedback

Even after SFT, model behavior can still need improvement.

RLHF uses human preferences.

Basic pipeline:

```text
Prompt
 ↓
Generate multiple responses
 ↓
Humans rank responses
 ↓
Train Reward Model
 ↓
Reward Model scores outputs
 ↓
Use reinforcement learning
 ↓
Improve LLM behavior
```

### Goal

Make responses more:

- Helpful
- Safe
- Preferred by users

---

# 30. Reward Model

A Reward Model takes:

```text
Prompt + Response
```

and outputs a score.

It learns from human preference rankings such as:

```text
A > C > B
```

The reward model acts as an automated preference judge during RL training.

### Interview answer

**Q: What is a reward model?**

A reward model is a separate model trained from human preference data to score how desirable an LLM response is. It replaces the need for humans to manually judge every training iteration.

---

# 31. PPO

The attached material describes **PPO (Proximal Policy Optimization)** as an algorithm commonly used in classic RLHF.

Simple understanding:

```text
LLM generates response
 ↓
Reward model scores response
 ↓
High reward → reinforce
Low reward  → discourage
 ↓
Model gradually shifts toward preferred responses
```

For a basic interview, you do not need PPO mathematics.

---

# 32. KL-Divergence in RLHF

The Word document includes **KL-divergence penalty** as an important stabilizer.

Purpose:

> Prevent the RLHF-updated model from drifting too far away from the SFT model.

Without this control, the model can exploit weaknesses in the reward model — called **reward hacking**.

### Interview answer

**Q: Why is KL-divergence used in RLHF?**

It constrains the updated model so its output distribution does not move too far from the original instruction-tuned model, helping prevent unstable or reward-hacking behavior.

---

# 33. Reasoning Fine-Tuning / Verifiable Rewards

The PDF also introduces reasoning fine-tuning.

For tasks such as:

- Mathematics
- Logic
- Other tasks with known correct answers

we can use correctness directly as a reward signal.

Flow:

```text
Prompt
 ↓
Model answer
 ↓
Compare with known correct answer
 ↓
Reward based on correctness
 ↓
Update model
```

The PDF calls this **Reinforcement Learning with Verifiable Rewards** and mentions **GRPO** as a technique.

### Interview point

**Preference task:** humans may need to say which answer is better.  
**Verifiable reasoning task:** correctness can provide the reward.

---

# 34. Generation Parameters

These parameters affect how a trained model generates text.

The PDF highlights:

1. Max tokens
2. Temperature
3. Top-K
4. Top-P
5. Frequency penalty
6. Presence penalty
7. Stop sequences

---

# 35. Max Tokens

Defines the maximum amount of output the model may generate.

Too low:

- Output may be truncated

Too high:

- More unnecessary compute may be used

---

# 36. Temperature

Temperature controls randomness.

### Low temperature

- More deterministic
- More predictable
- More focused

Useful for:

- Q&A
- Factual chatbot-style responses

### Higher temperature

- More diversity
- More creativity
- More randomness

Useful for:

- Brainstorming
- Creative writing

### Interview answer

**Q: What does temperature do?**

Temperature changes the shape of the next-token probability distribution. Lower values concentrate probability on likely tokens; higher values make the distribution flatter and generation more diverse.

---

# 37. Top-K

Top-K keeps only the **K most probable candidate tokens** before sampling.

Example:

```text
top_k = 5
```

Only the 5 highest-probability next tokens are considered.

---

# 38. Top-P / Nucleus Sampling

Top-P keeps the smallest set of tokens whose cumulative probability reaches **P**.

Example:

```text
top_p = 0.90
```

The model keeps enough high-probability candidates to cover 90% of the probability mass.

### Top-K vs Top-P

| Top-K | Top-P |
|---|---|
| Fixed number of candidates | Variable number of candidates |
| Example: keep 10 tokens | Example: keep tokens totaling 90% |
| Less adaptive | More adaptive to confidence |

### Interview memory trick

> **Top-K = Count**  
> **Top-P = Probability mass**

---

# 39. Frequency Penalty

Frequency penalty reduces repeated use of tokens that have already appeared frequently.

Use it when you want to reduce repetitive output.

---

# 40. Presence Penalty

Presence penalty encourages the model to introduce tokens/topics not already present in the generated text.

### Frequency vs Presence

**Frequency penalty:** How many times has the token appeared?  
**Presence penalty:** Has the token appeared at all?

---

# 41. Stop Sequence

A stop sequence tells generation when to stop.

Useful for:

- Structured output
- API workflows
- Enforcing response boundaries

---

# 42. Decoding Strategies

The PDF discusses four main generation strategies:

1. Greedy decoding
2. Multinomial sampling
3. Beam search
4. Contrastive search

---

## 42.1 Greedy Decoding

Always select the highest-probability next token.

Advantages:

- Simple
- Deterministic

Disadvantage:

- Can become repetitive

---

## 42.2 Multinomial Sampling

Sample the next token based on the probability distribution instead of always picking the highest one.

Advantage:

- More diverse output

Temperature can influence randomness.

---

## 42.3 Beam Search

Instead of keeping only one partial sequence, beam search keeps multiple candidate sequences.

At each step:

```text
Expand candidates
 ↓
Score candidates
 ↓
Keep best beams
 ↓
Continue
```

Useful when overall sequence quality/correctness is more important than creativity, such as machine translation.

---

## 42.4 Contrastive Search

Contrastive search tries to balance:

```text
High probability
      +
Low repetition
```

It penalizes candidate continuations that are too similar to what was already generated.

Goal:

- Maintain coherence
- Reduce repetition

---

# 43. Model Parameters vs Generation Parameters

Do not confuse them.

## Model Parameters

Learned during training.

Examples:

- Weights
- Biases

They represent what the model has learned.

## Generation Parameters

Set at inference/generation time.

Examples:

- Temperature
- Top-K
- Top-P
- Max tokens
- Stop sequence

They control how the trained model generates output.

---

# 44. Parameter vs Hyperparameter

## Parameter

Learned by training.

Examples:

- Model weights
- Biases

## Hyperparameter / configuration

Chosen by developers or training setup.

Examples mentioned across the materials include:

- Learning rate
- Batch size
- Training settings
- Temperature
- Top-K
- Top-P

---

# 45. Pretrained Model vs Instruct Model

This distinction is explicitly emphasized in the Word document.

## Pretrained / Base Model

- Mainly knows how to continue text
- Learned from pretraining
- Not necessarily instruction-following

## Instruct Model

A pretrained model that has received additional tuning such as:

- Instruction tuning
- Preference/alignment tuning

It is designed to:

- Follow user instructions
- Hold conversations
- Behave more safely

### Interview answer

**Q: Base model vs instruct model?**

A base model is primarily trained for next-token prediction, while an instruct model has undergone post-training so it follows instructions and behaves more like an assistant.

---

# 46. Dense Transformer vs Mixture of Experts (MoE)

This is an important advanced-basic topic.

## Dense Transformer

A traditional Transformer layer uses its feed-forward network for every token.

## Mixture of Experts

MoE has multiple smaller feed-forward **experts**.

A **router** selects only a subset of experts for each token.

```text
Token
 ↓
Router
 ↓
Select Top-K experts
 ↓
Selected experts process token
 ↓
Combine result
```

### Main benefit

The total parameter count can be very large while only a portion of parameters is active for a token.

### Interview answer

**Q: What is Mixture of Experts?**

MoE is a Transformer architecture where multiple expert feed-forward networks exist and a router selects a small subset of experts for each token, allowing higher model capacity without activating every parameter for every token.

---

# 47. MoE Router

The router behaves like a classifier over experts.

It produces scores for the available experts and selects the top experts.

The attached PDF also notes an important challenge:

> If the same experts are selected too often, some experts may become under-trained.

So MoE systems need expert-balancing mechanisms.

For a basic interview, knowing the router's role is sufficient.

---

# 48. Knowledge Distillation

Distillation transfers behavior/knowledge from a larger **Teacher model** to a smaller **Student model**.

```text
Teacher LLM
    ↓
Knowledge / outputs
    ↓
Student LLM
```

The PDF covers:

- Soft-label distillation
- Hard-label distillation
- Co-distillation

---

## 48.1 Soft-Label Distillation

Student tries to match the Teacher's entire output probability distribution.

Benefit:

- Transfers more information

Limitation:

- Large storage/memory requirement for full distributions

---

## 48.2 Hard-Label Distillation

Student learns from the Teacher's selected final output/token rather than the full probability distribution.

Benefit:

- Much cheaper to store

---

## 48.3 Co-Distillation

Teacher and Student can be trained together.

Student learns from:

- Teacher probabilities
- Ground-truth labels

### Interview-level definition

**Q: What is model distillation?**

Distillation trains a smaller Student model to imitate a larger Teacher model so the Student can retain useful behavior while being smaller or cheaper to run.

---

# 49. Ways to Run LLMs Locally

The PDF introduces:

- **Ollama**
- **LM Studio**
- **vLLM**
- **LlamaCPP**

You only need high-level differences for a basic interview.

## Ollama

Simple local LLM runtime with command-line and programmatic usage.

## LM Studio

Desktop GUI for downloading/loading and chatting with local models.

## vLLM

Fast inference and serving engine that can expose an OpenAI-compatible API.

## LlamaCPP

Local inference option designed for relatively easy and efficient execution.

---

# 50. Self-Hosted vs Cloud / Provider-Hosted LLMs

The Word document introduces three broad usage/hosting ideas:

### Self-hosted

You run the model on your own infrastructure.

Benefits may include:

- Data control
- Local/privacy requirements
- Greater infrastructure control

### Cloud-hosted

Cloud providers provide infrastructure/services for models.

Examples in the source material include:

- AWS
- Azure
- GCP

### Provider-hosted

The model provider hosts the service and applications call it through an API/service.

---

# 51. Open-Source vs Proprietary LLMs

Another basic interview distinction.

## Open / openly available models

Model artifacts are made available for developers to run or adapt under their respective licenses.

## Proprietary models

The model provider controls the model and generally exposes access through a hosted service/API.

For interviews, focus on the architectural difference:

```text
Self-host/open model → More infrastructure/control responsibilities
Hosted proprietary model → Provider manages model serving
```

---

# 52. Important Terms — Quick Recall

| Term | Interview Meaning |
|---|---|
| LLM | Large Transformer-based language model |
| Token | Small unit of text processed by the model |
| Tokenization | Splitting text into tokens |
| Token ID | Numeric identifier of a token |
| Embedding | Dense numeric vector representing a token |
| Parameter | Learned numerical value |
| Weight | Learned value controlling influence |
| Bias | Learned offset |
| Transformer | Attention-based neural-network architecture |
| Attention | Determines which tokens are relevant |
| Self-Attention | Tokens attend to tokens in the same sequence |
| Q/K/V | Vectors used to calculate attention |
| Multi-Head Attention | Multiple attention mechanisms in parallel |
| Positional Encoding | Adds sequence-order information |
| Residual Connection | Adds original input back to transformed output |
| LayerNorm | Stabilizes deep-network activations |
| FFN | Feed-forward network inside Transformer block |
| Logit | Raw model output score |
| Softmax | Converts logits to probabilities |
| Cross-Entropy Loss | Measures next-token prediction error |
| Backpropagation | Calculates gradients from error |
| AdamW | Optimizer mentioned for weight updates |
| Pretraining | Learn language by next-token prediction |
| SFT | Supervised fine-tuning / instruction tuning |
| RLHF | Reinforcement Learning from Human Feedback |
| Reward Model | Scores responses based on learned preferences |
| PPO | RL algorithm used in classic RLHF |
| KL Divergence | Helps control model drift during RLHF |
| Inference | Using trained model to generate output |
| Temperature | Controls randomness |
| Top-K | Sample from K highest-probability tokens |
| Top-P | Sample from tokens covering probability mass P |
| MoE | Mixture of Experts |
| Router | Selects experts in MoE |
| Distillation | Train Student model using Teacher model |

---

# 53. End-to-End LLM Flow — Must Know

This is the most important diagram to remember for an interview.

```text
USER PROMPT
    ↓
TOKENIZATION
    ↓
TOKEN IDs
    ↓
TOKEN EMBEDDINGS
    +
POSITION INFORMATION
    ↓
TRANSFORMER BLOCKS
    ├─ Self-Attention
    │    ├─ Q
    │    ├─ K
    │    └─ V
    ├─ Multi-Head Attention
    ├─ Residual Connection
    ├─ Layer Normalization
    └─ Feed-Forward Network
    ↓
FINAL HIDDEN REPRESENTATION
    ↓
LINEAR OUTPUT LAYER
    ↓
LOGITS
    ↓
SOFTMAX
    ↓
NEXT-TOKEN PROBABILITIES
    ↓
DECODING / SAMPLING
    ↓
NEXT TOKEN
    ↓
APPEND TO CONTEXT
    ↓
REPEAT
    ↓
FINAL RESPONSE
```

---

# 54. LLM Training Flow — Must Know

```text
MASSIVE TEXT DATA
     ↓
TOKENIZATION
     ↓
EMBEDDINGS
     ↓
TRANSFORMER FORWARD PASS
     ↓
LOGITS
     ↓
SOFTMAX
     ↓
PREDICT NEXT TOKEN
     ↓
COMPARE WITH ACTUAL NEXT TOKEN
     ↓
CROSS-ENTROPY LOSS
     ↓
BACKPROPAGATION
     ↓
GRADIENTS
     ↓
OPTIMIZER UPDATES WEIGHTS
     ↓
REPEAT AT MASSIVE SCALE
```

Then:

```text
PRETRAINED / BASE MODEL
        ↓
SUPERVISED FINE-TUNING
        ↓
INSTRUCTION-FOLLOWING MODEL
        ↓
PREFERENCE / ALIGNMENT TUNING
        ↓
ALIGNED ASSISTANT MODEL
```

---

# 55. Top Interview Questions and Answers

## Q1. What is an LLM?

An LLM is a Transformer-based neural-network model trained on large amounts of text using next-token prediction. It learns statistical language patterns that allow it to generate and understand text.

---

## Q2. What is next-token prediction?

Given previous tokens, the model predicts a probability distribution over possible next tokens and selects or samples one.

---

## Q3. What is a token?

A token is the basic unit of text processed by an LLM. It can be a word, part of a word, punctuation, or another tokenizer-defined text unit.

---

## Q4. Tokenization vs embedding?

Tokenization breaks text into tokens. Embedding converts each token into a numerical vector the neural network can process.

---

## Q5. What is a Transformer?

A Transformer is a deep-learning architecture built mainly around attention mechanisms and feed-forward networks for processing token sequences.

---

## Q6. What is self-attention?

Self-attention allows each token to compare itself with other tokens in the same sequence and combine information from the most relevant ones.

---

## Q7. Explain Query, Key and Value.

A token's Query is compared with other tokens' Keys to calculate relevance. Those relevance scores determine how strongly the corresponding Values contribute to the token's contextual representation.

---

## Q8. Why is positional encoding needed?

Attention does not inherently know word order. Positional information tells the model the position of tokens in the sequence.

---

## Q9. Why multi-head attention?

Multiple attention heads let the model learn different types of token relationships simultaneously.

---

## Q10. What is an embedding?

An embedding is a dense numerical vector representation of a token that enables the model to process its learned semantic/contextual properties mathematically.

---

## Q11. What are parameters in an LLM?

Parameters are learned numerical values such as weights and biases that are updated during training.

---

## Q12. What are logits?

Logits are the raw output scores produced by the model for possible next tokens before softmax.

---

## Q13. What does softmax do?

Softmax converts logits into a normalized probability distribution that sums to 1.

---

## Q14. What is cross-entropy loss?

It measures how different the predicted next-token probability distribution is from the true next-token target.

---

## Q15. What is backpropagation?

Backpropagation propagates the prediction error backward through the network to calculate gradients used to update the model's parameters.

---

## Q16. What happens during pretraining?

The model reads massive text datasets, predicts the next token, calculates loss against the actual token, backpropagates the error, and repeatedly updates its parameters.

---

## Q17. Why is LLM pretraining self-supervised?

Because the training text supplies its own labels: the next token already present in the sequence becomes the prediction target.

---

## Q18. Base model vs instruct model?

A base model is primarily trained to predict/continue text. An instruct model receives post-training that teaches it to follow instructions and behave like an assistant.

---

## Q19. Pretraining vs SFT?

Pretraining uses huge raw text corpora to learn general language patterns. SFT uses curated instruction-response examples to teach instruction-following behavior.

---

## Q20. What is RLHF?

RLHF uses human preferences to improve model behavior. Humans rank model responses, a reward model learns those preferences, and reinforcement learning is used to improve the main LLM.

---

## Q21. What is a reward model?

A separate model that assigns a preference score to a prompt-response pair.

---

## Q22. What is PPO?

PPO is a reinforcement-learning algorithm mentioned in the source material for updating an LLM against reward signals during classic RLHF.

---

## Q23. Why use KL-divergence in RLHF?

To keep the RL-updated model from moving too far away from the SFT model and reduce unstable behavior such as reward hacking.

---

## Q24. Training vs inference?

Training updates weights using loss and backpropagation. Inference uses fixed trained weights to generate outputs.

---

## Q25. What does temperature control?

Randomness. Lower temperature makes output more deterministic; higher temperature increases diversity and randomness.

---

## Q26. Top-K vs Top-P?

Top-K keeps a fixed number of highest-probability tokens. Top-P keeps a variable number of tokens whose cumulative probability reaches a chosen threshold.

---

## Q27. Frequency penalty vs presence penalty?

Frequency penalty discourages repeated use based on how often a token has appeared. Presence penalty encourages use of tokens that have not appeared yet.

---

## Q28. What is greedy decoding?

Always selecting the highest-probability next token.

---

## Q29. What is beam search?

A decoding strategy that maintains multiple candidate sequences and repeatedly keeps the most promising ones, rather than committing immediately to one path.

---

## Q30. What is Mixture of Experts?

MoE is a Transformer architecture with multiple expert networks where a router selects only a subset of experts for each token.

---

## Q31. What does the MoE router do?

It scores available experts and selects the top experts to process each token.

---

## Q32. What is knowledge distillation?

Knowledge distillation trains a smaller Student model to imitate a larger Teacher model.

---

## Q33. Soft-label vs hard-label distillation?

Soft-label distillation uses the Teacher's full probability distribution. Hard-label distillation uses the Teacher's selected output/token.

---

## Q34. What does an LLM actually store?

The attached Word material describes learned information as statistical patterns distributed across model weights, rather than a structured database of facts or symbolic rules.

---

## Q35. Explain an LLM end-to-end in one minute.

An LLM first tokenizes the user input and converts the tokens to embeddings with positional information. These representations pass through multiple Transformer layers containing self-attention and feed-forward networks. Self-attention uses Query, Key and Value vectors to determine relationships between tokens. The final hidden representation is projected to logits over the vocabulary, and softmax converts the logits into probabilities. A decoding strategy such as greedy, Top-K or Top-P selects the next token. The token is appended to the context and the process repeats until generation stops. The model originally learned its language abilities during pretraining through next-token prediction, then SFT taught it to follow instructions, and preference/alignment tuning further improved its behavior.

---

# 56. Interview Preparation Priority

Study these in this order.

## Priority 1 — Must Know

- What is an LLM?
- Token
- Tokenization
- Embedding
- Transformer
- Attention
- Self-Attention
- Q / K / V
- Multi-Head Attention
- Positional Encoding
- Parameters / weights / bias
- Logits
- Softmax
- Next-token prediction
- Training vs inference
- Pretraining
- Cross-entropy loss
- Backpropagation
- SFT / Instruction tuning
- Base vs Instruct model
- RLHF
- Reward Model
- Temperature
- Top-K / Top-P

## Priority 2 — Good Interview Knowledge

- Residual connections
- Layer normalization
- FFN
- AdamW
- PPO
- KL-divergence
- Greedy decoding
- Beam search
- Presence / frequency penalties
- Reasoning fine-tuning
- MoE and Router
- Distillation

## Priority 3 — Know Only at High Level

- GRPO
- Contrastive search
- Co-distillation
- Local runtimes: Ollama, LM Studio, vLLM, LlamaCPP
- Newer preference approaches such as DPO / KTO / RLAIF mentioned in the Word notes

---

# 57. 15-Minute Final Revision Sheet

Before an interview, remember this chain:

```text
LLM
 ↓
Transformer-based neural network
 ↓
Text is tokenized
 ↓
Tokens become embeddings
 ↓
Position information is added
 ↓
Transformer layers
 ↓
Self-attention uses Q/K/V
 ↓
Multi-head attention learns multiple relationships
 ↓
FFN refines representations
 ↓
Linear layer produces logits
 ↓
Softmax produces next-token probabilities
 ↓
Top-K / Top-P / temperature influence generation
 ↓
Next token selected
 ↓
Repeat autoregressively
```

Training:

```text
Raw Text
 ↓
Next-token prediction
 ↓
Cross-entropy loss
 ↓
Backpropagation
 ↓
Optimizer updates weights
 ↓
Base Model
 ↓
SFT
 ↓
Instruction Model
 ↓
RLHF / Alignment
 ↓
Aligned Assistant Model
```

And remember these five interview distinctions:

```text
Tokenization  != Embedding
Logits        != Probabilities
Training      != Inference
Base Model    != Instruct Model
Top-K         != Top-P
```

---

# 58. Final One-Line Definitions

**LLM:** Transformer-based model trained to predict tokens and generate language.

**Transformer:** Attention-based architecture for processing token sequences.

**Token:** Basic unit of text processed by an LLM.

**Embedding:** Numeric vector representation of a token.

**Attention:** Mechanism for determining which tokens are relevant.

**Self-Attention:** Attention among tokens in the same sequence.

**Q/K/V:** Vectors used to calculate contextual relevance.

**Logit:** Raw score for a possible output token.

**Softmax:** Converts logits to probabilities.

**Pretraining:** Learning general language using next-token prediction.

**SFT:** Teaching a pretrained model to follow instructions.

**RLHF:** Improving model behavior using human preference feedback.

**Inference:** Generating output with an already-trained model.

**Temperature:** Controls generation randomness.

**Top-K:** Keep K most likely next tokens.

**Top-P:** Keep tokens covering cumulative probability P.

**MoE:** Model architecture routing tokens to selected expert networks.

**Distillation:** Training a smaller Student model from a larger Teacher model.

---

## Recommended Study Approach

1. First understand the **end-to-end LLM flow**.
2. Then learn the **Transformer components**.
3. Then understand **how training works**.
4. Then study **SFT and RLHF**.
5. Finally learn **generation parameters and basic advanced concepts**.
6. Practice the **35 interview Q&As** above until you can answer each in 30–60 seconds.
