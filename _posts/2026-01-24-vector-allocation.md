---
layout: post
title: "Allocating 2D Vectors in C++"
categories: code
---


This entire post comes from my love-hate relationship with C++. I love Stepanov's idea for C++ and so, C++11, while obviously flawed, is my favorite version of the language. Today, I just want to play around with the eternal `std::vector`, and understand the real tradeoffs with `std::array`. Mind you, I am not going to play with C-style arrays for obvious reasons. In particular, I want to initialise a 2D `std::vector`, preferably of size $1000 \times 1000$.

# Step 0. Setup

I'm using [Google Benchmark](https://github.com/google/benchmark) to create micro-benchmarks for this post, and am using Clang v17.0.0. All code is available at : [MathTest.cpp](https://github.com/PrasannaMaddila/math-test-cpp/tree/main). All of these numbers are on my local machine, so your mileage will definitely vary.

I'm also setting global constants for the array sizes. These are the size of the matrices I want to create.

```c++
const int row_size = 1000;
const int col_size = 1000;
```

# Step 1. Nested Vectors

The simplest way to create a 2D vector is to create a vector of vectors. For example, we create a `vector<vector<double>>` object here called `vector_nres`, and use a `vector<double>` to store the row as we initialise it. This row is then pushed back onto `vector_nres`, which means a copy is created and pushed back. We then clear the row object (which resets the elements, but keeps the memory), and do this all over again.

```c++
std::vector< std::vector<double> > vector_nres;
std::vector<double> row(row_size);

for (int i = 0; i < row_size; i++) {
    row.clear() ; // reset the elements
    for (int j = 0; j < col_size; j++){
        row.push_back(1.0);
    }
    vector_nres.push_back(row);
}
```

