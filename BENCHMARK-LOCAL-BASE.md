# Benchmark: local base model vs Gemini vs ffmpeg9-whisper (2026-10-27)

Same 2 audios, same scoring script, same normalizer for all 3 systems.
No post-processing beyond normalization: lowercase, punctuation to space,
accents kept. What a system emitted counted, including bracketed tags.

## Corpus

- 3 voices: 134.16 s, 28 turns, 302 words in the answer key.
  Speakers MARCOS, BIA, LEO.
- 5 voices: 172.38 s, 43 turns, 436 words in the answer key.
  Speakers ANA, TATI, RAFA, PAULO, CAROL. Two scenes: a noisy bar
  and a video call.

## Systems (all base-size models)

- Whispero local 0.8.0, clean install:
  `whispero --provider local --model base --lang pt --diarize --format json`
- Gemini, `gemini-3.1-flash-lite`, faithful transcription prompt, text only.
- ffmpeg9-whisper, FFmpeg 9.0 with whisper.cpp, model `ggml-base.bin`,
  `language=pt`, `format=json` (sequential segments, no SRT window overlap).

## Metrics

- WER: word error rate against the answer key (substitutions +
  deletions + insertions, divided by answer-key words). Lower is better.
- Speaker accuracy: per turn, by largest time overlap between the
  answer-key turn and the predicted segment, with label-to-speaker
  mapping by total overlap. Only Whispero has diarization in this mode;
  Gemini and ffmpeg are text-only, so speaker accuracy is N/A, never 0.
- Wall time: elapsed time per run. RTF is wall time divided by audio
  duration; below 1 means faster than real time.

## Results

| System | Audio | WER | Speaker accuracy | Speakers detected | Wall time | RTF |
|---|---|---|---|---|---|---|
| Whispero local 0.8.0 base | 3 voices, 134 s, 302 words | 8.9% | 27/28, 96.4% | 3 of 3 | 33.9 s | 0.25 |
| Gemini gemini-3.1-flash-lite | 3 voices, 134 s, 302 words | 6.6% | N/A, text only | N/A | 4.8 s | 0.04 |
| ffmpeg9-whisper ggml-base | 3 voices, 134 s, 302 words | 30.1% | N/A, text only | N/A | 128.8 s | 0.96 |
| Whispero local 0.8.0 base | 5 voices, 172 s, 436 words | 14.0% | 38/43, 88.4% | 5 of 5 | 39.4 s | 0.23 |
| Gemini gemini-3.1-flash-lite | 5 voices, 172 s, 436 words | 8.0% | N/A, text only | N/A | 5.3 s | 0.03 |
| ffmpeg9-whisper ggml-base | 5 voices, 172 s, 436 words | 28.9% | N/A, text only | N/A | 149.2 s | 0.87 |

## Reading

- Gemini wins on text in both audios: 6.6% vs 8.9% on 3 voices,
  and 8.0% vs 14.0% on 5 voices.
- Whispero is second on text and the only system here that identifies
  speakers: 96.4% of turns on 3 voices, 88.4% on 5 voices, with the
  correct speaker count detected in both.
- ffmpeg is far behind on both counts despite a base model of the same
  size: WER near 30% in both audios, no diarization, and the slowest,
  close to real time.
- The Whispero-to-Gemini gap grows with speaker count: 2.3 points on
  3 voices, 6.0 points on 5 voices.

## Caveats

- Gemini wall time is API round-trip time, not local compute, and it
  needs internet and a key. Whispero runs fully offline.
- Gemini varied between two runs: 7.0% to 6.6% on 3 voices and
  7.8% to 8.0% on 5 voices. Whispero and ffmpeg repeated their exact
  WER on both runs.
- A short 6-second single-voice clip was left out of the table because
  it has no independent answer key; its reference would be a consensus
  of the systems themselves, not an external source.
- This benchmark measures transcription and diarization only, not
  subtitling, translation, or audio analysis.
