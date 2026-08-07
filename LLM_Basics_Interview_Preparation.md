
# LLM Basics — Interview Preparation

> **How to use this guide:** The questions below are kept exactly as in the original interview-preparation material.  
> The answers are intentionally written in a natural, conversational style so you can **understand the flow and explain it in your own words**, rather than memorizing textbook definitions.
>
> A good interview pattern is:
>
> **Simple idea → Why it is needed → Small example → Connect it to the next LLM concept**

---

## Q1. What is an LLM?

An LLM, or Large Language Model, is basically a neural-network model that has learned language patterns from a very large amount of text.

The easiest way I think about it is that an LLM is trained mainly to **predict what token should come next**. For example, if I give it:

```text
The capital of France is ...
```

it has learned from training that `Paris` is a very likely next token.

Of course, modern LLMs do much more than complete sentences. Because they have learned patterns from huge amounts of text, they can answer questions, summarize documents, generate code, translate text, and carry on conversations.

Most modern LLMs are built using the **Transformer architecture**. So the overall idea is:

```text
Large amount of text
        ↓
Transformer model learns patterns
        ↓
Next-token prediction
        ↓
Generate useful language
```

This naturally connects to the next question, because **next-token prediction is the basic mechanism behind how an LLM generates text**.

---

## Q2. What is next-token prediction?

Next-token prediction means the model looks at all the tokens it has already received and tries to predict what token is most likely to come next.

For example:

```text
The sky is ...
```

The model may internally assign probabilities such as:

```text
blue   → 80%
clear  → 10%
dark   → 5%
other  → 5%
```

It then selects or samples one of those possible tokens.

The important point is that the model does not generate an entire paragraph in one shot. It generates **one token at a time**.

For example:

```text
Once
↓
Once upon
↓
Once upon a
↓
Once upon a time
```

Every new token is added to the existing context, and then the model predicts again.

So next-token prediction leads directly to another important concept: **What exactly is a token?**

---

## Q3. What is a token?

A token is simply a small unit of text that the LLM works with.

People sometimes assume one token always means one word, but that is not necessarily true. A token can be:

- a full word,
- part of a word,
- punctuation,
- or another small character sequence.

For example, depending on the tokenizer:

```text
playing
```

might be represented roughly as:

```text
play + ing
```

The reason we need tokens is that a neural network cannot directly process normal text like humans do. We first break the text into manageable pieces.

The flow is:

```text
Text
 ↓
Tokens
 ↓
Token IDs
 ↓
Embeddings
```

So after understanding tokens, the next important distinction is **tokenization versus embedding**.

---

## Q4. Tokenization vs embedding?

I separate these two concepts because they happen one after another.

**Tokenization** is about breaking the text into pieces.

For example:

```text
"I am learning AI"
```

might become something like:

```text
["I", "am", "learning", "AI"]
```

Each token then gets a numeric token ID.

But a token ID is only an identifier. It does not by itself carry a rich mathematical meaning for the neural network.

That is where **embeddings** come in.

An embedding converts each token into a vector of numbers, something conceptually like:

```text
AI
↓
[0.21, -0.54, 0.83, ...]
```

So I remember it as:

```text
Tokenization = divide text into units

Embedding = represent those units as meaningful numerical vectors
```

Once we have embeddings, those representations are sent into the **Transformer**.

---

## Q5. What is a Transformer?

A Transformer is the neural-network architecture used by most modern LLMs.

The main reason Transformers became so important is that they use **attention** to understand relationships between tokens.

For example, consider:

```text
The animal didn't cross the road because it was tired.
```

To understand the word `it`, the model needs to recognize that it probably refers to `animal`, not `road`.

Attention helps the model learn that relationship.

A simplified Transformer flow is:

```text
Token embeddings
      ↓
Self-attention
      ↓
Feed-forward network
      ↓
Repeat through many Transformer layers
      ↓
Context-aware representations
```

Compared with older RNN or LSTM approaches, Transformers are also much easier to scale because training can be parallelized much more effectively.

The core idea inside a Transformer is therefore **self-attention**.

---

## Q6. What is self-attention?

Self-attention is the mechanism that allows every token to look at other tokens in the same input and decide which ones are important for understanding its context.

For example:

```text
The dog chased the ball because it was excited.
```

When the model processes the word `it`, it looks at the other words and tries to determine which ones are most relevant.

It may give more attention to:

```text
dog
excited
```

than to unrelated words.

So instead of treating every word independently, self-attention builds a **context-aware representation**.

This is important because the same word can have different meanings in different sentences.

For example:

```text
river bank
```

and

```text
bank account
```

