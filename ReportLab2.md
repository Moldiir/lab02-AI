# Lab 02 — Inside the Model

## 1. Introduction

In this lab I worked with GPT-2 small (124M parameters and a vocabulary of 50,257 tokens) to check some ideas from Lecture 3 with a real language model. I looked at tokenization, next-token probabilities, temperature, top-p, and attention weights. I also compared English, Russian, and Kazakh text to see how the same model handles different languages.

The experiments showed that the way a model processes a language depends not only on the language itself, but also on the tokenizer and training data used to build the model.

## 2. Part 0 — A Token Is Not a Word

I tested how GPT-2 splits different words into tokens. Before running the code, I predicted that the Kazakh word `бөлімшеңізде` would take 9 tokens.

The actual result was **19 tokens** for `бөлімшеңізде`. The other results were:

* `restore` → 2 tokens
* `bank` → 1 token
* `банк` → 5 tokens
* `бөлімшеңізде` → 19 tokens

The results show that English words can be represented with fewer tokens, while Cyrillic words are often split into smaller byte-level pieces. For example, `банк` was split into 5 tokens: `['Ð', '±', 'Ð°', 'Ð½', 'Ðº']`.

This happens because GPT-2's tokenizer learns merges from its training data. Character or byte combinations that occur frequently can become larger tokens. GPT-2 has much better coverage of English than Cyrillic text, so Cyrillic words often require more tokens.

This experiment showed me that a token is not necessarily a whole word or a syllable. It is simply a piece from the model's vocabulary. The same word can require a different number of tokens depending on the tokenizer.

## 3. Part 1 — The Output Is 50,257 Numbers

For the prompt:

`The capital of Kazakhstan is Astana. The capital of France is`

I predicted that the most likely next token would be ` Paris` with about 20% probability, and that the top 10 tokens would contain more than 90% of the total probability.

The actual result was different. `Ast` had the highest probability at **22.87%**, while ` Paris` was very close at **22.08%**. The top 10 tokens together contained only about **64%** of the probability. Therefore, my prediction was only partly correct.

I also checked ` Paris` and `Paris`. They are different tokens because GPT-2 can include the leading space as part of a token. This means that even a small difference in formatting can lead to different probabilities.

The experiment showed that the model does not simply output one correct answer. It produces a probability distribution over all 50,257 possible tokens. The highest-probability token is only the most likely next step and is not guaranteed to be correct.

## 4. Temperature

I predicted that the #1 token would stay the same for temperatures 0.25, 1, 2, and 5, while its probability would decrease as the temperature increased.

The result confirmed this. Temperature divides all logits by the same positive number before applying softmax. Because their order stays the same, the #1 token does not change. However, the probability distribution becomes more concentrated at low temperatures and more spread out at high temperatures.

This also showed that a temperature close to zero does not make the model more correct. It only makes the model more deterministic. If the highest-probability token is wrong, a low temperature can make the model choose that wrong token more consistently.

## 5. Top-p

Temperature and top-p affect the probability distribution in different ways.

**Temperature** reshapes the distribution by making it more concentrated or more spread out, but it does not remove tokens.

**Top-p** keeps the smallest group of the most likely tokens whose cumulative probability reaches the chosen value of `p`, and sets the probabilities of the remaining tokens to zero.

Therefore, reshaping and deleting are not the same operation. Temperature changes the probabilities of the tokens, while top-p changes which tokens are allowed to be selected.

## 6. Part 2 — Attention Is a Table of Weights

GPT-2 small has **12 layers × 12 attention heads = 144 attention heads**. Each attention matrix contains weights showing how much one token attends to other tokens. The upper triangle is empty because the causal mask prevents a token from looking at future tokens. The rows sum to approximately 1 because the attention weights are produced using softmax.

### Previous-token head

The strongest previous-token pattern was at **layer 4, head 11**.

The heatmap shows a diagonal band one position below the main diagonal. This means that a token at position *i* attends mostly to the previous token at position *i−1*. The main diagonal would represent a token attending to itself, so the shift below the diagonal shows the previous-token pattern.

Not every head in layer 4 behaves this way. Head 11 was a clear outlier, with a previous-token score close to 1.0, while the other heads had much lower values, around 0.1–0.4. This shows that different attention heads can specialize in different patterns.

### Attention to token 0

The highest average attention to token 0 appeared mainly in layers 5–10, with average values around **0.70–0.80**. One head had more than **95%** attention on token 0.

This does not mean that the model is simply "focusing on the first word." Attention weights show how much information from another token's value vector is mixed into the current representation. They do not directly explain why the model made a particular prediction.

Therefore, I would describe the result as some attention heads putting a large amount of their attention on token 0, rather than saying that the model is thinking about the first word. The heatmap shows part of the model's mechanism, but it is not by itself an explanation of the model's reasoning.

## 7. Part 3 — The Same Model in Kazakh

I compared the same sentence in English, Russian, and Kazakh. Before running the experiment, I predicted that Kazakh would need the most tokens per character because GPT-2 was trained mainly on English text.

The results were:

* English: **0.22 tokens/character**
* Russian: **1.11 tokens/character**
* Kazakh: **1.10 tokens/character**

Both Cyrillic languages needed about five times more tokens per character than English. However, Russian and Kazakh were almost identical, so my prediction that Kazakh would be clearly worse than Russian was not confirmed.

Generation quality also dropped for the Cyrillic prompts. With greedy decoding, the Russian prompt produced:

`Столица Казахстана — Столица Казахста`

The model repeated part of the prompt instead of producing a useful continuation.

For the Kazakh prompt, the result was:

`Қазақстанның астанасы — проссии проссии про`

The output quickly became repetitive and incoherent.

This does not mean that the Kazakh language itself is the problem. The main issue is that GPT-2's tokenizer and training data cover English much better than Cyrillic languages. Many Cyrillic combinations are poorly represented, so the same sentence requires more tokens and the model has less useful training information for these languages.

A multilingual tokenizer and a model trained on substantial Russian and Kazakh data would likely represent these languages more efficiently and produce more coherent text.

## 8. Conclusion

This lab helped me understand what happens inside a language model beyond simply seeing its final answer.

First, a token is not necessarily a complete word. GPT-2 can split a word into several byte-level pieces depending on its vocabulary and tokenizer. Second, the model does not produce one direct answer. It produces probabilities for all 50,257 possible next tokens, and the highest-probability token is not always the most sensible one.

I also saw that temperature changes how concentrated the probability distribution is without changing the top token, while top-p removes low-probability tokens from consideration. In the attention experiment, I found a clear previous-token head at **layer 4, head 11**, showing that individual attention heads can have different functions.

Finally, the English, Russian, and Kazakh comparison showed a large difference in token efficiency. Russian and Kazakh needed about five times more tokens per character than English, and their generated text was much less coherent. This showed me that model performance depends strongly on tokenizer design and training data coverage, rather than on the language itself.
