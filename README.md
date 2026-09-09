# nomercy-stemsplit-models

Pre-built source-separation models in GGUF format for the `stemsplit` audio
filter of [nomercy-ffmpeg](https://github.com/NoMercy-Entertainment/nomercy-ffmpeg).

Every release carries the model files, one `.sha256` sidecar per file, a
`manifest.json` and its detached signature `manifest.json.sig`, produced the
same way as the [whisper models](https://github.com/NoMercy-Entertainment/nomercy-whisper-models).
The NoMercy MediaServer downloads the latest release at start-up, verifies the
manifest signature and the file hashes, and places the model next to its
ffmpeg binary.

## Models

| file | architecture | source | licence |
|---|---|---|---|
| `spleeter-2stems-f16.gguf` | `spleeter-unet-v1`, two U-Nets (vocals, accompaniment), fp16 weights, ~39 MB | Deezer Spleeter `2stems` checkpoint, release v1.4.0 | MIT (Deezer) |

## How a release is built

The `Release` workflow (`.github/workflows/main.yml`, run by hand) clones
nomercy-ffmpeg at a chosen ref, builds the converter image in
`tools/spleeter-gguf`, downloads Deezer's checkpoint inside that image,
converts it with `convert.py` (which verifies every tensor name, shape and
dtype before it writes anything), and publishes a release tagged
`v<year>.<month>.<day>` with the signed manifest.

Nothing model-related is committed to this repository; the model is a build
artifact regenerated from the published checkpoint.

## Consuming the model

The filter takes the model as a path. On Windows a drive letter's colon splits
the filter argument, so either run ffmpeg with the model's directory as the
working directory and pass a relative name, or escape the colon as `\\:`.

```
ffmpeg -i in.flac -filter_complex "[0:a]stemsplit=model=spleeter-2stems-f16.gguf[voc][acc]" \
  -map "[voc]" vocals.wav -map "[acc]" accompaniment.wav
```