The word `bank` is the same, but attention to the surrounding words helps the model understand the intended meaning.

Technically, self-attention is calculated using **Query, Key and Value vectors**, which leads to the next question.

---

## Q7. Explain Query, Key and Value.

I usually explain Query, Key and Value using a search analogy.

For every token, the model creates three different vectors:

- **Query** — What information am I looking for?
- **Key** — What kind of information do I contain?
- **Value** — What actual information can I contribute?

Suppose the sentence is:

```text
The animal didn't cross the road because it was tired.
```

When processing the token `it`, the model's Query is compared against the Keys of other tokens.

If the Query for `it` matches the Key for `animal` strongly, then `animal` receives a high attention score.

The flow is roughly:

```text
Query
  ↓
Compare with Keys
  ↓
Attention scores
  ↓
Softmax converts them to attention weights
  ↓
Use those weights to combine Values
  ↓
Context-aware output
```

So Q and K decide **where to pay attention**, while V contains the **information that is actually passed forward**.

Once Q, K and V are understood, it becomes easier to understand why token order still needs to be added separately through **positional encoding**.

---

## Q8. Why is positional encoding needed?

Transformers use attention to compare tokens, but attention by itself does not naturally understand the order of those tokens.

For example:

```text
Dog bites man
```

and

```text
Man bites dog
```

contain almost the same words, but the meaning is completely different because the order changed.

So the model needs information about whether a token appears first, second, third, and so on.

That position information is combined with the token representation.

Conceptually:

```text
Token embedding
      +
Position information
      ↓
Position-aware representation
```

This allows the Transformer to understand both:

1. **what the token means**, through its embedding, and
2. **where the token appears**, through positional information.

After that, the model can apply attention. And instead of doing only one type of attention calculation, Transformers usually use **multi-head attention**.

---

## Q9. Why multi-head attention?

A single attention mechanism may learn one kind of relationship between tokens, but language contains many relationships at the same time.

For example, in one sentence the model may need to understand:

- which word is the subject,
- which word is the verb,
- which noun a pronoun refers to,
- semantic similarity,
- and long-distance dependencies.

Multi-head attention allows several attention calculations to happen in parallel.

You can think of it like several people reading the same sentence with different jobs:

```text
Head 1 → looks for grammatical relationships
Head 2 → looks for semantic relationships
Head 3 → looks for long-range context
Head 4 → looks for other useful patterns
```

The outputs from those heads are then combined.

So multi-head attention gives the model a richer understanding than relying on only one attention pattern.

All of this operates on numerical representations called **embeddings**, which is why embeddings are so important.

---

## Q10. What is an embedding?

An embedding is a numerical vector that represents a token in a form the neural network can work with.

For example, the word:

```text
cat
```

may internally be represented as something like:

```text
[0.42, -0.18, 0.76, ...]
```

The individual numbers are not something we normally interpret manually. The important part is that the complete vector captures learned relationships.

Words or concepts that are related can have related representations in the model's vector space.

So instead of the network receiving the literal string `"cat"`, it receives a mathematical representation.

The flow is:

```text
Text
 ↓
Token
 ↓
Token ID
 ↓
Embedding vector
 ↓
Transformer
```

During training, the model learns how to use these embeddings along with many other learned values. Those learned values are what we call **parameters**.

---

## Q11. What are parameters in an LLM?

Parameters are the numerical values the model learns during training.

The most common examples are **weights and biases**.

You can think of parameters as the model's adjustable internal settings.

At the beginning of training, the model does not know useful language patterns. It makes predictions, compares them with the correct next token, calculates an error, and then adjusts its parameters.

This happens repeatedly:

```text
Prediction
   ↓
Calculate error
   ↓
Backpropagation
   ↓
Adjust parameters
   ↓
Better future prediction
```

Large models may contain billions of these learned parameters.

The important point is that when we say a model has "learned something," that learning is represented through patterns distributed across these parameters.

After the Transformer processes the input using these learned parameters, the final layer produces **logits**.

---
# LLM Generation Parameters — Simple Interview Notes

Plain-English explanations for each parameter. No formulas — just what it does, why it matters, and a one-line way to say it in an interview.

---

## 0. The Big Picture (say this first if asked to explain generation)

> "When an LLM generates text, at every single step it's not just picking the 'best' next word — it's looking at a ranked list of possible next words, each with a confidence score. These parameters control **how that list gets narrowed down** and **which word actually gets picked** from it. A few of them also control **when the model should stop talking**."

That's really it. Everything below is a variation on one of those two ideas: *narrowing the choices* or *deciding when to stop*.

---

## 1. Max Tokens

**One-liner:** "The word limit for the response."

