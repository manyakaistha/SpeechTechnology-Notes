---
title: "Week 4 in plain language: hidden states, Viterbi and the speech recogniser"
week: 4
tags: [speech/eli12, speech/week-04]
---

# Week 4 in plain language: hidden states, Viterbi and the speech recogniser

**Explain-like-I'm-12 version** of the [full Week 4 notes](../week-4/week-4-notes.md). Exam wording is in the [exam-focused guide](../week-4/week-4-exam-focused-notes.md) and the [Week 4 past papers](../week-4/week-4-exam-questions.md). Read [Week 3 in plain language](week-3-eli12.md) first if Markov chains or GMMs feel new.

> [!NOTE]
> **The big idea of Week 4.** In speech we never *see* the sound a person is making inside their mouth. We only see numbers measured from the audio. A **Hidden Markov Model (HMM)** is a way to guess the hidden story (which sound state we are in) from visible clues (the measured features).

## 1. Hidden versus visible

- **Weather chain (Week 3):** you see Sunny, Cloudy, Rainy directly.
- **Weather HMM:** you are inside a windowless room. You only see whether the grass is Dry, Damp or Soggy. From the grass you must guess the weather outside.
- **Speech HMM:** the *hidden* thing is the sound state (like "start of /s/"). The *visible* thing is a vector of 13 MFCC numbers in each 10 ms frame.

An HMM is described by three tables:

| Name | Question it answers |
| :-- | :-- |
| $\pi$ (initial) | Which state do we start in? |
| $A$ (transitions) | From this state, which state comes next? (rows add to 1) |
| $B$ (emissions) | In this state, how likely is this clue? |

Two simplifying promises: (1) the next state depends only on the current state, and (2) each clue depends only on the state that produced it.

