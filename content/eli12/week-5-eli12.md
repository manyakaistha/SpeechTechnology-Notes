---
title: "Week 5 in plain language: from sounds to sentences"
week: 5
tags: [speech/eli12, speech/week-05]
---

# Week 5 in plain language: from sounds to sentences

**Explain-like-I'm-12 version** of the [full Week 5 notes](../week-5/week-5-notes.md). There is no past-paper bank for Week 5 yet, so the practice here uses the notes' own worked examples. Read [Week 4 in plain language](week-4-eli12.md) first.

> [!NOTE]
> **The big idea of Week 5.** A speech recogniser must answer "what words were said?" Three experts vote together: the **acoustic model** (does this sound match?), the **pronunciation dictionary** (how is each word pronounced?) and the **language model** (is this word sequence likely in real life?).

## 1. The three experts

| Expert | Job | Everyday comparison |
| :-- | :-- | :-- |
| **Acoustic model** (HMM + GMM) | Scores how well the audio matches a sequence of sounds | A friend who judges whether it *sounded* like "ice cream" |
| **Lexicon** (pronunciation dictionary) | Spells each word as sounds | A dictionary's pronunciation key |
| **Language model** | Scores how likely a word sequence is | Someone who knows people say "ice cream", not "I scream" in a shop |

The recogniser picks the word sequence $W$ that makes the **acoustic score times the language score** biggest:

$$\hat W = \arg\max_W \; P(O \mid W)\, P(W).$$

## 2. Revisiting the acoustic model

An HMM has start probabilities, transitions and emissions. In speech, each emission is usually a small GMM over the 13-number feature vector. Keep four counts apart:

| Letter | Counts | Example |
| :-- | :-- | :-- |
| $N$ | HMM states | 40 |
| $D$ | Numbers in one feature vector | 13 |
| $M$ | Labels in a codebook (discrete HMM) | 256 |
| $K$ | Bell curves inside one GMM | 4 |

If features are first rounded into codebook labels, the HMM is **discrete** and each state keeps a table of $M$ probabilities that add to 1. If features are used as they are, it is **continuous** and each state keeps a density that integrates to 1.

A density value is **not** the chance that a sound is a particular phoneme. To get that you need the priors and must compare with the other states, as in this example: densities 0.8 and 1.2, priors 0.75 and 0.25. Joint scores are $0.60$ and $0.30$, so the posteriors are $2/3$ and $1/3$. State 1 wins even though state 2 has the larger density.

### Shapes of HMM (topologies)

- **Ergodic / fully connected:** every state may go to every other state.
- **Left-to-right:** states go forward in order (with self-loops, maybe skips). It matches how a sound unfolds in time.

A tiny left-to-right table: from state 1 you can stay (0.6) or advance (0.4); from state 2 stay (0.6) or advance (0.4); state 3 stays. Zeros are the forbidden backward moves.

Counting reminders: 44 phones $\times$ 3 states $= 132$ state positions. For triphones with 50 phones, the *possible* labels number $50^3 = 125{,}000$ and with three states each, $375{,}000$ positions. Real systems **tie** states, so they train far fewer distinct models.

## 3. The pronunciation dictionary (lexicon)

The lexicon maps each **word** to one or more **sound sequences**. Some words have several pronunciations ("read" can sound two ways), and different words can sound alike ("to" and "two").

Important points:

- The sounds in the dictionary must match the sounds the acoustic model was trained on.
- The decoder explores **all** pronunciations while scoring; it does not first fix one string of sounds and look the word up afterwards.
- If a word is missing from the dictionary, the recogniser cannot output it. A tool that guesses pronunciations from spelling (**G2P**) helps, but the word still has to be added to the dictionary, the search vocabulary **and** the language model.

**Example.** A word has two pronunciations with priors 0.7 and 0.3 and acoustic scores 0.02 and 0.10. Contributions: $0.7 \times 0.02 = 0.014$ and $0.3 \times 0.10 = 0.030$. The less common pronunciation fits the audio better and contributes more. Adding both gives 0.044; a Viterbi-style shortcut would keep 0.030.

## 4. The language model: guessing the next word

Some word orders are just more natural. The **chain rule** is exact:

$$P(w_1 \ldots w_L) = \prod_i P(w_i \mid w_1 \ldots w_{i-1}).$$

But long histories are rarely seen in your data, so an **N-gram** keeps only the last $N-1$ words:

- **Unigram:** no history.
- **Bigram:** the previous word.
- **Trigram:** the previous two words.

### A toy corpus you can count by hand

Three sentences: *i eat apples*, *i eat bananas*, *you eat apples*, each followed by an end marker `</s>`.

- "eat" is followed by *apples* twice and *bananas* once, so $P(\text{apples} \mid \text{eat}) = 2/3$ and $P(\text{bananas} \mid \text{eat}) = 1/3$. The bottom number, 3, is how many times "eat" is followed by anything.
- Bigram chance of *i eat apples* then end: $P(i \mid \text{start}) = 2/3$, $P(\text{eat} \mid i) = 1$, $P(\text{apples} \mid \text{eat}) = 2/3$, $P(\text{end} \mid \text{apples}) = 1$, so the total is $\tfrac23 \cdot 1 \cdot \tfrac23 \cdot 1 = 4/9$.
- Trigram (look back two words) gives $\tfrac23 \cdot 1 \cdot \tfrac12 \cdot 1 = 1/3$.
- The bigram model gives the *unseen* sentence "you eat bananas" a chance of $1/9$ because each piece was seen elsewhere. The trigram model gives it **zero**, because "you eat" was never followed by "bananas".

Zero chances are dangerous: one zero wipes out the whole sentence. **Smoothing** (also backoff or interpolation) takes a little probability from seen events and shares it with unseen ones.

> [!TIP]
> **Boundary markers.** The start marker is only ever *context* (we never predict it). The end marker *is* predicted. Always say which convention your denominator uses.

A language model is **not** a grammar checker. It only says what was common in its training text.

## 5. Putting it together: the audio and the words vote

Candidate $W_a$ has acoustic score 0.02 and language prior 0.1. Candidate $W_b$ has acoustic score 0.01 and prior 0.4. Multiply: $0.002$ versus $0.004$. $W_b$ wins even though the audio fit is worse, because it is a far more likely sentence. This is exactly how a recogniser chooses "recognise speech" over "wreck a nice beach".

Real systems add a **language-model weight** and a **word-insertion bonus** to balance the experts, but the idea stays the same.

## Check yourself

1. Which count controls the number of rows of the transition matrix: $N$, $D$, $M$ or $K$? *($N$, the number of states.)*
2. Does a bigger density value mean a more probable phoneme? *(No. You need priors and a comparison.)*
3. In the toy corpus, what is $P(\text{bananas} \mid \text{eat})$? *(1/3.)*
4. Why can a trigram sentence score be zero while the bigram one is not? *(A needed three-word combination was never seen.)*
5. A new surname has a G2P pronunciation but is not in the vocabulary. Is the out-of-vocabulary problem fixed? *(No. It must be in the dictionary, search vocabulary and language model.)*

## You have finished the plain-language path

For exam-style wording and marks, go back to the [exam study route](../exam-study-guide.md), then work through the [past-paper topics](../past-paper-topics.md).
