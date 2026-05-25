# Project 4: CUDA Applications

## Overview

This project contains two CUDA programs. The first program implements an iota-style array fill using CUDA. The second program generates a Julia/Mandelbrot-style fractal image using CUDA.

## Program 1: CUDA iota

The iota program fills an array so that each position contains its index plus a starting value.

The CPU version uses a normal loop through the array. The CUDA version launches one GPU thread per array element. Each thread computes its own index and writes one value into the array.

The basic operation is:

    values[i] = i + startValue;

## iota timing results

CPU results:

| Vector Length | Wall Clock Time | User Time | System Time |
|:--:|--:|--:|--:|
| 10 | 0.00 | 0.00 | 0.00 |
| 100 | 0.00 | 0.00 | 0.00 |
| 1000 | 0.00 | 0.00 | 0.00 |
| 10000 | 0.00 | 0.00 | 0.00 |
| 100000 | 0.00 | 0.00 | 0.00 |
| 1000000 | 0.00 | 0.00 | 0.00 |
| 5000000 | 0.02 | 0.00 | 0.02 |
| 100000000 | 0.56 | 0.09 | 0.46 |
| 500000000 | 2.81 | 0.44 | 2.36 |
| 1000000000 | 5.61 | 0.90 | 4.71 |
| 5000000000 | 34.26 | 6.04 | 28.21 |

GPU results:

| Vector Length | Wall Clock Time | User Time | System Time |
|:--:|--:|--:|--:|
| 10 | 0.36 | 0.01 | 0.32 |
| 100 | 0.26 | 0.00 | 0.24 |
| 1000 | 0.26 | 0.00 | 0.23 |
| 10000 | 0.26 | 0.01 | 0.23 |
| 100000 | 0.26 | 0.01 | 0.23 |
| 1000000 | 0.27 | 0.01 | 0.23 |
| 5000000 | 0.28 | 0.02 | 0.24 |
| 100000000 | 1.04 | 0.33 | 0.69 |
| 500000000 | 3.29 | 0.71 | 2.56 |
| 1000000000 | 6.54 | 1.45 | 5.06 |
| 5000000000 | 43.06 | 11.59 | 31.46 |

## Are the iota results what I expected?

The results were not exactly what I expected. I expected the CUDA version to be faster because the GPU has many cores. However, the GPU version was slower.

CUDA is not a great solution for this problem because each array element requires very little work. Each thread only calculates an index, adds the starting value, and writes one number. The overhead of GPU memory allocation, memory copies, kernel launch, and synchronization is large compared to the amount of computation.

## Program 2: CUDA Julia/Mandelbrot Set Generator

The Julia/Mandelbrot program generates a PPM image. This is a better fit for CUDA because each pixel can be computed independently.

The CUDA version assigns one GPU thread to each pixel. Each thread maps its pixel to a point in the complex plane, repeatedly applies the fractal equation, counts the number of iterations before escape, and assigns a color.

## Generated image

![Generated Mandelbrot image](julia.ppm)

Caption: Generated Mandelbrot-style image using the default starting value z = 0 + 0i.

## Build and run

    make
    ./runTrials.sh ./iota.cpu
    ./runTrials.sh ./iota.gpu
    ./julia.gpu
