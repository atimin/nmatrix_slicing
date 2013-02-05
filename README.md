Implementation of slicing operations in NMatrix
--------------------------------------------------

Author: Aleksey Timin
License: Public Domain

This document is intended for core developers of NMatrix and describes features of slicing implementation which is necessary to take into consideration by developing a new code.


Introduction
=================================================
Currently the NMatrix has two ways to slice. The first way has been implemented by copping all elements of a slice in a new matrix. The new matrix is independent of source matrix and changing one doesn't change other. In Ruby code this way is provided by method `#slice`.
The second way to create a new matrix from a slice which contains only a reference to the source matrix. This method don't allocate new memory and provide access to elements of a source matrix by matrices-reference. When ...

Concept
=================================================

Algorithms
=================================================

Garbage collector
=================================================

LAPACK
=================================================

