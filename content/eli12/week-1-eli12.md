---
title: "Week 1 in plain language: how a voice becomes numbers"
week: 1
tags: [speech/eli12, speech/week-01]
---

# Week 1 in plain language: how a voice becomes numbers

**Explain-like-I'm-12 version.** This page teaches the same ideas as the [full Week 1 notes](../week-1/week-1-notes.md) with pictures-in-words, small numbers and no jargon left unexplained. When you are ready for the exam wording, move to the [exam-focused guide](../week-1/week-1-exam-focused-notes.md) and the [Week 1 past papers](../week-1/week-1-exam-questions.md).

> [!NOTE]
> **The big idea of Week 1.** A computer cannot listen. It can only read lists of numbers. Week 1 is about turning a sound into a list of numbers without losing the important part, and then describing what simple "machines" (systems) do to those numbers.

## 1. What is sound, really?

When you speak, your voice makes the air push and pull very quickly. That pushing and pulling is **sound**. A microphone turns it into a wiggly electrical signal. We call the wiggly line $x(t)$: it says "how hard is the air pushing at time $t$ seconds?"

A computer cannot store a smooth wiggly line, because it has infinitely many points. So it does the next best thing: it **looks at the line every tiny moment and writes down one number**. That is called **sampling**.

## 2. Sampling: taking photos of a moving thing

Think of a flipbook. A cartoon of a ball bouncing is just many still pictures shown quickly. If you take enough pictures per second, you can rebuild the movement. Sampling a sound is the same: we "photograph" the signal many times per second.

- The number of photos per second is the **sampling rate** $f_s$. At $f_s = 16{,}000$ we take 16,000 numbers every second.
- One number we wrote down is a **sample**, written $x[n]$. Square brackets just mean "we are now talking about the list of numbers, number $n$ in the list".

### How many samples fit in a short time?

Multiply the rate by the time (in seconds!):

$$N = f_s \times T$$