Keep the two arrows apart: **transition** goes state to state; **emission** goes state to clue. The exam question that asks how a chain differs from an HMM ([2023 Quiz 2 Q7](../week-4/week-4-exam-questions.md#ee4001_2023T3_Q2_NA--q7)) wants exactly this: in an HMM the states are not directly observed.

## 2. Scoring one story

To score a guessed story (path) together with the clues, multiply as you go: initial chance, then for each step one **transition** and one **emission**.

| What you are asked | What you do |
| :-- | :-- |
| Score of one given path and the clues | Multiply all the transition and emission factors |
| Chance of the clues if the path is fixed | Multiply only the emissions |
| Best path | Use **Viterbi** (take the max) |
| Total chance of the clues over every path | Use **Forward** (take the sum) |

## 3. Viterbi: finding the single best story

With 3 states and 4 clues there are already $3^4 = 81$ possible stories. Checking all of them gets out of hand quickly. Viterbi is a shortcut.

**The trick.** For each state at each time, keep only the *best* score of any story that ends there, and write down where it came from (a **back-pointer**). Two stories that reach the same state have the same future, so the weaker one can be thrown away.

**Steps:**

1. Fill the first column with start score times clue score.
2. For each next column and each state, look at every previous state, multiply by the transition, keep the **largest**, multiply by the clue, remember the winner.
3. At the end, take the best final state and follow the back-pointers backwards.

**A real exam result** ([2022 Quiz 2 Q9](../week-4/week-4-exam-questions.md#ee4001_2022T3_Q2_NA--q9)): known Cloudy start, then clues Dry, Damp. Best path over Days 1-2: Sunny, Cloudy, with score $0.25 \times 0.60 \times 0.375 \times 0.25 = 0.0140625$.

**Ties happen.** In [2026 Quiz 1 Q9](../week-4/week-4-exam-questions.md#ee4001_2026T1_Q1_NA--q9) two different paths (S,S,R and S,R,R) both score $0.00864$. A full answer says both are optimal.

The same machinery tags words: for "Time flies fast" it finds Noun, Verb, Verb ([2026 Quiz 1 Q11](../week-4/week-4-exam-questions.md#ee4001_2026T1_Q1_NA--q11)).

> [!TIP]
> **Mistakes to avoid:** forgetting the emissions; adding instead of taking the max; picking the best state at every time separately instead of tracing back; ignoring ties.

## 4. Forward: the total chance of the clues

Forward uses the same table but **adds** the contributions instead of taking the max:

$$\alpha_t(j) = b_j(o_t) \sum_i \alpha_{t-1}(i)\,a_{ij}.$$

Adding the last column gives $P(\text{clues})$ over every possible story. In the Cloudy-start example the total is 0.05, while the single best path scores 0.0140625. So the best path has a share of $0.0140625 / 0.05 = 0.28125$ of all the evidence.

A joint score of 0.03 out of a total of 0.12 means the path's *posterior* is $0.03 / 0.12 = 0.25$.

**Forward-Backward** combines "what came before" (forward) with "what comes after" (backward) to tell, for each frame, *how much* it belongs to each state, like the soft fractions in EM. Training with this soft method is called **Baum-Welch**.

> [!NOTE]
> **Max versus sum.** Viterbi asks *"which single story is best?"*. Forward asks *"how likely are the clues overall?"*. With long sequences the numbers get tiny, so we work in logs: Viterbi uses plain addition; Forward must use log-sum-exp, not max.

## 5. The speech recogniser built from HMM + GMM

- The **HMM** says which sound states can follow which.
- In each state, a **GMM** (several bell curves added together, from Week 3) says how likely a given feature vector is. That is the emission.

A **density** from a GMM can be larger than 1. That is fine; it is probability per unit of feature space, not a point probability.

Two ways to feed features in:

| | Discrete HMM | Continuous HMM |
| :-- | :-- | :-- |
| What the state sees | A label from a codebook | The real feature vector |
| Emission | A table of probabilities | A density (like a GMM) |

[2026 Quiz 2 Q1](../week-4/week-4-exam-questions.md#ee4001_2026T1_Q2_NA--q1) asks the advantage of the continuous version: it models real-valued features directly, so no quantisation step throws detail away.

### Topology and duration

A common **left-to-right** HMM moves through a sound in order: beginning, middle, end. Each state can also **loop** back to itself so it can last several frames. With self-loop probability $a$, the average stay is $1/(1-a)$ frames. For $a = 0.8$ that is 5 frames, or about 50 ms at a 10 ms step. So self-loops model **variable duration** ([2026 Quiz 2 Q2](../week-4/week-4-exam-questions.md#ee4001_2026T1_Q2_NA--q2)).

### Tri-state versus triphone: two different ideas

| Word | Changes with | Example |
| :-- | :-- | :-- |
| **Three-state** phone | Position in time inside the sound | beginning / middle / end of /ae/ |
| **Triphone** | The neighbours on each side | /ae/ with /k/ before and /t/ after, written k-ae+t |

For "cat" (/k/, /ae/, /t/) with silence at each end: 3 phones $\times$ 3 states $= 9$ emitting positions; and the triphone units are sil-k+ae, k-ae+t and ae-t+sil ([2025 Quiz 2 Q8](../week-4/week-4-exam-questions.md#ee4001_2025T2_Q2_NA--q8)).

With 50 phones, the possible left/right neighbours for one fixed centre phone is $50 \times 50 = 2500$. Counting every possible centre as well gives $50^3 = 125{,}000$. Real systems do not train all of them separately; **state tying** lets similar states share one set of parameters, which helps when data is scarce.

**Is Viterbi only for testing?** No. It is also used during training to line frames up with states (a "hard" alignment), after which the GMMs are re-fitted. Forward-Backward is the "soft" version.

## Check yourself

1. Three states, four time steps: how many unconstrained paths? *(81.)*
2. Self-loop probability 0.75: expected stay in frames? *(4 frames, about 40 ms.)*
3. Best-path score 0.03, total clue probability 0.12: path posterior? *(0.25.)*
4. What is hidden in a speech HMM? *(The acoustic state sequence; features are the visible clues.)*
5. Two paths reach the same state with scores 0.2 and 0.1. Which does Viterbi keep? *(0.2; both share the same future.)*

## What to carry into Week 5

We can now score and decode *sound states*. But people say **words**, and some word sequences are far more likely than others. Week 5 adds the missing pieces: a pronunciation dictionary and a language model. Continue with [Week 5 in plain language](week-5-eli12.md).
