---
title: "Your Music App Can't Hear Sadness"
date: 2026-06-29
draft: false
tags: ["recsys", "audio", "embeddings", "machine-learning"]
description: "I cancelled Spotify, then spent 18 months building the recommender I wanted. The first thing it taught me: audio models hear energy fine and emotion barely at all."
---

I cancelled Spotify about a year and a half ago. I had fed it years of listening and hundreds of artists, and it still looped the same popular names back at me. Anything genuinely new, I had to go dig up myself. I left a note in the cancellation box about why I was leaving, and then spent eighteen months building the thing I had wanted it to be.

The first version picked songs by sound: frozen audio embeddings, no metadata, no fine-tuning. Hand it a track and it finds others that sound like it. Energy came for free. Ask for a workout song and you get one. Ask for something to fall asleep to and the room goes quiet.

Then I asked it for a sad song, and it gave me a love song.

So I measured why. I keep a set of reference vectors for different moods, and when I checked the distances between them, "sad," "romance," and "chill" were all sitting around 0.96 cosine similarity. For any practical purpose that is a single location. The model had no axis that pulled heartbreak away from tenderness. It could hear that a song was slow and quiet. It could not hear which kind of slow and quiet.

That is not a quirk of my setup. It is how this whole family of models behaves, and understanding why is what changed everything I built afterward.

## What a recommender actually does

You would assume a music app recommends music by listening to it. Mostly it does not.

Every large recommender runs in two stages. Retrieval first: out of roughly a hundred million tracks, pull the few hundred worth a closer look. Then ranking: take those few hundred and put them in order for the moment, usually tuned to whether you will play a track to the end.

The question that matters is what signal drives the first stage. For Spotify, Deezer, and everyone else at scale, it is your behavior. Collaborative filtering, the "people who played this also played that" machinery. Not the audio.

Audio does appear, in one specific corner: cold start. A track released yesterday has no plays, so there is no behavioral signal to retrieve it with. Spotify [wrote this up](https://sander.ai/2014/08/05/spotify-cnns.html) back in 2014: "if there is no usage data to analyze, the collaborative filtering approach breaks down — the cold-start problem." Their fix was an audio model that guesses a new track's behavioral vector from its sound, so the song can sit next to similar ones until real play data shows up.

Audio is the patch they reach for when behavior is missing. The rest of the time they search on behavior, because what you have already played predicts what you will play next far better than what the music sounds like.

## Why the sound can't carry the mood

Maybe valence, the happy-to-sad axis, is just hard for my particular model. It is not a me problem. The labs that do this for a living have hit the same wall and published it.

Deezer trained a model to read mood off 18,000 tracks and presented it at [ISMIR 2018](https://arxiv.org/abs/1809.07276). From audio alone, it predicted arousal, how energetic a track is, at an R² of 0.235, and valence at 0.179.

| dimension | audio-only R² |
|---|---|
| arousal (energy) | 0.235 |
| valence (happy ↔ sad) | 0.179 |

The only thing that moved valence was feeding in the lyrics. The [DEAM benchmark](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0173392) said it more plainly: across systems, "very good performance on arousal and completely unsatisfactory performance on valence."

Think about what valence actually is. Whether a song reads as joyful or wrecked is carried mostly by the words and the world around them: who is singing, to whom, in what tradition. Run a breakup ballad and a love ballad through an audio encoder and you throw all of that away. What is left is tempo, key, timbre, dynamics. On those axes the two songs are often the same song. The model is not underfitting the mood. The mood was never in the file you gave it, and no amount of extra data recovers a signal the input does not contain.

## A better encoder doesn't fix it

The obvious next move is to blame the encoder and swap in a better one. So I ran the bake-off: CLAP against MERT against MuQ-MuLan, each scored on how cleanly a held-out set of tracks I love separated from random ones.

CLAP came out 0.034 ahead of MERT. Then I stopped trusting the single best run and measured across twenty random seeds. The real separation was 0.029 plus or minus 0.011, which puts a 0.034 lead well inside the noise. It was nothing. My own first writeup had quoted 0.045, which was simply the luckiest seed. Report the spread, not your favorite draw.

MuQ earned its place in exactly one spot. On free-text vibes, a model trained only on music beats one trained on general audio. Type "liminal, 3am, half-asleep" and CLAP gives you ABBA and Daft Punk; MuQ gives you Swans and Portishead. Its authors [put it](https://arxiv.org/abs/2501.01108) at 79.3 against CLAP's 73.9 on a music-tagging benchmark. A real gain, on text matching, and nowhere else.

None of it touched the original problem. With MuQ running, sad and romance still fell onto the same point. The blind spot lives in the act of turning music into audio, not in the choice of audio model. There is no encoder waiting on the far side of it that solves this.

## What I built instead

If the sound cannot carry taste, taste has to come from somewhere else.

So I stopped asking the encoder to be clever and let it be a sensor. It is an excellent texture detector, so I freeze it and never touch its weights. On top of it I fit one small thing per listener: a logistic probe over the frozen vectors, regularized to lean on the encoder less the more it learns about you. It does not try to read your taste out of the audio. It learns the direction, inside the encoder's own space, that points toward the music you keep.

The part the audio cannot hear, I take from what you do. A skip is the nearest thing to a free valence label I will ever get. Not a tag, just a vote that this pocket of sound, for you, in this moment, is wrong. I never rule that a song is sad. I let the rejections carve out the shape the waveform left blank.

And the moment goes on the surface, where you can reach it. Spotify keeps your mood and your context buried, inferred from clicks and never asked out loud. I let you say it, and I retrieve against it directly.

That is the whole wager. They search on behavior and reach for audio when behavior runs out. I search on a frozen per-listener audio lens, aimed at a moment you name, with your own feedback filling in the feeling the sound never carried.

The numbers here are from my own library so far, and I am opening it up to more listeners now. The shape has held the whole way: audio tells you a great deal about how a song moves and almost nothing about how it feels. Whether that lens actually beats the raw sound is the question I am testing next.
