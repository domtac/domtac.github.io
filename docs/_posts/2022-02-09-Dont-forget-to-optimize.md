---
layout: post
title:  "Don't forget to optimize"
date:   2022-02-09 13:55:38 +0200
categories: rust performance python go
---

## Comparing performance

Out of curiosity and while it came up a lot recently I wanted to do a performance comparison between RUST and other languages. On a little bit of googling I did find some other posts doing various comparisons before me. Like [here](https://marshalshi.medium.com/performance-comparison-rust-vs-pyo3-vs-python-6480709be8d) or [here](https://benchmarksgame-team.pages.debian.net/benchmarksgame/fastest/rust-go.html).  What I noticed was that often the comparison was done with unoptimized rust, e.g. compiled in debug mode.
On the other hand I wanted to have something to tinker around and try different algorithms in rust, go and python.

> Edit:
> Someone send me a link to this interessting [paper](https://greenlab.di.uminho.pt/wp-content/uploads/2017/10/sleFinal.pdf) exploring execution time and energy consumption of different programming languages

So I created an unsorted collection to try out some sorting algorithms. The algorithms used here are differnet apporaches to sorting random numbers. 
So far I have done:
- [Bubblesort](https://en.wikipedia.org/wiki/Bubble_sort)
- [Insertions sort](https://en.wikipedia.org/wiki/Insertion_sort)
- [Quick sort](https://en.wikipedia.org/wiki/Quicksort)
- [Parallel quicksort](https://en.wikipedia.org/wiki/Quicksort#Parallelization)

I won't go into detail on how these algorithms work, but have linked each bullet point to its repsective wikipedia article, which does a great deal explaining the basic concepts.
The [code](https://github.com/domtac/performance_comparison) is badly written, but I think might suffice to emphazise the concept here.


## Results

The benchmarks have been manually run on my computer and go without any scientific claim. 
But nonetheless I got reproducible results. The times measured are the average values over 10 consecutive runs. 

The sorting algorithms have been run on randomized data sets of the sizes 10k and 100k. For python and unoptimized rust I skipped bubblesort for 100k data sets, mainly because my time on earth is limited.


Runs on MacBookPro M1 (2020), times are in milliseconds.

| Algo | number of values | python | rust | rust (release) | go |    
|----|-----|----|-----|----|----|    
| Bubble | 10000 | 5133 | 7442 | 90 | 105 |    
| Bubble | 100000 | - | - | 18231 | 11000 |    
| Insertion | 10000 | 1400 | 1700 | 37 | 54 |    
| Quicksort | 10000 | 14.5 | 12.9 |  0.48 | 0.6 |    
| Quicksort | 100000 | 180 | 140 |  6 | 9 |    


### Bonus: Parallel quick sort

As the manufacturers of my  laptop where so kind to provide me with several computation cores, I was willing to try a parallel quicksort to speed up the sorting even more. Another reason is that I wanted to try out the [rayon](https://crates.io/crates/rayon/1.2.1) crate.

The interesting effect here is, that rayon really allowed parallelism without significant overhead, whereas the parallel inplementation in python had little effect.

| Algo | number of values | python | rust | rust (release) | go (GOMAXPROCS=8) |
|----|-----|----|-----|----|----|
| Parallel quick sort | 100000 | 150 | 25 | 1.7 | 2 |


## Conclusion

My key take away is to never forget to `--release` my rust code ever again, this alone speeds up immensly.
For future performance optimizations I will take into account, that it seems that changing the programming language does have more effect than optimizing your algorithms. Even when you take into account, that you might have way more complicated algorithms than some sorting, I think it is worth thinking about changing from python to rust, at least partially ([PyO3](https://github.com/PyO3/pyo3)).

