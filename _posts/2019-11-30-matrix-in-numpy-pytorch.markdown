---
layout: post
comments: true
title:  "Matrix operations in Numpy and Pytorch"
excerpt: "Common and useful matrix or tensor operations in Numpy and Pytorch."
date:   2019-11-30 9:00:00
mathjax: true
---

## concatenation vs stack
### Numpy
**Vector concatenation**:
For example, if we want to extend a longer vector by doging \\( c= a + b \\), we can use the `np.concatenate` function. NB: setting `axis=1` is not permitted for a vector which only has a single dimension.
```python
a = np.array([1,2,3])
b = np.array([4,5,6])
c = np.concatenate([a,b], axis=0)

>>> c
array([1, 2, 3, 4, 5, 6])
```