**Simple explanation:**
It's a hard cutoff on how long the response can be. Once the model hits that limit, it stops immediately — even if it was in the middle of a sentence.

**Why it matters:**
- Too low → answer gets cut off awkwardly.
- Too high → you might pay for/wait for more than you need.

**Interview soundbite:**
> "Max tokens is just a length cap — it doesn't change what the model says, only how much of it you let it finish saying."

---

## 2. Temperature

**One-liner:** "How random or safe the model's word choices are."

**Simple explanation:**
Think of the model as having a shortlist of possible next words, each more or less "confident." Temperature controls how much it's willing to gamble on a less-obvious choice.

- **Low temperature** → always picks the safest, most obvious word. Predictable, focused, sometimes boring.
- **High temperature** → more willing to pick surprising, creative words. More interesting, but riskier — can go off the rails.

**Analogy:**
> "It's like asking someone a question — at low temperature they give you the textbook answer every time. At high temperature they start improvising and going off-script."

**When to use what:**
- Low temperature → coding, facts, customer support.
- High temperature → brainstorming, creative writing, poetry.

**Interview soundbite:**
> "Temperature controls the model's willingness to take risks with word choice — low is predictable, high is creative but noisier."

---

## 3. Top-k

**One-liner:** "Only consider the top K most likely next words."

**Simple explanation:**
Before picking a word, the model narrows its options down to a fixed number — say, the top 10 most likely candidates — and only chooses from that shortlist. Everything else is thrown away, no matter how the rest of the list looks.

**Analogy:**
> "It's like a hiring manager who always looks at exactly the top 5 resumes, no matter how many great candidates there really are or aren't."

**Trade-off:**
- Too small K → responses feel repetitive and robotic.
- Reasonable K → keeps things focused without being too rigid.

**Interview soundbite:**
> "Top-k limits the model to picking from a fixed number of top candidates — it's a simple, blunt way to cut out unlikely words."

---

## 4. Top-p (Nucleus Sampling)

**One-liner:** "Only consider enough top words to cover, say, 90% of the confidence."

**Simple explanation:**
Instead of a fixed number of words like top-k, top-p keeps adding the next most-likely words until their combined confidence adds up to a target percentage (e.g., 90%). Then it picks from just that group.

**Why it's smarter than top-k:**
It automatically adjusts. If the model is very sure about the next word, the shortlist might just be 1–2 words. If the model is unsure, the shortlist naturally grows to include more options. Top-k can't do that — it always uses the same fixed count.

**Analogy:**
> "Instead of always looking at exactly 5 resumes, the hiring manager keeps reading resumes until they've covered '90% of the good candidates' — sometimes that's 2 resumes, sometimes it's 15, depending on how competitive the pool is."

**Interview soundbite:**
> "Top-p is the adaptive version of top-k — instead of a fixed count of words, it uses a fixed amount of confidence."

---

## 5. Frequency Penalty

**One-liner:** "Discourages the model from repeating the same words too much."

**Simple explanation:**
The more a word has already been used in the response, the more the model gets discouraged from using it again. The penalty builds up the more times a word repeats.

**Analogy:**
> "Like a teacher telling a student, 'You've already used the word "amazing" four times — try something else.' The more you overuse a word, the stronger the nudge to stop."

**When to use it:**
- Turn it up for summaries/reports where you don't want redundant phrasing.
- Turn it down (or make it negative) if some repetition is intentional, like in poetry.

**Interview soundbite:**
> "Frequency penalty punishes words based on how *often* they've already shown up — the more repeats, the bigger the penalty."

---

## 6. Presence Penalty

**One-liner:** "Encourages the model to bring in new words/topics it hasn't used yet."

**Simple explanation:**
This one doesn't care *how many times* a word has been repeated — it only checks *has this word shown up at all yet?* If yes, it gets a small, flat penalty, encouraging the model to explore something new instead.

**How it's different from frequency penalty:**
- Frequency penalty = "you've used this word 5 times, stop."
- Presence penalty = "you've used this word even once — try mixing in something new."

**Analogy:**
> "It's like a brainstorming coach saying 'you already mentioned that idea — what's something you haven't brought up yet?' — it doesn't matter if you mentioned it once or five times, the nudge is the same."

**When to use it:**
Great for brainstorming or idea generation, where you want breadth of topics rather than depth on one thing.

**Interview soundbite:**
> "Presence penalty nudges the model toward new topics it hasn't touched yet, regardless of how many times it repeated something else."

---

## 7. Stop Sequences

**One-liner:** "Specific words/symbols that tell the model 'stop right here.'"

