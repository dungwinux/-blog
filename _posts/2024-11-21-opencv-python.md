---
layout: post
title: "Building OpenCV with CUDA for Python in 3 lines through conda"
date: 2024-11-21 23:15:00 +0000
categories: pet_project
tags: software
excerpt: Keeping the commands as simple as possible
---

As part of a group project in one university course, I had to pick up OpenCV.
Our group agreed to use Python, so I need to install OpenCV for Python. Should be easy, my past self thought.

> Pre-built CPU-only OpenCV packages for Python.
>
> Check the manual build section if you wish to compile the bindings from source to enable additional modules such as CUDA.
>
> ~ [https://github.com/opencv/opencv-python?tab=readme-ov-file#opencv-on-wheels](opencv-python: OpenCV on Wheels)

But reality is cruel. the _opencv-python_ package on pypi.org is only built for CPU.
The performance is not terrible, to be honest, but it does not match the scale of the project, especially when I have a GPU in the machine already.

I had a little hope when I saw [this](https://github.com/cudawarped/opencv-python-cuda-wheels) project that builds _opencv-python_ against CUDA, Nvidia Video Codec SDK and cuDNN.
But the Python code still does not run. Thus, I decided to make myself a build.

Now, many online solutions are pretty manual: `git clone`, `apt install`, etc. Instead, I devised a solution that utilizes the tools for Python: `pip` and `conda`. (`mamba` is `conda`-compatible, so the instructions apply to `mamba` as well).
We will perform on Ubuntu and `bash`, other environment can also try with few adjustments.

First, we create a new environment using `conda`. Here I chose the name _compvision_ and selected python 3.11. Since NVIDIA has provided conda package for the CUDA SDK, I include it in the command:

```bash
conda create -n compvision python=3.11 cuda cudnn -c nvidia
```

There is only one small issue in the package which makes `cicc` invisible. This is necessary if you are targeting specific `CUDA_ARCH_BIN`. We can fix it by creating a link in `bin` folder of the virtual environment.

```bash
ln -s $CONDA_PREFIX/nvvm/bin/cicc $CONDA_PREFIX/bin/cicc
```

Now we will build from source _opencv-python_ using `pip`. Since we are building against CUDA, we must use the _contrib_ variant. And if you don't need GUI interface like me (using OpenCV through Jupyter notebook), then you can opt-in the _headless_ variant. I also disabled OpenCL since the build would try compiling Intel VAAPI and fail. The following is my build command:

```bash
CMAKE_ARGS="\
  -DWITH_CUDA=ON -DWITH_CUDNN=ON -DOPENCV_DNN_CUDA=ON \
  -DWITH_NVCUVID=OFF -DCUDA_ARCH_BIN=7.5 -DWITH_CUBLAS=ON \
  -DWITH_OPENCL=OFF \
  " pip install --no-binary opencv-contrib-python-headless opencv-contrib-python-headless
```

Optionally, if you like the library to run faster while trading off floating-point arithmetic precision, you can add `-DENABLE_FAST_FATH=ON -DCUDA_FAST_MATH=ON` in `CMAKE_ARGS`. You also can find more compile examples/flags in [StackOverflow](https://stackoverflow.com/questions/70334087/how-to-build-opencv-from-source-with-python-binding) and [OpenCV docs](https://docs.opencv.org/4.x/db/d05/tutorial_config_reference.html).

Now you can go have a coffee break while waiting for it to build. For this part, it is likely there are multiple compiler errors due to dependencies, and I will not account for them here since it varies depending on the machine. The majority of issues would likely be missing libraries or files located in a different path. If you are using `apt`, you can use `apt-file` to search for packages including the missing files.

After the build is complete (which took me roughly 30 minutes), we can check in Python by:

```py
import cv2
print(cv2.getBuildInformation())
```

And there we go:

```

General configuration for OpenCV 4.10.0 =====================================
  Version control:               unknown

  Extra modules:
    Location (extra):            /tmp/pip-install-q_5601f_/opencv-contrib-python-headless_8899128e35a84dcb90525b8113d5a9a5/opencv_contrib/modules
    Version control (extra):     unknown

  Platform:
    Timestamp:                   2024-11-20T04:43:55Z
    Host:                        Linux 6.8.0-45-generic x86_64
    CMake:                       3.31.0
    CMake generator:             Ninja
    CMake build tool:            /usr/bin/ninja
    Configuration:               Release

  CPU/HW features:
    Baseline:                    SSE SSE2 SSE3
      requested:                 SSE3
    Dispatched code generation:  SSE4_1 SSE4_2 FP16 AVX AVX2 AVX512_SKX
      requested:                 SSE4_1 SSE4_2 AVX FP16 AVX2 AVX512_SKX
      SSE4_1 (16 files):         + SSSE3 SSE4_1
      SSE4_2 (1 files):          + SSSE3 SSE4_1 POPCNT SSE4_2
      FP16 (0 files):            + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 AVX
      AVX (8 files):             + SSSE3 SSE4_1 POPCNT SSE4_2 AVX
      AVX2 (36 files):           + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2
      AVX512_SKX (5 files):      + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2 AVX_512F AVX512_COMMON AVX512_SKX
...
```
