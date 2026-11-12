---
title: "Week 2 in plain language: slicing sound and describing it with MFCCs"
week: 2
tags: [speech/eli12, speech/week-02]
---

# Week 2 in plain language: slicing sound and describing it with MFCCs

**Explain-like-I'm-12 version** of the [full Week 2 notes](../week-2/week-2-notes.md). Exam wording lives in the [exam-focused guide](../week-2/week-2-exam-focused-notes.md) and the [Week 2 past papers](../week-2/week-2-exam-questions.md). Start with [Week 1 in plain language](week-1-eli12.md) if sampling is new to you.

> [!NOTE]
> **The big idea of Week 2.** A recogniser should not stare at raw numbers. It should look at short slices of sound and describe each slice with a few numbers that capture *what the sound is like*, roughly the way our ears do. Those few numbers are called **MFCCs**.

## 1. The voice is a buzzer plus a tube

Your voice works like a trumpet:

- The **vocal folds** buzz, like the player's lips. This buzz has a **pitch** ($F_0$): how many buzzes per second.
- The **vocal tract** (throat, mouth, nose) is the tube. Its shape adds **resonances** that boost some frequencies. These boosted humps are called **formants**, and they are what make an "ee" sound different from an "ah".

Change the tube shape and the formants move while the pitch stays the same. Change the buzz speed and the pitch moves while the tube stays the same. Two separate things.

## 2. How we hear