**Simple explanation:**
You give the model a list of trigger strings (like `"\n\n"`, `"END"`, or `"}"`). The moment the model produces one of those, generation halts immediately — no matter what else it might have been about to say.

**Why it matters:**
Super useful when you need clean, structured output — like JSON — where you don't want the model to add extra commentary after the actual answer is done.

**Analogy:**
> "It's like giving someone a magic word — the second they say it, they have to stop talking, mid-thought if necessary."

**Interview soundbite:**
> "Stop sequences are a hard, rule-based stop trigger — unlike max tokens, which is just a length cap, this stops generation based on *content*, not count."

---

## Quick Comparison Table (great for a whiteboard answer)

| Parameter | What it controls | Simple mental model |
|---|---|---|
| Max tokens | Response length | "Word limit" |
| Temperature | Randomness/creativity | "How much it gambles on unusual words" |
| Top-k | Candidate pool (fixed size) | "Only look at the top K options" |
| Top-p | Candidate pool (adaptive size) | "Only look at options covering X% confidence" |
| Frequency penalty | Repetition (scales with count) | "Stop repeating that word so much" |
| Presence penalty | Topic diversity (flat, one-time) | "Bring in something new" |
| Stop sequences | Hard stop trigger | "Stop the moment you see this" |

---

## Likely Interview Questions + Simple Answers

**Q: What's the difference between top-k and top-p?**
> "Top-k always looks at a fixed number of word choices. Top-p instead looks at however many words it takes to cover a target confidence level — so it adjusts automatically depending on how sure the model is."

**Q: What's the difference between frequency penalty and presence penalty?**
> "Frequency penalty cares about *how many times* something's been repeated — the more repeats, the bigger the discouragement. Presence penalty is simpler — it just checks *has this been said before, yes or no* — and nudges toward new material either way."

**Q: When would you use a low vs. high temperature?**
> "Low temperature for anything that needs to be accurate and consistent — like code or factual Q&A. High temperature for anything creative — like brainstorming or storytelling — where you want variety over precision."

**Q: Why would you use stop sequences instead of just max tokens?**
> "Max tokens just cuts things off after a certain length, which can chop a sentence in half. Stop sequences let you end generation cleanly at a meaningful point — like right after a closing bracket in JSON — so the output stays clean and structured."

**Q: Can you combine these parameters?**
> "Yes, they're not exclusive — a typical setup might use a moderate temperature together with top-p to balance creativity and coherence, plus stop sequences to keep the output clean."

---

## Q12. What are logits?

Logits are the raw scores the model produces for possible next tokens.

Suppose the current prompt is:

```text
The capital of France is
```

The model may produce raw scores like:

```text
Paris   → 8.2
London  → 4.1
Berlin  → 2.8
```

These values tell us that the model prefers `Paris`, but they are not probabilities yet.

They do not have to add up to 1 or 100%.

So the generation flow at this stage is:

```text
Transformer output
      ↓
Linear output layer
      ↓
Logits
```

To turn those raw scores into something easier to interpret as a probability distribution, we use **softmax**.

---

## Q13. What does softmax do?

Softmax takes the raw logits and converts them into a probability distribution.

For example, if the logits favor `Paris`, softmax might give:

```text
Paris   → 0.80
London  → 0.15
Berlin  → 0.05
```

Now the values are normalized and add up to 1.

So I remember:

```text
Logits = raw scores

Softmax = probabilities
```

During **inference**, those probabilities are used to decide which next token to generate.

During **training**, those probabilities are compared with the correct next token. That comparison is measured using **cross-entropy loss**.

So softmax connects the forward pass directly to the training loss.

---

## Q14. What is cross-entropy loss?

Cross-entropy loss tells us how wrong the model's next-token prediction was.

Suppose the correct next token is:

```text
Paris
```

If the model gives `Paris` a probability of 95%, that is a good prediction, so the loss will be low.

If it gives `Paris` only 2% probability and strongly predicts another word, the loss will be high.

The simple idea is:

```text
Correct token gets high probability
           ↓
        Low loss
```

and:

```text
Correct token gets low probability
           ↓
        High loss
```

Loss by itself only tells us **how bad the prediction was**.

The model still needs to know **which parameters should be changed and in what direction**.

That is the role of backpropagation.

---

## Q15. What is backpropagation?

Backpropagation is the process used during training to figure out how the model's parameters contributed to the prediction error.

The flow is:

```text
Input
 ↓
Forward pass
 ↓
Prediction
 ↓
Calculate loss
 ↓
Backpropagation
 ↓
Calculate gradients
 ↓
Optimizer updates weights
```

The **gradient** tells us approximately how changing a parameter would affect the loss.

