Implementation of slicing operations in NMatrix
--------------------------------------------------

Author: Aleksey Timin

License: Public Domain

###This document is intended for core developers of NMatrix and describes features of slicing implementation which is necessary to take into consideration by developing a new code.


Introduction
=================================================
Currently the NMatrix has two ways to slice. The first way has been implemented by copping all elements of a slice in a new matrix. The new matrix is independent of source matrix and changing one doesn't change other. In Ruby code this way is provided by method `#slice`.

`Ruby
  require "nmatrix"
  require "pp"

  # Create dense matrix with size [3,3] and fill it.
  m = NMatrix.new(:dense, [3,3], (0..8).to_a)

  r = m.slice(0..1, 1..2) #=> [1,2] [4,5]
  r.is_ref?         #=> false

  r[0,0] = 999    

  pp r              #=> [999,2] [4,5]
  pp m              #=> [0,1,2] [3,4,5] [6,7,8]
`

The second way to create a new matrix from a slice which contains only a reference to the source matrix. This method don't allocate new memory and provide access to elements of a source matrix by matrices-reference. When you change elements of such matrix really you change elements of the source matrix. For getting slicing by reference use method `#[]`. 

`Ruby
  require "nmatrix"
  require "pp"

  # Create dense matrix with size [3,3] and fill it.
  m = NMatrix.new(:dense, [3,3], (0..8).to_a) 
  r = m[0..1, 1..2] #=> [1,2] [4,5]
  r.is_ref?         #=> true

  r[0,0] = 999    

  pp r              #=> [999,2] [4,5]
  pp m              #=> [0,999,2] [3,4,5] [6,7,8]
`

Current implementation of slicing mechanism restricts a using algorithms and demands of developers two way of developing for new matrix operations:

1. If we have matrix-reference when duplicate its elements in a new normal matrix and use ordinary algorithm;
2. Using special universal algorithms which work with matricies and matrix-references equally.

Each method have dignities and lacks. It will be considered in the part "Algorithm" in detail.

Concept
=================================================

Algorithms
=================================================

Garbage collector
=================================================

LAPACK
=================================================

