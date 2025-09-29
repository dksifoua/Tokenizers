# Tokenizers - Subword Tokenization from scratch

Computers understand numbers, not text. Tokenization is the process of breaking down raw text into smaller units (
**tokens**) that can be converted into numerical IDs for a model to process. The choice of tokenizer has a huge impact
on a model's performance, vocabulary size, and ability to handle unseen words.

## Why tokenization is important?

- **Numerical Representation**: Tokenization translates text into a format that LLMs can process as numbers.
- **Understanding Context**: LLMs learn to understand the semantic relationships between tokens, which helps them
  generate meaningful responses.
- **Vocabulary and Efficiency**: Tokenization determines the model's vocabulary, and the method used significantly
  impacts the model's performance, cost, and ability to handle context.

## How does tokenization works?

1. **Text Decomposition**: The input text is divided into discrete units (tokens).
2. **Vocabulary Mapping**: Each token is assigned a unique numerical index from the LLM's predefined vocabulary.
3. **Numerical Input**: This sequence of numerical IDs serves as the input to the LLM's architecture.

## Common tokenization techniques

### 1. The naive approach - Word Tokenization

This is the most intuitive method. We split text into words based on spaces and punctuation.

**Pros:**

- **Intuitive**: The tokens are meaningful linguistic units (words).
- **Simple**: Easy to implement for languages like English that use spaces.

**Cons:**

- **Massive Vocabulary**: A model must have a complete vocabulary of every word it might see. This can easily grow to
  hundreds of thousands or even millions of tokens.
- **Out-of-Vocabulary (OOV) Problem**: What if the model encounters an unseen word like "tokenization"? It has to use a
  special `<UNK>` (unknown) token, losing all meaning.
- **Poor Handling of Morphology**: The model doesn't understand that `run`, `running`, and `ran` are related. Each is a
  separate, unrelated token.
- **Language Dependent**: Doesn't work well for languages without clear word boundaries (e.g., Chinese, Japanese).

### 2. The extreme opposite - Character Tokenization

To solve the OOV problem, we tokenize down to the smallest unit: individual characters.

**Pros:**

- **Tiny Vocabulary**: The vocabulary is just the set of possible characters (e.g., ~26 letters + punctuation for
  English).
- **No OOV Problem**: Any word can be constructed from the character set.

**Cons:**

- **Loss of Meaning**: Individual characters (mostly) carry no semantic meaning. The letter `c` alone tells you very
  little.
- **Long Sequences**: A sentence becomes a very long sequence of tokens (e.g., 50 characters instead of 10 words). This
  is computationally harder for models to process and learn from.
- **Linguistically Naive**: The model has to learn words and grammar from scratch, which is inefficient.

### 3. The best of the two worlds - Subword Tokenization

Subword tokenization strikes a balance between words and characters. The core idea is that frequent words should stay as
whole tokens, while rare words should be split into meaningful subwords.

This allows the model to:

- Keep the efficiency of word-level tokenization for common words.
- Handle unseen words by breaking them into known subwords, avoiding the `<UNK>` token.
- Understand morphology (e.g., `un+break+able`).

## Popular Subword Tokenization Algorithms

### A. Byte Pair Encoding (BPE) - Used by GPT, RoBERTa

1. Start with a character-level vocabulary.
2. Count the frequency of all adjacent pairs of tokens in your training corpus.
3. Merge the most frequent pair into a new token and add it to the vocabulary.
4. Repeat steps 2-3 for a set number of iterations or until the vocabulary reaches a desired size.

**Example:** Training on "low", "lower", "newest", "widest"
Most common pair is "e" + "s" -> merge to "es". (Now we have tokens for "es").
Next, maybe "es" + "t" -> merge to "est". (Now we have "est").
Later, "l" + "o" -> "lo", and then "lo" + "w" -> "low".
Now, to tokenize the unseen word "lowest", the tokenizer can split it into ["low", "est"], both of which are known,
meaningful tokens.

### B. WordPiece - Used by BERT

WordPiece is very similar to BPE but uses a slightly different merging strategy. Instead of just merging the most
frequent pair, it merges the pair that maximizes the likelihood of the training data once merged. In practice, it works
very similarly to BPE.

### C. Unigram Language Model - Used by T5, ALBERT

This approach starts from the opposite end:

1. It begins with a large vocabulary (e.g., all words and common subwords).
2. It trains a language model to compute the probability of a sentence given its tokens.
3. It iteratively removes tokens from the vocabulary that least affect the overall likelihood of the training data. This
   makes it very good at finding the most efficient tokenization.

# Summary & Comparison

| Feature              | Word Tokenization            | Character Tokenization      | Subword Tokenization               |
|----------------------|------------------------------|-----------------------------|------------------------------------|
| Vocabulary Size      | Very Large                   | Very Small                  | Medium, Controllable               |
| OOV Problem          | Severe (Uses `<UNK>` token)  | None                        | Rare (Uses subwords)               |
| Sequence Length      | Short                        | Very Long                   | Medium                             |
| Handling New Words   | Fails                        | Good                        | Excellent                          |
| Linguistic Meaning   | High (Words)                 | Low (Characters)            | Medium-High (Morphemes)            |
| Example              | ["Let", "s", "explore"]      | ["L", "e", "t", "s", ...]   | ["Let", "'", "s", "expl", "ore"]   |

# Conclusion

The shift from word and character tokenization to subword tokenization was a critical enabler for the modern era of
large language models (LLMs). It provides the perfect compromise, allowing models like GPT-4 and Llama to have a
manageable vocabulary while maintaining the flexibility to understand and generate any word, no matter how obscure, by
building it from familiar parts.
