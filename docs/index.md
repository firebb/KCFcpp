# C++ Parallel KCF Tracker
Team members:
- Yuchen Huo (<yhuo1@andrew.cmu.edu>)
- Danhao Guo (<danhaog@andrew.cmu.edu>)

# Summary
We are going to extend the the Kernalized Correlation Filter (KCF) [1, 2], a fast oject tracker, to a multicore platform and even GPU.

# Background
Visual oject tracking is one of the popular tasks in the computer vision area. There are a lot of different implementations focusing on how to improve the accuracy of the tracking algorithm. These implementations mostly target at the "real time" systems, these algorithms have been aimed at approximate frame rate of 30 Frames Per Second(FPS), which is enough for previous devices and workloads. However, for higher frame rate cameras (240 FPS) or an offline video processing pipeline, 30 FPS still seems to be slow. We look through some of the benchmark results [3, 4] and find this Kernalized Correlation Filter which provide top speed on CPU and pretty good accuracy. We decide to further extend this algorithm to be able to benefit from multicore systems and be fast enough to be used offline.

By creating the circulant training data matrix, KCF uses Discrete Fourier Transform (DFT) to compute the close form solution of Ridge Regression and reduce both the storage and computation. It further uses HOG feature instead of raw pixels to gain better mean precision on the testing dataset. The main computation lies in the dft and HOG feature extraction. There are plenty of fast parallel algorithms for dft and HOG feature extraction should also be able to be parallelized so we decide to build the parallel oject tracker based on the KCF implementation. 

Fast Fourier Transforms (FFT) is an efficient way for calculation of DFT by utilizing DFT's symmetry properties. Compared with $O(n^2)$ computation complexity, the computational cost of FFT is only $O(n\log{n})$ It is a key tool in most of digital signal processing systems including object tracking ones. In the KCF object tracking algorithm, FFT is heavily used for learning and evaluation to achieve lower computational complexity. For example, by utilizing FFT operation, KRLS learning algorithm is improved from $O(n^4)$ to only $O(n^2\log{n})$ for nxn 2D images.

# Challenge
The biggest challenge is that we are not very familiar with the computer vision algorithm so it may take time us to understand the implementation details of the KCF tracker. However, since we have the open source implementation, we may not need to understand every single lines in the code. Instead, we may identify the paralizable parts and do optimization on these parts.

Although FFT brings out great improvement on performance, it is still a computational intensive with $O(n\log{n})$ complexity. According to the properties of FFT algorithm itself, we believe that there should be some computational independent tasks inside it such as the DFT calculation of even terms and odd terms in each iteration. So, it may benefit from parallel implementation on multi-threaded processing. To implement FFT in parallel, the major points is that multi-dimensional FFT is utilized. In this regard, analyzing computation independence becomes tough. Meanwhile, applying such complicated algorithm with parallel technologies especially AVX instructions are pretty hard. We plan to combine both AVX, OpenMP and MPI to support different level of parallelism for FFT.

# Resources
The Author of KCF open source their [code](https://github.com/joaofaro/KCFcpp) on the github so we are able to build our parallel version based on their implementation. We would develop and test our software on the GHC machines which have 8 cores and we may further test the performance on Xepm Phi machines or develop the GPU version to see how our implementation scales if everything goes smoothly.

# Goals
Minimum Goal: ReImplement FFT algorithm to support multicore parallelism. The parallel FFT should be faster than the original FFT implementation in KCF and improve the overall performance of KCF algorithm in multi-core machines.

Expected Goal: The parallel FFT could achieve sub-linear scalability with the increase of core number in the machine. 

Stretch Goal: The parallel FFT could beat other excellent DFT calculation libraries, such as FFTW, in performance.

# Reference

[1] J. F. Henriques, R. Caseiro, P. Martins, J. Batista,   
"High-Speed Tracking with Kernelized Correlation Filters", TPAMI 2015.

[2] J. F. Henriques, R. Caseiro, P. Martins, J. Batista,   
"Exploiting the Circulant Structure of Tracking-by-detection with Kernels", ECCV 2012.

[3] H. Kiani Galoogahi, A. Fagg, C. Huang, D. Ramanan, S.Lucey.
"Need for Speed: A Benchmark for Higher Frame Rate Object Tracking", 2017, arXiv preprint arXiv:1703.05884

[4] Y. Wu and J. Lim and M. Yan
"Online Object Tracking: A Benchmark", CVPR 2013