- **Frequency** is physical (measured in Hz). **Pitch** is what you *feel*. The exam asks for the psychological partner of frequency: **pitch** ([2023 Quiz 1 Q13](../week-2/week-2-exam-questions.md#ee4001_2023T3_Q1_NA--q13)).
- Inside your ear, a membrane called the **basilar membrane** responds to different frequencies at different places, so it acts as a built-in frequency analyser ([2026 Quiz 1 Q4](../week-2/week-2-exam-questions.md#ee4001_2026T1_Q1_NA--q4)).
- **Masking:** a loud sound can hide a quiet one at a nearby frequency. The quiet one is still in the recording; you just cannot detect it.
- The **Mel scale** stretches low frequencies and squeezes high ones, because our ears separate low tones more finely than high tones.

## 3. Why slice the sound?

Speech keeps changing: you move your tongue and lips all the time. If you analyse a whole sentence at once, you learn *which* frequencies appeared, but not *when*.

So we cut the signal into short slices called **frames**, treat each one as "almost steady" for a moment, and measure each frame. Over a whole sentence it is non-stationary (always changing); inside one short frame it is roughly stationary (nearly steady). Both statements are true at different zoom levels.

The recipe:

1. Take a short piece (a frame).
2. Fade its edges using a **window** (a smooth taper, so abrupt cut-offs do not cause fake frequencies).
3. Compute the frequencies in it (DFT).
4. Slide forward a little (the **hop**) and repeat.

Stacking all the results side by side gives a **spectrogram**: time across, frequency up, brightness for loudness.

### Frame arithmetic you can do in your head

At 16,000 samples per second:

| Thing | Time | Samples |
| :-- | :-- | :-- |
| Window (frame length) | 25 ms | $16000 \times 0.025 = 400$ |
| Hop (step) | 10 ms | $16000 \times 0.010 = 160$ |
| Overlap between neighbours | | $400 - 160 = 240$ |

One second of audio gives $1 + \lfloor (16000 - 400)/160 \rfloor = 98$ complete frames. Because frames overlap so much, neighbouring frames look alike. That is exactly the observation in [2026 End Term Q12](../week-2/week-2-exam-questions.md#ee4001_2026T1_ET_FN--q12): adjacent feature vectors are **correlated**.

### Long window or short window?

You cannot have everything:

| Window | You see clearly | You see poorly |
| :-- | :-- | :-- |
| **Long** | Fine frequency detail (separate harmonics) | *When* things happen |
| **Short** | *When* things happen | Fine frequency detail |

Long windows give a **narrowband** spectrogram (thin horizontal stripes). Short windows give a **wideband** one (vertical pitch stripes). Rule: **longer window, better frequency resolution** ([2026 Quiz 1 Q3](../week-2/week-2-exam-questions.md#ee4001_2026T1_Q1_NA--q3) discusses the stationarity wording).

## 4. Cepstrum: un-mixing the buzzer and the tube

In the spectrum the buzz and the tube are *multiplied* together, which is hard to separate. A **logarithm** turns multiplying into adding, and adding is much easier to pull apart. Doing a transform on the log-spectrum gives the **cepstrum**. Its horizontal axis is called **quefrency**. The slow, smooth part (the tube) lands at low quefrency; the fast ripples (the buzz) land elsewhere. Keeping only the low part is **liftering**.

## 5. MFCCs, one arrow at a time

MFCC stands for **Mel-Frequency Cepstral Coefficients**. Here is why each step exists:

| Step | What happens | Why, in plain words |
| :-- | :-- | :-- |
| 1. Frame + window | Cut a short, faded slice | Speech is steady only for a moment |
| 2. DFT, then power | Find how much energy at each frequency | Describe the slice by frequency |
| 3. **Mel filter bank** | Add up energy in overlapping triangular bands spaced like our ears | Treat frequencies the way hearing does; fewer, smarter numbers |
| 4. **Log** | Take the logarithm | Squash huge differences in loudness; turns products into sums |
| 5. **DCT** | Mix the band numbers with cosines, keep the first ~13 | Compact the information and make the numbers less tangled together |

A tiny filter example: powers $[1, 4, 1]$ with triangle weights $[0.5, 1, 0.5]$ give energy $0.5(1) + 1(4) + 0.5(1) = 5$, and its log is $\ln 5 \approx 1.609$. Do this *per filter*, and take the log of the filter's total, not of each piece.

What MFCCs are: a **short-term spectral description on a perceptual scale**. What they are not: they are not a pitch detector and they do not measure phase. They are also **not** forbidden in modern systems; log-Mel features without the DCT are used too.

Extras: **deltas** measure how a coefficient *changes* between frames. 13 static + 13 delta + 13 delta-delta = **39 numbers per frame**.

The five-mark question ([2025 Quiz 2 Q7](../week-2/week-2-exam-questions.md#ee4001_2025T2_Q2_NA--q7)) wants the ordered pipeline *and* the reason for each stage, not just "FFT, Mel, DCT".

## 6. A quick Gaussian refresher

Measure the same thing many times (say one MFCC value in many frames) and you get a cloud of values. A **Gaussian** (bell curve) describes that cloud with two numbers:

- the **mean** $\mu$: the centre,
- the **variance** $\sigma^2$: the spread.

Facts the exam repeats:

- The total area under the curve is 1.
- It is symmetric about the mean.
- The height of a density curve is **not** a probability. It can even be larger than 1 for a narrow curve. Probability comes from the **area** over a range.

Fitting one to data $[1, 2, 3]$: mean $= 2$; squared gaps are $[1, 0, 1]$; variance $= 2/3$ (divide by $N$, the maximum-likelihood way).

## Check yourself

1. At 16 kHz, how many samples are in a 25 ms frame? *(400.)*
2. You want to separate two close harmonics. Long or short window? *(Long.)*
3. Put these in order: log, Mel filters, DCT, power spectrum. *(Power spectrum → Mel filters → log → DCT.)*
4. Can an MFCC be negative? *(Yes. Logs and signed cosine sums can be negative; it is not a probability.)*
5. Can a Gaussian density value exceed 1? *(Yes, if the curve is narrow. Only the total area must be 1.)*

## What to carry into Week 3

You now have a compact feature vector per frame. Week 3 asks: **what shape does the cloud of such vectors have, and how do we model clouds with several bumps?** Continue with [Week 3 in plain language](week-3-eli12.md).