At 16,000 samples per second, a 10 millisecond piece is $16000 \times 0.010 = 160$ samples. This exact question appears in the exam: [2026 End Term Q3](../week-1/week-1-exam-questions.md#ee4001_2026T1_ET_FN--q3).

> [!TIP]
> **Milliseconds trap.** 10 ms is 0.010 seconds, not 10 seconds. Always divide milliseconds by 1000 first.

## 3. Aliasing: the wagon-wheel trick

In old films, a fast-spinning wagon wheel sometimes looks like it is spinning slowly or even backwards. The camera takes pictures too slowly to follow the wheel, so the pictures fool your eyes. That fooling is **aliasing**.

Sound does the same thing. If you sample too slowly, a fast wiggle looks exactly like a slow wiggle in your list of numbers. For example, a 5 kHz tone sampled at 8 kHz looks like a 3 kHz tone ($8 - 5 = 3$). Once the numbers are written down, you can never tell which tone was real.

### The rule that prevents it

Take photos **at least twice as fast as the fastest wiggle you care about**:

$$f_s \ge 2 f_{\max}$$

- If your signal is cut off at 10 kHz, you need at least 20 kHz. That is [2026 Quiz 1 Q6](../week-1/week-1-exam-questions.md#ee4001_2026T1_Q1_NA--q6): answer **20 kHz**, and always show the formula.
- The multiple-choice version asks the same thing: the sampling frequency must be *greater than or equal to twice the maximum frequency* (option B). See [Quiz 1 Q5](../week-1/week-1-exam-questions.md#ee4001_2026T1_Q1_NA--q5).

Two names people mix up:

| Name | What it is | Example |
| :-- | :-- | :-- |
| **Nyquist rate** | The slowest rate that is safe: $2 f_{\max}$ | 20 kHz for a 10 kHz signal |
| **Nyquist frequency** | The highest tone a given rate can hold: $f_s / 2$ | 8 kHz when $f_s = 16$ kHz |

An **anti-alias filter** removes the too-fast wiggles *before* sampling. Afterwards it is too late, just as you cannot fix a wagon-wheel film by editing it.

## 4. Quantization: rounding the height

Sampling decides *when* to write a number. **Quantization** decides *how precisely* to write it. A computer only has a fixed number of "steps" for the height. With 8 bits there are $2^8 = 256$ steps. Between two steps the number is rounded. More bits means finer steps and less rounding error.

- Sampling makes **time** into steps.
- Quantization makes **height** into steps.

They are different jobs. Do not mix them up.

## 5. Systems: machines that change signals

A **system** is a machine: numbers go in, numbers come out. Three questions describe it.

### Is it linear? (Does "double in, double out" hold, and do sums add up?)

If you feed in two signals together, you should get the same as feeding them in separately and adding the answers. A quick way to *disprove* linearity: feed in all zeros. A linear machine must give zeros back.

- $y[n] = 2x[n] + 3$: zero in gives 3 out. **Not linear.**
- $y[n] = x[n]^2$: double the input and the output becomes four times bigger. **Not linear.**
- $y[n] = n\,x[n]$ and $y[n] = x[2n]$: these obey superposition. **Linear.**

### Is it time-invariant? (Does waiting change the machine?)

Delay the input by a few steps. If the output is just the old output delayed by the same amount, the machine does not care what time it is: **time-invariant**. If the machine behaves differently at different times, it is **time-varying**.

- $y[n] = n\,x[n]$ multiplies by the clock number $n$, so it behaves differently later. **Time-varying.**
- $y[n] = x[2n]$ squeezes time, so a delay of $n_0$ turns into a delay of $2n_0$. **Time-varying.**
- $2x[n]+3$ and $x[n]^2$ do the same thing at every moment. **Time-invariant.**

### Is it causal? (Does it peek into the future?)

A **causal** machine only uses the present and the past. If the answer needs $x[n+1]$, it is peeking at the future, so it is not causal.

### The exam answers, all in one table

| Rule | Linear? | Time-invariant? | Appears in |
| :-- | :-- | :-- | :-- |
| $n\,x[n]$ | Yes | No | [2022 Q1 Q9](../week-1/week-1-exam-questions.md#ee4001_2022T3_Q1_AN--q9) |
| $e^{x[n]}$ | No (zero in gives 1) | Yes | [2022 Q1 Q9](../week-1/week-1-exam-questions.md#ee4001_2022T3_Q1_AN--q9) |
| $2x[n]+3$ | No | Yes | [2026 Q1 Q8](../week-1/week-1-exam-questions.md#ee4001_2026T1_Q1_NA--q8) |
| $x[n]^2$ | No | Yes | [2026 Q1 Q8](../week-1/week-1-exam-questions.md#ee4001_2026T1_Q1_NA--q8) |
| $x[2n]$ | Yes | No | [2026 Q1 Q8](../week-1/week-1-exam-questions.md#ee4001_2026T1_Q1_NA--q8) |

## 6. Frequencies: building a sound from simple waves

Any sound can be built by adding simple, smooth waves of different speeds (frequencies), like mixing paint from a few basic colours. The recipe says how much of each wave to add. This recipe is the **Fourier** idea. On a computer we use the **DFT** (and its fast version, the **FFT**) to find the recipe.

A tiny example: for the list $[1,1,1,1]$, which never changes, the recipe is "4 units of the *constant* wave (DC) and none of anything else". A steady signal has no wiggle in it.

The word the exam wants for "building blocks": any point in 3-D space is a **linear** combination of its basis directions.

## 7. Convolution: smearing a signal through a filter

If a system is both linear and time-invariant (**LTI**), one small list, the **impulse response** $h$, tells you everything. The output is a "smear" of the input through $h$, called **convolution**. A tiny case: input $[1,2]$ through $h = [1,1]$ gives $[1,3,2]$.

Think of a pebble dropped in water: each input sample makes a ripple shaped like $h$, and all the ripples add up.

## 8. How speech is made (so you can recognise the waveform)

| Sound | How it is made | What the wave looks like |
| :-- | :-- | :-- |
| Vowel (voiced) | Vocal folds buzz | Repeats again and again (roughly periodic) |
| /f/, /s/ (unvoiced fricative) | Air hisses through a narrow gap | Noisy, no clear repeats |
| Stop (like /p/) | Close, then burst open | A quiet gap, then a sudden pop |
| Silence | Nothing is being said | Very low energy |

/f/ is a **fricative** (friction hiss), made with lower lip and upper teeth. See [2026 Quiz 1 Q2](../week-1/week-1-exam-questions.md#ee4001_2026T1_Q1_NA--q2) and [2025 Quiz 1 Q2](../week-1/week-1-exam-questions.md#ee4001_2025T2_Q1_NA--q2).

## Check yourself

Try each one before you open the answer in your head.

1. A signal contains tones up to 3.4 kHz. What is the slowest safe sampling rate? *(Answer: 6.8 kHz.)*
2. At 12 kHz sampling, how many samples are in 20 ms? *(Answer: 240.)*
3. Why is $y[n]=x[n]+5$ not linear? *(Zero in gives 5 out.)*
4. Which is the "camera speed" and which is the "rounding of height": sampling or quantization? *(Sampling = camera speed; quantization = rounding of height.)*
5. In one sentence, why must the anti-alias filter come *before* sampling? *(After sampling the fast and slow tones already look identical.)*

## What to carry into Week 2

You now know how a sound becomes a list of numbers. Week 2 asks the next question: **which numbers matter?** A single long list is not much use, so we cut it into short slices and measure each slice. Continue with [Week 2 in plain language](week-2-eli12.md).