Then an optimizer, such as AdamW in many Transformer training setups, uses those gradients to update the weights.

So I would summarize the training loop as:

```text
Forward pass → Loss → Backpropagation → Weight update
```

This happens repeatedly during **pretraining**, which is the first major stage in building an LLM.

---

## Q16. What happens during pretraining?

Pretraining is the stage where the model learns general language patterns from a huge amount of text.

The model sees text from sources such as books, websites, code, articles, and other datasets.

It repeatedly performs next-token prediction.

For example:

```text
Input:
The sky is

Actual next token:
blue
```

The model predicts a probability distribution.

Then:

```text
Prediction
 ↓
Compare with "blue"
 ↓
Calculate cross-entropy loss
 ↓
Backpropagation
 ↓
Optimizer updates weights
```

This happens at enormous scale.

Over time, the model learns patterns involving grammar, sentence structure, semantics, code, relationships between concepts, and statistical knowledge.

The important thing is that humans do not need to manually label every training example. That is why this process is called **self-supervised learning**.

---

## Q17. Why is LLM pretraining self-supervised?

LLM pretraining is called self-supervised because the training data itself provides the correct answer.

For example, suppose the original text already contains:

```text
The sky is blue.
```

We can create a training example automatically:

```text
Input:
The sky is

Target:
blue
```

No person has to manually label `blue` as the correct answer. It already exists in the text.

We can repeat the same idea across huge amounts of data.

So:

```text
Existing text
     ↓
Previous tokens become input
     ↓
Next token becomes target
```

That makes it possible to create extremely large training datasets automatically.

After pretraining, we get a **base model**. But a base model is not necessarily a good conversational assistant yet, which is why we distinguish between base and instruct models.

---

## Q18. Base model vs instruct model?

A base model has mainly learned how language works through next-token prediction.

It is very good at continuing text, but it may not naturally understand that we want it to behave like a helpful assistant.

For example, if we ask:

```text
What is the capital of France?
```

a base model may simply continue the text in a style similar to its training data.

An instruct model has gone through additional post-training, such as supervised fine-tuning, so it learns:

```text
User gives instruction
        ↓
Understand the request
        ↓
Produce a useful assistant-style response
```

So I think of it as:

```text
Pretraining
   ↓
Base model
   ↓
Instruction tuning / SFT
   ↓
Instruct model
```

This naturally leads to the difference between **pretraining and SFT**.

---

## Q19. Pretraining vs SFT?

The main difference is what we are trying to teach the model and what kind of data we use.

During **pretraining**, we use massive amounts of general text so the model learns language through next-token prediction.

For example:

```text
The capital of France is → Paris
```

During **Supervised Fine-Tuning, or SFT**, we use curated instruction-response examples.

For example:

```text
User:
Explain gravity simply.

Assistant:
Gravity is the force that...
```

Pretraining teaches:

```text
How language works
```

SFT teaches:

```text
How to follow an instruction and respond like an assistant
```

Both can still involve next-token prediction. The major difference is the training data and the behavior we are trying to create.

Even after SFT, we may want to further improve which kinds of responses humans prefer. That is where **RLHF** comes in.

---

## Q20. What is RLHF?

RLHF stands for Reinforcement Learning from Human Feedback.

After SFT, a model can follow instructions, but some answers may still be less useful, unsafe, poorly written, or simply not what users prefer.

RLHF introduces human preference information.

A simplified flow is:

```text
Prompt
 ↓
Model generates multiple answers
 ↓
Humans rank the answers
 ↓
Learn what humans prefer
 ↓
Use that signal to improve the model
```

For example:

```text
Response A → preferred
Response B → acceptable
Response C → poor
```

Instead of asking humans to evaluate every model output forever, the classic RLHF approach trains another model to learn those preferences.

That model is called a **reward model**.

---

## Q21. What is a reward model?

A reward model is like an automated judge that has learned from human preferences.

It receives something like:

```text
Prompt + Model Response
```

and produces a score.

For example:

```text
Helpful answer → high reward
Poor answer    → low reward
```

The reward model itself is trained using examples where humans ranked responses.

So the overall RLHF idea becomes:

```text
Humans rank responses
       ↓
Train reward model
       ↓
Reward model scores new responses
       ↓
Use scores to improve the LLM
```

In classic RLHF, one of the algorithms historically used to update the LLM from this reward signal is **PPO**.

---

## Q22. What is PPO?

PPO stands for Proximal Policy Optimization.

For a general GenAI interview, I would not go deeply into the mathematics unless the interviewer asks.

The practical idea is that PPO is a reinforcement-learning algorithm that can update the language model based on reward signals.

