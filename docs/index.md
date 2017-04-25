# NOTICE
We have changed our project to [Parallel Hidden Markov Model ](https://github.com/joaofaro/KCFcpp)

# C++ Parallel KCF Tracker
Team members:
- Yuchen Huo (<yhuo1@andrew.cmu.edu>)
- Danhao Guo (<danhaog@andrew.cmu.edu>)

# Summary
We are going to extend the the Kernalized Correlation Filter (KCF) [1, 2], a fast oject tracker, to the multicore platform.

# Background
Visual Object Tracking is one of the popular tasks in the computer vision area. There are a lot of different implementations focusing on how to improve the accuracy of the tracking algorithm. These implementations mostly target at the "real time" online systems, which have been aimed at approximate frame rate of 30 Frames Per Second(FPS), enough for previous devices and workloads. However, for higher frame rate cameras (240 FPS) or an offline video processing pipeline, 30 FPS still seems to be slow. We look through some of the benchmark results [3, 4] and find this Kernalized Correlation Filter which provide top speed on CPU and pretty good accuracy. We decide to further extend this algorithm to be able to benefit from multicore systems and be fast enough to be used offline.

By creating the circulant training data matrix, KCF uses Discrete Fourier Transform (DFT) to compute the close form solution of Ridge Regression and reduce both the storage and computation. It further uses HOG feature instead of raw pixels to gain better mean precision on the testing dataset. The remaining main computation lies in the dft and HOG feature extraction. There are plenty of fast parallel algorithms for dft and HOG feature extraction so we decide to build the parallel oject tracker based on the KCF implementation. 

Fast Fourier Transforms (FFT) is an efficient way for calculation of DFT by utilizing DFT's symmetry properties. Compared with O(N^2) computation complexity, the computational cost of FFT is only O(N log(N)) It is a key tool in most of digital signal processing systems including object tracking ones. In the KCF object tracking algorithm, FFT is heavily used for learning and evaluation to achieve lower computational complexity. For example, by utilizing FFT operation, KRLS learning algorithm is improved from O(N^4) to only O(N^2 log(N)) for NxN 2D images.

# Challenge
The biggest challenge is that we are not very familiar with the computer vision algorithm so it may take time for us to understand the implementation details of the KCF tracker. However, since we have the open source implementation, we may not need to understand every single lines in the code. Instead, we may identify the paralizable parts and do optimization on these parts.

Although FFT brings out great improvement on performance, it is still computational intensive with O(n log(n)) complexity. According to the properties of FFT algorithm itself, we believe that there should be some computational independent tasks inside it such as the DFT calculation of even terms and odd terms in each iteration. So, it may benefit from parallel implementation on multi-threaded processing. To implement FFT in parallel, the major points is that multi-dimensional FFT is utilized. In this regard, analyzing computation independence becomes tough. Meanwhile, applying such complicated algorithm with parallel technologies especially AVX instructions are pretty hard. We plan to combine both AVX and OpenMP to support different level of parallelism for FFT.

For HOG feature extraction, there have already been a few multicore CPU and GPU implementations for different computer vision tasks. We would be able to borrow some the ideas from them but their optimizations are often combined with the specific vision task algorithm, so we may need to develop our own version and some further optimization might be adopted to offer better performance for the KCF tracker.

KCF has done a lot of optimization to reduce the computation which makes it the fastest cpu object tracker implementation, we are not sure that the remaining computation would still be enough to gain benefits from multithreading. This might be a problem. We would try to identify this as soon as possible.

# Resources
The Author of KCF open source their [code](https://github.com/joaofaro/KCFcpp) on the github so we are able to build our parallel version based on their implementation. OpenCV is an open source library for computer vision. It provides tons of functions for real-time computer vision. The [code](https://github.com/opencv/opencv) is available on GitHub. we could build our parallel FFT implementation based on the top of OpenCV's source code. We would develop and test our software on the GHC machines which have 8 cores and we may further test the performance on Xeon Phi machines or develop the GPU version to see how our implementation scales if everything goes smoothly.

# Evaluation and Goals
Since KCF tracker currently is a sequential implementation, we hope our parallel version could achieve close to linear speedup comparing to the sequential version. The basic performance measure would just be the speedup/ core number graph. However, as we haven't identified the dependency in the current implementation, we are not very sure if this is possible to achieve the linear speedup.

We may try to use other libraries like FFTW to compare the performance to see if we have achieved a good enough parallel implementation. Opencv also have implemented the same object tracker, which we may use to perform the comparison.

Minimum Goal: 1. ReImplement FFT algorithm to support multicore and SIMD parallelism. The parallel FFT should be faster than the original FFT implementation in KCF and improve the overall performance of KCF algorithm on multicore machines. 2. Implement multithread and SIMD version of the HOG feature computation, along with parallel FFT, to improve the performance of KCF algorithm.

Expected Goal: 1. The parallel FFT could achieve sub-linear scalability with the increase of core number in the machine. 2. Perform enough experiment to make parallel HOG feature processing fast enough to reach some other implementation's speedup.

Stretch Goal: The parallel FFT could beat other excellent DFT calculation libraries, such as FFTW, in performance.

# Reference
[1] J. F. Henriques, R. Caseiro, P. Martins, J. Batista,   
"High-Speed Tracking with Kernelized Correlation Filters", TPAMI 2015.

[2] J. F. Henriques, R. Caseiro, P. Martins, J. Batista,   
"Exploiting the Circulant Structure of Tracking-by-detection with Kernels", ECCV 2012.

[3] H. Kiani Galoogahi, A. Fagg, C. Huang, D. Ramanan, S.Lucey,   
"Need for Speed: A Benchmark for Higher Frame Rate Object Tracking", 2017, arXiv preprint arXiv:1703.05884.

[4] Y. Wu and J. Lim and M. Yan,   
"Online Object Tracking: A Benchmark", CVPR 2013

# Schedule
Week 1 (Apr. 3rd): 

Deciding project idea; 
Investigation background (Papers and Code); 
Writing proposal. 

Week 2 (Apr. 10th): 

Implementing parallel FFT with OpenMP;
Implementing parallel HOG features extraction with OpenMP.

Week 3 (Apr. 17th): 

Incremental developing parallel FFT with AVX;
Incremental developing parallel HOG features extraction with AVX.
Integrating two implementations into KCF algorithm;

Week 4 (Apr. 24th):

Conducting experiments to evaluate ParaKCF on both performance improvement and scalability;
Tuning the implementation to improve the performance.

Week 5 (May 1st): 

Cleaning code;
Perfecting documentation;
Writing final report;
Preparing for the final competition.
