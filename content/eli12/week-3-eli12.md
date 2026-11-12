---
title: "Week 3 in plain language: bell curves, mixtures, clusters and weather chains"
week: 3
tags: [speech/eli12, speech/week-03]
---

# Week 3 in plain language: bell curves, mixtures, clusters and weather chains

**Explain-like-I'm-12 version** of the [full Week 3 notes](../week-3/week-3-notes.md). Exam wording is in the [exam-focused guide](../week-3/week-3-exam-focused-notes.md) and the [Week 3 past papers](../week-3/week-3-exam-questions.md). If bell curves are new, read [Week 2 in plain language](week-2-eli12.md) first.

> [!NOTE]
> **The big idea of Week 3.** Real sound features form *clouds* of points, often with more than one bump. Week 3 teaches four tools: (1) use a bell curve to **decide** between two groups, (2) use **several** bell curves added together (a GMM), (3) **group** points by distance (K-means), and (4) predict a **sequence** one step at a time (Markov chains).

## 1. Probability words you will keep seeing

- **Prior** $P(C)$: how common a group is *before* you look at the measurement. (Most people you meet are not astronauts.)
- **Likelihood** $p(x \mid C)$: how well the measurement fits that group's bell curve.
- **Posterior** $P(C \mid x)$: how likely the group is *after* looking at the measurement. This is the one you decide with.

To get a posterior: multiply prior by likelihood for every group, then divide by the total so they add up to 1.

**Tiny example.** Two groups. Likelihoods are 0.2 and 0.5. Priors are 0.8 and 0.2.

- Group 1: $0.8 \times 0.2 = 0.16$
- Group 2: $0.2 \times 0.5 = 0.10$
- Total $= 0.26$, so posteriors are $0.16/0.26 \approx 0.615$ and $0.10/0.26 \approx 0.385$.

Group 1 wins even though its likelihood is smaller, because it is much more common. Forgetting the prior is the classic mistake.

## 2. Two bell curves, one decision line

Give each group its own bell curve. For a new measurement, pick the group whose (prior-weighted) curve is taller there. If both groups have equal spread and equal priors, the curves cross exactly halfway between their centres, and that crossing is the **decision boundary**.

The **error** is the area of a curve that falls on the wrong side of the line. Overall error must be weighted by how common each group is (the priors). The notes' example gives each tail $\approx 0.1587$ and an overall error of $0.1587$, **not** $0.3173$. Adding the two tails without the weights counts people twice.

## 3. More than one feature: covariance

With two features (for example two MFCC numbers) each point is a dot on a plane, and a bell curve becomes a **hill with oval outlines**.

- **Variance** tells how spread out each feature is.
- **Covariance** tells whether the two features rise together. Positive covariance tilts the oval upward; negative tilts it downward.

All this is stored in a small square table, the **covariance matrix**. Its diagonal holds the variances; the off-diagonal entries hold the covariances. Example: the four points $(0,0), (2,0), (0,2), (2,2)$ have mean $(1,1)$, each variance $= 1$ and covariance $= 0$, so the matrix is the identity.

## 4. Gaussian Mixture Model (GMM): several bumps added together

One bell curve has one bump. But the sounds one phoneme makes can form **several** bumps (different speakers, different contexts). So we add several weighted bell curves:

$$p(x) = \sum_k w_k\, \mathcal N(x;\mu_k,\sigma_k^2), \qquad \text{weights add to 1.}$$

Think of three friends each shouting a number; the mixture is the combined crowd noise, not just the loudest voice.

### The exam calculation, step by step