We can try to do a little better by adding a `reserve` call to pre-allocate some memory. This is because `std::vector` is a dynamically growing list (allocated on the heap, we come back to this in a minute). This means that when we push back an element without the space for it, 
- A new buffer of twice the size (usually) is allocated somewhere in memory
- the old contents of the vector are copied over 
- and the new one is inserted.
This has what we call in the business ["amortized constant"](https://en.cppreference.com/w/cpp/container/vector/push_back) complexity. This means that on average, it's free to add an element, and once in a while, we pay a big bill to reallocate. But since we do this infrequently, we can imagine that the big bill is paid off over all the push-backs we do, like an EMI :)

Theory aside, we didn't do too badly.

```bash
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
Vector_NoReserve                                   2.02 ms         2.02 ms          342
Vector_Reserve                                     2.05 ms         2.04 ms          439
```


# Step 2. My Neighbour Uses Arrays

An knee-jerk reaction to have to the previous code is to use `std::array`.

```c++
std::array< std::array<double, col_size>, row_size> arr;

for (int i = 0; i < row_size; i++) {
  for (int j = 0; j < col_size; j++)
    arr[i][j] = 1.0; 
}
```

On the other hand, we can try the infamous 1D-buffer-is-2D-buffer trick. So, we allocate one huge buffer of doubles, with size `row_size * col_size` and index it outselves.

```c++
std::array< double, row_size * col_size > arr_1d;

for (int i = 0; i < row_size; i++) {
  for (int j = 0; j < col_size; j++)
    arr_1d[i*col_size + j] = 1.0; 
}
```

This gives us a nearly 10x performance boost ! Note that we can consider the distance between their times basically noise. The compiler writers have put in a lot of work to make that happen, I'll bet.

```bash
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
Array                                             0.220 ms        0.220 ms         3182
Array_1D                                          0.221 ms        0.221 ms         3145
```

So, we have some catching up to do ! On the other hand, we can't use arrays willy-nilly here. The difference is that arrays are statically allocated objects on the stack, not the heap. That means that their size must be known at compile time. Alternatively stated, we cannot use arrays that are bigger than the program stack :( - for example, setting row and column sizes to $5000$ gives me the classic `Segmentation fault: core dumped` error that should be intimate knowledge to all of us here. 

# Step 3. My Vector is an Array

I'm not pretending. Other than the stack/heap difference, vectors are contiguous, just like arrays are. So, we can try the infamous 1D-buffer-is-2D-buffer trick one more time, only this time, we're using vectors.

```bash
std::vector< double > vector_1d(row_size * col_size);

for (int i = 0; i < row_size; i++) {
  for (int j = 0; j < col_size; j++)
    vector_1d[i * col_size + j] = 1.0;
}
```

On the other hand, there's also the wonderful `.at` function for C++ vectors that performs some bounds checking before allowing us to modify or see anything. So, that gives us

```bash
std::vector< double > vector_1d(row_size * col_size);

for (int i = 0; i < row_size; i++) {
  for (int j = 0; j < col_size; j++)
    vector_1d.at(i * col_size + j) = 1.0;
}
```

That puts us right back at array speeds, if not a little better. Since our memory access in this example is best friends with the cache and the branch predictor, I suspect we get some sort of bonus.

```bash
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
Vector_1D_ReserveIndexing                         0.202 ms        0.202 ms         3447
Vector_1D_ReserveIndexing_BoundsCheck             0.137 ms        0.137 ms         5166
```

The last thing I tried was noticing that we always calculate `i * col_size` repeatedly in the hot-loop. If the compiler doesn't optimise that away, then we might have one last low-hanging fruit to reap.

```c++
std::vector< double > vector_1d(row_size * col_size);

for (int i = 0; i < row_size; i++) {
  int offset = i * col_size;
  for (int j = 0; j < col_size; j++)
    vector_1d[offset + j] = 1.0;
}
```

This pushes our code to execute even faster, to >5000 iterations! 

```bash
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
Vector_1D_ReserveIndexing_Offset                  0.202 ms        0.202 ms         3461
Vector_1D_ReserveIndexing_Offset_BoundsCheck      0.135 ms        0.135 ms         5207
Array_1D_Offset                                   0.221 ms        0.221 ms         3163
```

# Step 4. Use the STL !

Since we're already here, we can use nice things™️ like `std::fill`, to leverage what the STL has already done for us. This should be as clear as we can get with our intentions, which should (hopefully) allow the compiler to optimise even more. 

```c++
std::vector< double > vector_1d(row_size * col_size);
std::fill(vector_1d.begin(), vector_1d.end(), 1.0);
```

And does it deliver ! A marginal improvement over our handwritten champion (1d vector + pre-reserve + offset variable + bounds-checked accesses), and a much cleaner implementation too - 1 single line.

```bash
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
Vector_1D_Fill                                    0.132 ms        0.132 ms         5322
```

# Conclusions 

Overall, I learned that 

- CMake basics + Google Benchmark for the software side.
- nested vectors are bad, 1d buffers are where it's at
- the `.at()` might actually help, since we're clearer about our intentions with the vector. This is also the same with use of the STL.
- Arrays are allocated on the stack, so their speed benefits are limited by their size.
- The STL is actually quite nice sometimes :)

**Huge caveat** This is only for initialising a 2d vector. I say nothing about operations on these objects, since I haven't measured anything on that yet.

## Full Table of Benchmarks

````bash
$ ./bin/allocation_test --benchmark_unit_time=ms
---------------------------------------------------------------------------------------
Benchmark                                             Time             CPU   Iterations
---------------------------------------------------------------------------------------
Vector_NoReserve                                   2.02 ms         2.02 ms          342
Vector_Reserve                                     2.05 ms         2.04 ms          439
Vector_1D_NoReserve                                5.59 ms         5.59 ms          147
Vector_1D_Reserve                                  5.31 ms         5.31 ms          155
Vector_1D_ReserveIndexing                         0.202 ms        0.202 ms         3447
Vector_1D_ReserveIndexing_Offset                  0.202 ms        0.202 ms         3461
Vector_1D_ReserveIndexing_BoundsCheck             0.137 ms        0.137 ms         5166
Vector_1D_ReserveIndexing_Offset_BoundsCheck      0.135 ms        0.135 ms         5207
Vector_1D_STL                                     0.538 ms        0.538 ms         1296
Vector_1D_Fill                                    0.132 ms        0.132 ms         5322
Array                                             0.220 ms        0.220 ms         3182
Array_1D                                          0.221 ms        0.221 ms         3145
Array_1D_Offset                                   0.221 ms        0.221 ms         3163
```
