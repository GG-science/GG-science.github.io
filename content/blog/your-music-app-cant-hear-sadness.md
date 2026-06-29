---
title: "Your Music App Can't Hear Sadness"
date: 2026-06-29
draft: false
tags: ["recsys", "audio", "embeddings", "machine-learning"]
description: "Audio models can tell a gym track from a chill track instantly. They can't tell a breakup song from a love song. That's not a bug — it's a property of the whole approach."
---

I built a music recommender that picks songs by sound — frozen audio embeddings, no metadata, no fine-tuning. It can tell a gym track from a chill track instantly.

It can't tell a breakup song from a love song.

They're acoustic twins: slow, sparse, minor key. I measured it across my context vectors — the *sad*, *romance*, and *chill* poles all sit at about 0.96 cosine. Effectively one point in space.

That's not a bug in my model, and it's not the wrong model. It's a property of the entire class of audio embeddings. Once you accept it, it changes what you build.

---

## What recommenders actually do

Most people assume music recommendation works by listening to the song. It mostly doesn't.

Every large recommender has the same two stages. First **retrieval**: from ~100 million tracks, narrow to a few hundred. Then **ranking**: order those few hundred for the moment, usually optimizing for whether you'll finish the track.

The real question is never "retrieval or not." It's *retrieve on what signal?* For the big players, the answer is **behavior** — collaborative filtering, "users like you played this." Not sound.

Audio shows up in one narrow place: cold-start. A brand-new track has no plays, so there's no collaborative signal to retrieve it. As Spotify's own [2014 writeup](https://sander.ai/2014/08/05/spotify-cnns.html) put it: *"if there is no usage data to analyze, the collaborative filtering approach breaks down — the cold-start problem."* Their fix was an audio model that predicts the track's behavioral vector from its sound.

So audio is a patch for the gap where behavior is missing. It is not what they retrieve or rank on. Behavior does that, because behavior predicts behavior.

---

## The valence ceiling

Here's why audio *can't* take over the taste job, even if you wanted it to. Audio carries energy well and emotion poorly — and the labs have published this themselves.

Deezer trained a model to predict mood from 18,000 tracks ([ISMIR 2018](https://arxiv.org/abs/1809.07276)). From audio alone:

| dimension | audio-only R² |
|---|---|
| arousal (energy) | 0.235 |
| valence (happy ↔ sad) | 0.179 |

The only thing that lifted valence was adding the **lyrics**. The benchmark consensus is blunter — the [DEAM study](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0173392) found algorithms show *"very good performance on arousal and completely unsatisfactory performance on valence."*

The reason is obvious once you say it out loud. A breakup song and a love song can be acoustically identical. What separates them lives in the lyrics and the context — *"I miss you"* over the same arrangement as *"I love you"* — and an audio model never reads the words. It's an information ceiling, not a data problem.

---

## A better encoder won't save you

The tempting fix is to swap encoders. So I ran the bake-off: CLAP vs MERT vs MuQ-MuLan, measuring how well held-out loved tracks separate from random ones.

CLAP came out 0.034 ahead of MERT. Except the honest separation, measured across 20 seeds instead of the best one, is 0.029 ± 0.011 — so that margin sits *inside* the noise. A non-result. (My own early notes quoted 0.045. That was the top seed. Report the distribution, not your favorite draw.)

MuQ-MuLan did earn a spot, but a narrow one. On free-text vibes, a music-trained model beats a general-audio one: ask for *"liminal, 3am, half-asleep"* and CLAP returns ABBA and Daft Punk while MuQ returns Swans and Portishead. Its authors [report](https://arxiv.org/abs/2501.01108) 79.3 vs 73.9 ROC-AUC over CLAP on a music-tagging benchmark. Real, but modest, and only on text matching.

The punchline: switching to MuQ did not fix valence. Sad and romance still collapse. That's a property of embedding music as audio at all — not of which audio model you pick. You can't encoder-swap your way out of it.

---

## What actually works

You don't embed your way to taste. You learn it.

Freeze the encoder — it's a great texture detector and a terrible taste model. Put a small per-user head on top: a logistic probe on the frozen embeddings, shrinkage-regularized, that learns the direction in audio space meaning *your* taste rather than generic texture.

Then let feedback carry the part audio can't. A skip is the acoustic shadow of valence. I never label a song "sad" — I learn that this neighborhood, for you, right now, is wrong. Behavior supplies what the waveform drops.

And make the moment explicit. The big apps keep your mood and moment latent — inferred from clicks, never asked. I let you name it, and retrieve on it directly.

That's the whole bet. They retrieve on behavior and patch the gaps with audio. I retrieve on a frozen per-user audio taste lens, fired at a moment you name, with feedback supplying the emotion the audio can't hear.

---

One honest caveat: nearly every number here is from one user's library — mine. That buys internal validity, not a benchmark. The effect sizes are small and none of it generalizes until there are more testers.

But the shape holds, and it's the useful part. Audio embeddings hear texture brilliantly and taste barely at all. The interesting work isn't the encoder. It's the lens you learn on top of it.