In classic RLHF:

```text
LLM generates response
        ↓
Reward model scores it
        ↓
PPO uses reward signal
        ↓
Adjust LLM behavior
```

If a certain type of response receives a better reward, the training process tries to make that type of behavior more likely.

But there is a risk: if we optimize too aggressively only for reward, the model can change too much or exploit weaknesses in the reward model.

That is why classic RLHF also uses ideas such as **KL-divergence control**.

---

## Q23. Why use KL-divergence in RLHF?

KL-divergence is used as a control mechanism so the RLHF-trained model does not move too far away from the original supervised fine-tuned model.

Imagine we have:

```text
SFT model
   ↓
RLHF optimization
   ↓
Improved model
```

We want the model to improve, but we do not want it to change so aggressively that it loses useful language behavior or starts exploiting the reward model.

So conceptually the objective is:

```text
Improve reward
     +
Stay reasonably close to the original model
```

KL-divergence helps measure that difference.

So in an interview, I would connect it to **training stability and preventing excessive model drift or reward hacking**.

That entire RLHF process happens during training/post-training. At runtime, we normally perform **inference**, which is different.

---

## Q24. Training vs inference?

Training is when the model learns.

Inference is when we use the already-trained model.

During training:

```text
Input
 ↓
Prediction
 ↓
Loss
 ↓
Backpropagation
 ↓
Update weights
```

So the parameters change.

During inference:

```text
User prompt
 ↓
Forward pass
 ↓
Logits
 ↓
Probabilities
 ↓
Generate token
```

The model's learned weights normally stay fixed.

A simple memory trick is:

```text
Training = learning

Inference = using what was learned
```

During inference, we can still control **how the model chooses tokens** using generation settings such as temperature, Top-K and Top-P.

---

## Q25. What does temperature control?

Temperature controls how conservative or random token selection is during generation.

The model already has a probability distribution for possible next tokens.

With a **low temperature**, the distribution becomes more focused on high-probability choices.

That usually gives:

- more predictable output,
- more consistent output,
- less randomness.

With a **higher temperature**, the model gives lower-probability alternatives more of a chance.

That gives:

- more variation,
- more creativity,
- but also more unpredictability.

So I use:

```text
Low temperature  → focused / predictable

High temperature → diverse / creative
```

Temperature works with the probability distribution. Two other common ways to restrict which tokens can be sampled are **Top-K and Top-P**.

---

## Q26. Top-K vs Top-P?

Both Top-K and Top-P limit the candidate tokens the model considers during generation, but they do it differently.

### Top-K

Top-K keeps a fixed number of the highest-probability tokens.

For example:

```text
Top-K = 5
```

means only the five most likely next tokens are considered.

### Top-P

Top-P keeps enough high-probability tokens to reach a chosen cumulative probability.

For example:

```text
Top-P = 0.90
```

means we keep the smallest group of likely tokens whose total probability reaches roughly 90%.

The easiest way I remember it is:

```text
Top-K = count

Top-P = probability mass
```

Top-P is more adaptive because the number of candidate tokens can change depending on how confident the model is.

Another type of generation control deals with repeated words or topics: **frequency and presence penalties**.

---

## Q27. Frequency penalty vs presence penalty?

Both are related to repetition, but they look at repetition differently.

**Frequency penalty** considers how many times a token has already appeared.

If something has been repeated many times, the penalty becomes stronger.

So I think:

```text
Frequency = How often?
```

**Presence penalty** mainly considers whether a token has already appeared at all.

It can encourage the model to introduce something new rather than staying with the same vocabulary or topic.

So I remember:

```text
Presence = Has it appeared?
```

These are generation-time controls, just like temperature and Top-P.

Another important decision during generation is the **decoding strategy**. The simplest one is greedy decoding.

---

## Q28. What is greedy decoding?

Greedy decoding means that at every generation step, the model simply chooses the token with the highest probability.

For example:

```text
Paris   → 70%
London  → 20%
Berlin  → 10%
```

Greedy decoding always selects:

```text
Paris
```

The advantage is that it is simple and deterministic.

But the disadvantage is that choosing the locally best token at every step does not always produce the best overall sentence, and the output can sometimes become repetitive.

So greedy decoding follows only one path.

**Beam search**, on the other hand, keeps several possible paths alive at the same time.

---

## Q29. What is beam search?

Beam search is a decoding strategy where the model keeps multiple candidate sequences instead of immediately committing to only one.

For example, instead of keeping just:

```text
The cat is ...
```

it might keep several promising continuations.

Conceptually:

