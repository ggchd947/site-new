+++
title = 'Best Language for ML Inference'
date = 2024-10-09T08:14:44-07:00
+++
### @ac_crypto
There's been a lot of talk about "optimised" inference libraries written in rust (e.g. candle) or zig (e.g. zml) recently.

The argument goes that they will be faster for production inference workloads and as a result win in the long term over Python libs like PyTorch.

I don't buy this. Here's why:

The bottleneck with inference is not userspace code. It's the stuff running on the accelerators. Your userspace code does need to be written well to ensure that you're  for example managing memory efficiently however the thing that really matters is your kernels.

This is why I'm really excited about @__tinygrad__. It's written in 100% Python and solves the combinatorial problem you have with libraries like PyTorch where you end up with thousands of handwritten kernels that have to be maintained. Instead, tinygrad requires implementing a small amount of code for each accelerators and generates kernels, using search (currently Beam, moving to MCTS, hoping they move to entropy-based methods eventually) to find the best kernels. tinygrad strikes a nice tradeoff here, staying within the incredible Python ML ecosystem while focusing on the part that is slow: kernels.

As much as possible (there are obvious exceptions like iOS) I'm looking to keep @exolabs in 100% Python too. We've had many contributors from the wider Python ML ecosystem who were able to easily jump into the codebase and contribute. Also, there are just so many great inference libraries available in Python. Currently exo supports MLX (h/t @awnihannun) and tinygrad (h/t @realGeorgeHotz) but support for PyTorch, JAX and llama.cpp are coming. Different libraries run fast on different devices so exo can leverage the best library for the device it's running on without needing to do the heavy lifting of writing or generating optimised kernels (the libs take care of that).

When a new model comes out, exo can add support almost instantly since it's built on top of a thriving Python ML ecosystem. Any algorithmic innovations can also be integrated quickly without much effort.

Personally I love Go and rust so writing everything in Go or rust would be a dream, but the reality is that Python is the obvious choice here. It's cool to see alternative libraries in other languages but I do believe Python will win ML inference.
