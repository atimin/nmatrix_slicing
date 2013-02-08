Implementation of slicing operations in NMatrix
--------------------------------------------------

Author: Aleksey Timin

License: Public Domain

This document is intended for core developers of NMatrix and describes features of slicing implementation which is necessary to take into consideration by developing a new code. 

All examples for the dense type of matrix.


Introduction
=================================================
Currently the NMatrix has two ways to slice. The first way has been implemented by copping all elements of a slice in a new matrix. The new matrix is independent of source matrix and changing one doesn't change other. In Ruby code this way is provided by method `#slice`.

```Ruby
  require "nmatrix"
  require "pp"

  # Create dense matrix with size [3,3] and fill it.
  m = NMatrix.new(:dense, [3,3], (0..8).to_a)

  r = m.slice(0..1, 1..2) #=> [1,2] [4,5]
  r.is_ref?         #=> false

  r[0,0] = 999    

  pp r              #=> [999,2] [4,5]
  pp m              #=> [0,1,2] [3,4,5] [6,7,8]
```

The second way to create a new matrix from a slice which contains only a reference to the source matrix. This method don't allocate new memory and provide access to elements of a source matrix by matrices-reference. When you change elements of such matrix really you change elements of the source matrix. For getting slicing by reference use method `#[]`. 

```Ruby
  require "nmatrix"
  require "pp"

  # Create dense matrix with size [3,3] and fill it.
  m = NMatrix.new(:dense, [3,3], (0..8).to_a) 
  r = m[0..1, 1..2] #=> [1,2] [4,5]
  r.is_ref?         #=> true

  r[0,0] = 999    

  pp r              #=> [999,2] [4,5]
  pp m              #=> [0,999,2] [3,4,5] [6,7,8]
```

Current implementation of slicing mechanism restricts a using algorithms and demands of developers two way of developing for new matrix operations:

1. If we have matrix-reference when duplicate its elements in a new normal matrix structure and use ordinary algorithm;
2. Using special universal algorithms which works with matrices and matrix-references equally.

Each method has dignities and lacks. It will be considered in the part "Algorithm" in detail. 

Concept
=================================================

For implementation slicing by reference the base storage structures have been changed:

1. Added information about offsets of the reference's begin relatively the begin source matrix;
2. Added the recursive pointer **src** which references to a storage structure here is or references to source matrix;
3. Added the count of references for GC. See part Garbage Collector

```C
  struct  DENSE_STORAGE {
    size_t  dim;                     
    size_t* shape;                   
    size_t* offset;               // !!!
    int			count;                // !!!
    STORAGE*		src;              // !!!
    void* elements;
  }
```

**NOTE:** If matrix isn't reference when the field offset consists zeroes and the field **src** storages pointer to its structure.


By **src** we can access to elements of matrix without knowledges of its type.

```C
  DENSE_STORAGE *s = fict_slice_func(src_matrix, ....);
  s->src->elements;
```

But we cannot use they simple as a continuous sequence when we have a matrix-reference. For example:

We have a source dense matrix:

|1|2|3|
|4|5|6|
|7|8|9|

Its elements storage in the one-dim array 1,2,3,4,5,6,7,8,9

When we have sliced by reference the matrix from point [1,1] with size [2,2] we could get the matrix:

|5|6|
|8|9|

But `s->src->elements` still reference to the source matrix and have the array 1,2,3,4,5,6,7,8,9. We must access to elements using information about offsets relatively the source matrix.

Algorithms
=================================================

Garbage collector
=================================================

LAPACK
=================================================

