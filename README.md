# Envision Gesture Challenge 2026 — PDR + TCN

## Image availability

Docker Hub: [diobrando3379/egc-pdr, tag v4](https://hub.docker.com/r/diobrando3379/egc-pdr/tags?name=v4)

Image: `docker.io/diobrando3379/egc-pdr:v4` (Linux/amd64). All weights are bundled in this image.

Tested local image identifier:

`sha256:d328c20bb58e4828e7daf57089fe0ef8c84eb78a499d7cdb34a284012be37ed9`

The identifier above records the tested local image. Use the registry digest returned by `docker pull` when pinning a downloaded copy.

## Running the model

Pull the image once:

```bash
docker pull docker.io/diobrando3379/egc-pdr:v4
```

The image contains all model weights and inference dependencies, and inference requires no network access.

Clip leaderboard:

```bash
docker run --rm --network none --gpus all \
  -v /absolute/path/to/input:/input:ro \
  -v /absolute/path/to/output:/output \
  docker.io/diobrando3379/egc-pdr:v4 --input /input --output /output --mode clip
```

Video leaderboard:

```bash
docker run --rm --network none --gpus all \
  -v /absolute/path/to/input:/input:ro \
  -v /absolute/path/to/output:/output \
  docker.io/diobrando3379/egc-pdr:v4 --input /input --output /output --mode video
```

Please explicitly select `clip` or `video` for the respective evaluation. The default probability threshold is 0.42. No additional input is required. Place NPZ files directly in the input directory. MP4 files are ignored. Class labels, speaker identifiers and corpus identifiers are not used as prediction features; FPS and frame count are used for temporal alignment.

For every `<name>.npz`, the model writes:

- `<name>_predictions.csv`, containing one row per original frame and the columns `frame_index`, `prediction`, `confidence`, `Gesture_confidence`, `NoGesture_confidence`.
- `<name>.eaf`, containing ELAN annotations.

Labels are `Gesture` and `NoGesture`; Move is treated as NoGesture. In clip mode every frame receives the clip decision. In video mode the model produces temporal segments.

## Hardware and implementation

The image is based on PyTorch 2.7.1 with CUDA 12.8. GPU execution requires an NVIDIA driver compatible with CUDA 12.8 and NVIDIA Container Toolkit.

Previous local tests on an RTX 5070 Ti, 16 GB: approximately 36–40 seconds for 300 clips, approximately 195 seconds for 341 seconds of synthetic concatenated video, and approximately 2.3 GB peak GPU memory. These are hardware-specific timing measurements, not accuracy measurements on continuous recordings. CPU fallback is available but substantially slower.

The system averages 15 PDR models (joint, bone and motion; five speaker folds per stream) and 10 TCN models (joint and motion; five folds per stream), with total family weights 0.75 and 0.25. It uses core-aware 25 fps inputs and horizontal-flip averaging. The fixed threshold is 0.42. The submitted v4 image does not implement the later experimental `--pos-rate` option.

## Development evaluation

Five speaker-independent folds; ZHUBO is training-only. The following are development out-of-fold results on 24,054 clips, not official challenge test scores:

| AUC | Gesture-F1 | Macro-F1 | Balanced accuracy |
|---|---|---|---|
| 0.863024 | 0.776034 | 0.775127 | 0.776681 |

Threshold and model choices used development predictions, so these are not estimates from an untouched final test set. They are not directly comparable with published baselines using different folds. Continuous-video detection accuracy has not been established on a representative labeled continuous-video test set.

On 2026-10-21, offline GPU execution was rechecked using 14 clips from seven corpus partitions. Both explicit modes produced 1,512 frame predictions with valid CSV fields and parseable ELAN files. All 65 source, configuration and weight files inspected in the image matched the original local submission bundle by SHA-256. These checks establish packaging and output format, not leaderboard performance.