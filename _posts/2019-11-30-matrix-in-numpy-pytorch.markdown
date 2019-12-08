---
layout: post
comments: true
title:  "Matrix operations in Numpy and Pytorch"
excerpt: "Common and useful matrix or tensor operations in Numpy and Pytorch."
date:   2019-11-30 9:00:00
mathjax: true
---

## concatenation vs stack
In short, *concatenation* merges vectors or matrice without introducing new dimensions. At the opposite side, *stack* increases the dimensionality, for example, stacking two vector will create a 2-dimensional matrix, stacking two matrices will make a 3-dimensional tensor.

### Numpy
**Vector concatenation**:
If we want to extend a longer vector by doging \\( c= a + b \\), we can use the `np.concatenate` function. NB: setting `axis=1` is not permitted for a vector which only has a single dimension.
```python
a = np.array([1,2,3])
b = np.array([4,5,6])
c = np.concatenate([a,b], axis=0) # axis must be in [-1,0]

>>> c
array([1, 2, 3, 4, 5, 6])
```
### Pytorch
In pytorch, the API is called `torch.cat` instead of `np.concatenate`, and the dimension is called  `dim` instead of `axis` .

```python
aa = torch.Tensor([1,2,3])
bb = torch.Tensor([4,5,6])
cc = torch.cat([aa,bb], dim=0) # dim must be in [-1,0]

>>> cc
tensor([1., 2., 3., 4., 5., 6.])
```