```text
Start
 ↓
Generate multiple candidates
 ↓
Score them
 ↓
Keep the best few
 ↓
Expand those candidates
 ↓
Repeat
```

The number of candidates kept is called the beam width.

This can be useful for tasks where the quality of the complete sequence matters, such as traditional machine translation.

Compared with greedy decoding:

```text
Greedy → one path

Beam search → multiple promising paths
```

Generation strategies describe how a dense model chooses output tokens. Another architectural idea used to make very large models more compute-efficient is **Mixture of Experts**.

---

## Q30. What is Mixture of Experts?

Mixture of Experts, or MoE, is a model architecture where we have multiple specialized neural-network "experts," but we do not use every expert for every token.

A simple analogy is a company.

Suppose a company has specialists in:

- finance,
- legal,
- engineering,
- sales.

If a technical problem arrives, we do not ask every employee to solve it. We route it to the most relevant specialists.

MoE does something similar.

```text
Token
 ↓
Router
 ↓
Select a few experts
 ↓
Selected experts process token
 ↓
Combine their outputs
```

The benefit is that the model can have a very large total number of parameters while activating only part of the model for each token.

The component that decides which experts receive a token is called the **MoE router**.

---

## Q31. What does the MoE router do?

The MoE router decides which experts should process each token.

For a token, the router produces scores for the available experts.

Then it may select the top one, top two, or another small subset.

Conceptually:

```text
Token
 ↓
Router scores experts
 ↓
Expert 1 → 0.10
Expert 2 → 0.75
Expert 3 → 0.05
Expert 4 → 0.65
 ↓
Choose top experts
 ↓
Process token
```

So the router is essentially the traffic controller of an MoE model.

One challenge is that we do not want every token to go to the same expert, because then some experts would be overloaded while others would not learn enough. That is why MoE systems often use load-balancing techniques.

MoE tries to make large models more efficient. Another technique for efficiency is to create a **smaller model from a larger model**, which is called knowledge distillation.

---

## Q32. What is knowledge distillation?

Knowledge distillation is a technique where a large model, called the **Teacher**, is used to train a smaller model, called the **Student**.

The idea is:

```text
Large Teacher Model
        ↓
Transfers useful behavior/knowledge
        ↓
Smaller Student Model
```

Why do this?

Because a very large model may give excellent results but can be:

- expensive,
- slower,
- memory intensive,
- difficult to deploy.

A smaller student model can be cheaper and faster while still learning useful behavior from the teacher.

The teacher can transfer information in different ways. Two common approaches are **soft-label and hard-label distillation**.

---

## Q33. Soft-label vs hard-label distillation?

The difference is how much information from the Teacher model is given to the Student.

### Soft-label distillation

The Student learns from the Teacher's complete probability distribution.

For example:

```text
Paris   → 70%
London  → 20%
Berlin  → 10%
```

This gives the Student more information because it can see not only what the Teacher selected, but also how confident the Teacher was about alternatives.

### Hard-label distillation

The Student receives only the Teacher's selected answer, for example:

```text
Paris
```

This is simpler and cheaper to store, but it loses some information about the Teacher's probability distribution.

So:

```text
Soft label → full distribution / richer signal

Hard label → final selected output / simpler signal
```

The fact that models can transfer behavior this way also raises an important question: **What information is actually stored inside an LLM?**

---

## Q34. What does an LLM actually store?

I would be careful not to describe an LLM as if it were a normal database.

A database might explicitly store something like:

```text
France → Paris
Japan  → Tokyo
India  → New Delhi
```

An LLM does not normally store knowledge as clean rows and columns like that.

Instead, during training, patterns are learned across billions of model parameters.

So the model's knowledge is **distributed across its weights**.

That is why an LLM can often answer factual questions, but it is not doing a guaranteed database lookup.

It is generating an answer based on learned statistical patterns.

That also helps explain hallucinations: the model can sometimes generate an answer that fits learned language patterns but is factually incorrect.

To bring all of these ideas together, the most useful final interview question is explaining the complete LLM flow end to end.

---

## Q35. Explain an LLM end-to-end in one minute.

I would explain the complete flow like this:

When a user sends a prompt, the first step is **tokenization**. The text is split into tokens, and each token is mapped to a token ID.

Those token IDs are converted into **embeddings**, which are numerical vectors the neural network can process. We also include **positional information** so the model understands the order of the tokens.

Those representations then pass through many **Transformer layers**.

Inside the Transformer, **self-attention** allows each token to look at other tokens and determine which ones are relevant. The attention calculation uses **Query, Key and Value vectors**. Multi-head attention performs several types of these attention calculations in parallel, and feed-forward networks further transform the token representations.

