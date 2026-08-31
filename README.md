# Snapdragon Vision (SV) API

## Introduction

This document describes the Snapdragon Vision Application Programming
Interface (SV API) suite. It is a comprehensive set of APIs designed to
empower developers to integrate powerful computer vision capabilities into
their applications effortlessly. With a focus on simplicity, flexibility,
and scalability, this API suite offers a wide range of features leverageable
across different business units and addressing a wide range of use cases.
Each API is designed to be hardware and software agnostic, ensuring
compatibility across various target platforms. Additionally, they are
implemented using C++ with an object-oriented design approach, providing
modularity, re-usability, and separation of concerns.

This repository packages the prebuilt SV libraries and headers for
Debian/Ubuntu based Qualcomm Linux targets.

### Practical Applications

The building blocks exposed by the SV API map directly onto common
embedded computer-vision workloads:

* Depth Estimation (Stereo Disparity) enables 3D perception for robotics,
  drones, and ADAS/surround-view systems.
* Feature Extraction and Matching (Descriptor, Descriptor Match, FPX, NCC,
  Pyramid FPX, Blob Detector, DetectComputeMatch) underpins visual
  odometry, SLAM, object recognition, and image stitching pipelines.
* Geometric Transformation (Warp) supports lens/distortion correction,
  image rectification, and multi-camera surround-view stitching.
* Motion Estimation (GME, LME, DL LME) is used for video stabilization,
  encoder pre-processing, and scene/motion analytics.
* Scaling (Scaler, Pyramid Scaler) supports multi-resolution processing
  pipelines feeding downstream CV/ML stages.
* Statistics (Spatial Statistics) provides frame/region statistics used
  for auto-exposure, auto-focus, and analytics pipelines.
* Semantic Segmentation & Tracking (Semantic Object Tracker) enables
  object tracking for surveillance, robotics, and automotive use cases.
* Facial Features enables face-detection driven applications such as
  camera auto-framing, access control, and user-presence detection.

### Features supported

* Depth Estimation
  * Stereo Disparity Estimation
* Feature Extraction and Matching
  * Descriptor
  * Descriptor Match
  * Feature Point Extraction (FPX)
  * Normalized Cross Correlation (NCC)
  * Pyramid FPX
  * Blob Detector
  * DetectComputeMatch
* Geometric Transformation
  * Warp
* Motion Estimation
  * Global Motion Estimation (GME)
  * Local Motion Estimation (LME)
  * Deep Learning Local Motion Estimation (DL LME)
* Scaling
  * Scaler
  * Pyramid Scaler
* Statistics
  * Spatial Statistics
* Semantic Segmentation & Tracking
  * Semantic Object Tracker (SOT)
* Facial Features
  * Facial Features

### SV Workflow

SV (libsv) is Qualcomm's vision-acceleration library, exposing a
session → feature → submit model over three possible backends: EVA
hardware, Hexagon DSP, or CPU/NEON fallback.

* A client calls `SV::Session::Create()` with a config (priority, perf
  mode, secure flag) and starts it.
* It creates one or more Feature objects on that session (e.g. LME,
  SOT, StereoDisparity, Scaler) and configures each.
* Work is submitted via `SubmitSync` / `SubmitAsync` / `SubmitFence`,
  returning results directly, via callback, or via a sync fence.
* Each feature reports which backends it supports; the library picks
  EVA hardware if present, otherwise falls back to DSP or CPU.
* EVA hardware path: `hfi_common_lib` speaks the HFI packet protocol to
  EVA/CVP firmware; `eclib` packs warp/geometry parameters for it.
* DSP/CPU path: `svswlib/common` holds shared algorithm logic,
  `svswlib/cpu` has NEON implementations, `svswlib/dsp` has Hexagon HVX
  implementations invoked over a FastRPC skel/stub.
* `sv/common/utils` provides shared cross-platform primitives
  (threading, sync, logging).
* A separate `sv-service` daemon handles lightweight debug-dump/SFR
  duties standalone.

### Building

The debian/rules file extracts prebuilt binaries from the tarball and installs them into the appropriate package staging directories under `data/<package-name>/arm64/`.

### Installation

Building produces two binary packages:

* `libsv1` — the SV runtime shared libraries.
* `libsv-dev` — headers, unversioned `.so` symlinks, and pkg-config
  files needed to build against SV; depends on `libsv1 (= ${binary:Version})`.

- Make sure to install [Fastrpc Debian package](https://github.com/qualcomm/fastrpc/tree/development) and the jsoncpp Debian
  package (`libjsoncpp`) otherwise the SV Debian package will give
  dependency errors.
- Install the package using command:
  `sudo dpkg -i qcom-sv-binaries1_1.0.0-1_arm64.deb`
- Once installation is complete one should see `libsv.so.1` in
  `/usr/lib/aarch64-linux-gnu`.

For development against the API, also install:

```
sudo dpkg -i libsv-dev_1.0.0-1_arm64.deb
```

Once installed, the libraries are placed under
`/usr/lib/<DEB_HOST_MULTIARCH>/` (e.g. `/usr/lib/aarch64-linux-gnu/`) and
headers under `/usr/include/`. Applications can pick up the compile/link
flags via `pkg-config` using the `.pc` file shipped in `libsv-dev`.

### Debugging

To enable runtime logs for debugging purposes use below command:<br>
  `adb shell setprop vendor.runtime.sv.debuglogen 1`<br>
It will enable additional runtime logs.

### Bug Reporting Guidelines

When reporting bugs, please provide the following details to facilitate debugging:<br>
- **Platform/SoC Name:** Specify the name of the platform or System on Chip (SoC) being used.
- **User Space Library Version/HLOS Build Details:** Include the version of the user space library and details of the High-Level Operating System (HLOS) build.
- **stdout & stderr for User Space:** Share the standard output and standard error logs for the user space.
- **User Library Logs:** Can be captured from `/var/log/syslog`.
- **Kernel Version:** Provide the version of the kernel.
- **dmesg Logs:** Include the dmesg logs.
- **QXDM Logs:** Provide QXDM logs for DSP failures.
- **Tests Run & Parameters:** Detail the tests that were run along with their parameters, including any environment variables explicitly set for SV, API used and any custom parameters given.
- **Custom Test Code:** If a custom test was conducted, please share a code snippet or the complete code to reproduce the issue.

### License

The SV libraries and headers (`Files: *`) are licensed under
Qualcomm's proprietary license; see `debian/copyright` for the full
license reference.

The packaging (`Files: debian/*`) is licensed under the BSD-3-Clause-Clear
License. See
[LICENSE.txt](https://github.com/qualcomm-linux/pkg-qcom-sv-binaries/blob/qcom/ubuntu/resolute/LICENSE.txt)
for the full text.