At $x = 2$, two bell curves centred at 1 and 3, spread 1, with weights 0.6 and 0.4 ([2026 Quiz 1 Q10](../week-3/week-3-exam-questions.md#ee4001_2026T1_Q1_NA--q10)):

1. Both curves are one unit away from $x=2$, so both have height $\approx 0.2420$.
2. Multiply by the weights: $0.6 \times 0.2420 = 0.1452$ and $0.4 \times 0.2420 = 0.0968$.
3. Add them: total density $\approx 0.2420$.
4. The chance that component 1 made this point: $0.1452 / 0.2420 = 0.6$. That is the **responsibility**.

> [!TIP]
> **Four different numbers, four different jobs:** component height → weighted piece → total density → responsibility. Only the last one is a probability of membership.

For many points we multiply the totals of every point (the **likelihood**) or, more easily, add their logs (the **log-likelihood**). The log of a *product* becomes a sum. The log of a *mixture sum* does **not** split apart.

## 5. EM: fitting a mixture when nobody tells you the labels

If every point came with a label, fitting would be easy: average each group. But the labels are unknown. **EM** (Expectation-Maximization) repeats two steps:

- **E-step:** for each point, guess *how much* it belongs to each component (the responsibilities, like 0.8 and 0.2).
- **M-step:** re-fit each component using those fractions as weights (a point with responsibility 0.8 counts as 0.8 of a vote).

A mini case: data $[0, 2]$, responsibilities $[0.8, 0.2]$ and $[0.2, 0.8]$. Each component has an effective count of 1, so weights become 0.5 each. New means: $0.4$ and $1.6$. New variance for each: $0.64$.

## 6. K-means and LBG: grouping with a codebook

**K-means** is the "hard" cousin of EM: each point goes to exactly one nearest centre.

Algorithm: pick centres, give every point to its nearest centre, move each centre to the average of its points, repeat.

**Exam example** ([2025 Quiz 1 Q6](../week-3/week-3-exam-questions.md#ee4001_2025T2_Q1_NA--q6)): data $[2,4,5,8,9]$, starting centres 4 and 8.

- The halfway line is 6. Points 2, 4, 5 go to centre 4; points 8, 9 go to centre 8.
- New centres: $(2+4+5)/3 \approx 3.667$ and $(8+9)/2 = 8.5$.

**LBG** starts with one centre and repeatedly **splits** each centre in two, using a small nudge $\epsilon$, then refines ([2025 Quiz 1 Q7](../week-3/week-3-exam-questions.md#ee4001_2025T2_Q1_NA--q7)): mean 5.5 splits into 6.05 and 4.95 for $\epsilon=0.1$.

**Codebook size:** each frame is replaced by a label (an index). With $K=50$ labels you need 6 bits per label; with $K=2000$ you need 11 ([2026 End Term Q18](../week-3/week-3-exam-questions.md#ee4001_2026T1_ET_FN--q18) asks a related nearest-vector question).

| | K-means | EM / GMM |
| :-- | :-- | :-- |
| Membership | Hard (one group) | Soft (fractions) |
| Fits | Just centres | Centres, spreads and weights |
| Reaches best answer? | Not guaranteed | Not guaranteed |

Both can get stuck in a "local best". Starting from several different places and comparing results helps.

## 7. Markov chains: tomorrow depends only on today

A **Markov chain** predicts the next step using only the current state. Weather example with states Sunny, Cloudy, Rainy: a table lists, for each *today*, the chance of each *tomorrow*. Each row of the table adds to 1.

To estimate a transition from data, count departures: in S, C, S, C, C the state C leaves twice (to S once and to C once), so $P(S \mid C) = 1/2$. The final C has no next step, so it adds no count.

### Greedy versus best whole path

*Greedy* picks the best next step every time. Starting Sunny, greedy stays Sunny three days in a row with score $0.5^3 = 0.125$.

The better method, **Viterbi**, tracks, for each state, the best score that *ends* there, and remembers where it came from. Tracing back from the best end gives the best whole path: Sunny, Cloudy, Rainy, Rainy, with score $0.375 \times 0.625 \times 0.675 = 0.158203125$. That beats greedy's 0.125, even though the first step (to Cloudy) looked weaker. See [2023 Quiz 1 Q6](../week-3/week-3-exam-questions.md#ee4001_2023T3_Q1_NA--q6).

> [!TIP]
> **Why keeping one best path per state is safe:** two paths that end in the same state face exactly the same future, so the weaker one can never catch up.

## Check yourself

1. Equal likelihoods, priors 0.7 and 0.3: what is the first posterior? *(0.7; equal likelihoods cancel.)*
2. In EM, a point has responsibilities $[0.2, 0.8]$. Does it belong only to component 2? *(No. It counts fractionally to both.)*
3. K-means, data $[1,2,3,8,9,10]$ with centres 2 and 9. Where do 3 and 8 go? *(3 to centre 2, 8 to centre 9.)*
4. Does picking the best state in every Viterbi column give the best path? *(No. Follow the stored back-pointers.)*
5. How many bits label one of 50 codewords? *(6.)*

## What to carry into Week 4

In this week the weather was *visible*. In Week 4 the states become **hidden**: we only see clues (like the dampness of the grass) and must guess the states behind them. That is the Hidden Markov Model. Continue with [Week 4 in plain language](week-4-eli12.md).
