---
title: "Cs231n Labnotes1"
date: 2024-03-10T15:48:40+08:00
tags:
- cs231n
draft: false
---
> I'm recently working on *[cs231n](http://cs231n.stanford.edu/)*. This is the *[github link](https://github.com/CaoeUU/CS231n-assignment-2023)* and I'll update the labnotes simultaneously here.
> 
<!--more-->

## k-NN
> The whole process of working on this assignment was quite frustrating and time-consuming. But finally it's done!! Here's some notes about it.
### The implementation of the distance function
The calculation using *one loop* and *no loop* presented to be challenging to me. It's primarily because I am not familar with the ***broadcasting*** in numpy. Click here to orient you to the *[official document](https://numpy.org/doc/stable/user/basics.broadcasting.html)*.
- **Broadcasting**
  
  An automatic adjustment when numpy dealing with two arrays that have seemingly unmatched dimensions. 
  
  Below are two useful and legible pictures taken from the official website, which granted me great assitance to understand how it works.

  ![Operation between matrix and vector](/images/broadcasting-pic1.png)

  ![Operation between two vectors](/images/broadcasting-pic2.png)

  Something worth notice in the second pic is that when doing this computation, you are expected to transfer the left one from a row vector into a column vector. A little trick to realize it could be using `vector_a[:, np.newaxis]`.

  Actually, before acquiring knowledge about broadcasting, I thought about using matrix transformations to get the vector into the shape I wanted, but quickly I realized it might be impossible or quite complicated, so I gave up.

### Other experiences accumulated
1. Initially, I forgot to make square root for the distance. Then I got an incrediblely large deviation from the validated computation designed before hand. So I might be more meticulous next time......
2. When defining variables, be careful to avoid repeating some names previously even though you might think it's okay to overwrite previous variable because they have similar functions. Or you might be dragged to craziness like me and don't know what's wrong with the programme... (sincere advice!!)
