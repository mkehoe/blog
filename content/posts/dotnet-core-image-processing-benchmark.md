---
date: "2020-06-09"
title: "Benchmarking .Net Core Platforms for Image Processing"
slug: "benchmarking-net-core-platforms-image-processing"
tags: [
    "dotnet core",
    "image processing",
    "web api",
    "mac",
    "windows",
    "docker"
]
categories: [
    "dotnet core"
]
disable_comments: true
---

Recently I had to build a CMS that had a web api that handled some image manipulations for the frontend. The project I was working on was already in .Net Core, so with the new cross platform support I wondered about the performance of it.

There are a lot of [benchmarks](https://devblogs.microsoft.com/dotnet/net-core-image-processing/) [around](https://www.imageflow.io/benchmarks/) [image resizing](https://awesomeopensource.com/project/saucecontrol/PhotoSauce). My needs were a little more complicated, so there are 2 benchmarks: resizing and tiling in the context of web api. Resizing is a standard image scaling, and tiling is downscaling the image and painting 4 copies into a single image.

### Benchmark 1

The first benchmarks are run synchronously(one request at a time). Timing is collected for both serverside operations and the total REST request. All runs were done on the same hardware, a Macbook Pro dualbooting Windows. The 4 platforms are Windows native, Mac native, Docker Mac hosted, and Docker Windows(WSL2).

###### Resize Synchronous

| Platform    | T1   | T2    | T3   | T4     | T5   | Total  |
| ----------- | ---- | ----- | ---- | ------ | ---- | ------ |
| Windows     | 0.24 | 9.49  | 0.52 | 37.82  | 4.51 | 57.66  |
| Docker WSL2 | 0.21 | 12.14 | 0.34 | 151.88 | 3.31 | 173.91 |
| Docker Mac  | 0.52 | 13.95 | 0.75 | 157.30 | 3.54 | 190.47 |
| MacOS       | 0.07 | 16.87 | 1.87 | 124.10 | 9.60 | 154.77 |

###### Tile Synchronous

| Platform    | T1   | T2    | T3   | T4     | T5   | Total  |
| ----------- | ---- | ----- | ---- | ------ | ---- | ------ |
| Windows     | 0.19 | 9.29  | 0.68 | 37.53  | 4.25 | 56.07  |
| Docker WSL2 | 0.09 | 12.19 | 0.28 | 181.67 | 2.61 | 202.41 |
| Docker Mac  | 0.52 | 12.94 | 0.57 | 196.14 | 2.88 | 225.32 |
| MacOS       | 0.08 | 16.85 | 1.81 | 201.17 | 6.97 | 227.02 |

Time intervals all in ms:

* _T1 - Copy to MemStream_
* _T2 - Create Bitmap_
* _T3 - Create New Bitmap_
* _T4 - Draw Image_
* _T5 - Save to Jpeg_

### Benchmark 2

To mimic more real time behavior of the server, the second benchmarks are collected asyncronously. Ten rounds of 12 requests are simultaneously run. The 12 requests are all submitted in tasks and waited on before the next round is started. 

###### Resize Asynchronous

| Platform    | T1    | T2    | T3   | T4     | T5    | Total  |
| ----------- | ----- | ----- | ---- | ------ | ----- | ------ |
| Windows     | 19.34 | 16.55 | 0.94 | 190.33 | 4.96  | 430.83 |
| Docker WSL2 | 1.32  |	28.01 |	0.85 | 325.59 | 5.36  | 451.71 |
| Docker Mac  | 16.94 | 31.11 | 1.17 | 303.01 | 5.05  | 596.43 |
| MacOS       | 6.76  | 50.01 | 2.23 | 369.65 | 20.63 | 470.48 |

###### Tile Asynchronous

| Platform    | T1    | T2    | T3   | T4     | T5    | Total  |
| ----------- | ----- | ----- | ---- | ------ | ----- | ------ |
| Windows     | 41.69 | 15.43 | 0.93 | 167.33 | 4.76  | 403.22 |
| Docker WSL2 | 2.1   |	31.29 | 0.95 | 454.74 | 4.55  | 587.96 |
| Docker Mac  | 20.28 | 38.1  | 2.3  | 443.85 | 4.9   | 760.68 |
| MacOS       | 6.01  | 50.78 | 1.63 | 613.96 | 14.83 | 733.47 |

#### Conclusions

As expected, Windows is the hands-down winner. Dotnet Core uses GDI+ for image manipulation and on Windows it is using the native api. Mac and Linux implementations are using libgdi+, which in this case is suboptimal. 

One oddity is the copy to memstream time (T1) on Windows Docker Linux containers. I suspect it an optimization for WSL2 and that using Hyper-V instead would be more in line with Docker on Mac. This merits future research.

Github links: [Web API](https://github.com/mkehoe/dotnetcore-imageprocessing) and [Benchmark App](https://github.com/mkehoe/benchmark-imageprocessing)