After all Transformer layers are processed, the final representation goes through an output layer that produces **logits** for all possible next tokens.

Softmax or the model's output scoring pipeline gives us a probability distribution over possible next tokens.

Then a decoding or sampling strategy—such as greedy decoding, Top-K or Top-P, often influenced by temperature—chooses the next token.

That token is added to the context, and the entire process repeats one token at a time until the response is complete.

So the inference flow is:

```text
Prompt
 ↓
Tokenization
 ↓
Token IDs
 ↓
Embeddings + Position Information
 ↓
Transformer Layers
 ↓
Self-Attention using Q / K / V
 ↓
Feed-Forward Processing
 ↓
Logits
 ↓
Next-token probabilities
 ↓
Decoding / Sampling
 ↓
Next Token
 ↓
Append to Context
 ↓
Repeat
```

And if the interviewer asks how the model learned to do this, I connect it to the training lifecycle:

```text
Massive text data
 ↓
Pretraining using next-token prediction
 ↓
Cross-entropy loss
 ↓
Backpropagation
 ↓
Optimizer updates weights
 ↓
Base Model
 ↓
Supervised Fine-Tuning (SFT)
 ↓
Instruction-following Model
 ↓
Preference / Alignment Training such as RLHF
 ↓
Assistant Model
```

That connects the full story—from **how the model learns** to **what happens when a user actually sends a prompt**.

---

# Interview Revision Flow

Instead of memorizing 35 separate answers, remember how the topics connect.

## 1. Input Processing

```text
User Prompt
    ↓
Tokenization
    ↓
Tokens
    ↓
Token IDs
    ↓
Embeddings
    +
Positional Information
```

## 2. Transformer Processing

```text
Embeddings
    ↓
Transformer
    ↓
Self-Attention
    ↓
Query / Key / Value
    ↓
Multi-Head Attention
    ↓
Feed-Forward Network
    ↓
Context-Aware Representation
```

## 3. Output Generation

```text
Final Representation
    ↓
Logits
    ↓
Next-token probabilities
    ↓
Temperature / Top-K / Top-P
    ↓
Decoding
    ↓
Next Token
    ↓
Repeat
```

## 4. Model Training

```text
Training Text
    ↓
Next-token prediction
    ↓
Cross-Entropy Loss
    ↓
Backpropagation
    ↓
Gradients
    ↓
Optimizer
    ↓
Update Weights
```

## 5. Model Lifecycle

```text
Pretraining
    ↓
Base Model
    ↓
SFT / Instruction Tuning
    ↓
Instruct Model
    ↓
Preference / Alignment Training
    ↓
Assistant Model
```

---

# Five Important Comparisons to Remember

| Concept 1 | Concept 2 | Easy Difference |
|---|---|---|
| Tokenization | Embedding | Split text vs convert token to numerical vector |
| Logits | Probabilities | Raw scores vs normalized likelihoods |
| Training | Inference | Learn/update weights vs use fixed trained weights |
| Base Model | Instruct Model | Text completion capability vs instruction-following behavior |
| Top-K | Top-P | Fixed token count vs cumulative probability threshold |

---

# How to Answer Naturally in an Interview

For almost every LLM question, use this structure:

```text
1. Start with the simple idea.
2. Explain why we need it.
3. Give a small example.
4. Connect it to the next step in the LLM flow.
```

For example, if asked about tokenization:

> Tokenization is the first step where we break the user's text into smaller units called tokens. We need this because the neural network cannot directly process normal text. Each token gets a token ID, and then that token ID is converted into an embedding. Those embeddings are what we actually send into the Transformer.

This sounds much more natural than only saying:

> Tokenization is the process of converting text into tokens.

The goal is to show the interviewer that you understand **where the concept fits in the complete LLM system**.

---

# 15-Minute Final Revision

Remember this one chain before the interview:

```text
LLM
 ↓
Transformer-based neural network
 ↓
Text becomes tokens
 ↓
Tokens become embeddings
 ↓
Position information is included
 ↓
Transformer layers
 ↓
Self-attention
 ↓
Q / K / V
 ↓
Multi-head attention
 ↓
Feed-forward processing
 ↓
Logits
 ↓
Next-token probabilities
 ↓
Temperature / Top-K / Top-P
 ↓
Next token selected
 ↓
Repeat autoregressively
```

And training:

```text
Text Data
 ↓
Next-token prediction
 ↓
Loss
 ↓
Backpropagation
 ↓
Optimizer updates parameters
 ↓
Base Model
 ↓
SFT
 ↓
Instruct Model
 ↓
Alignment / Preference Training
 ↓
Assistant Model
```


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
