+++
title = "Domains and data parallelism"
slug = "chapel-15-domains-and-data-parallel"
weight = 15
katex = true
+++

<!-- ## Multi-locale data parallelism -->

## Local domains

We start this section by recalling the definition of a **range** in Chapel. A range is a 1D set of
integer indices that can be bounded or infinite:

```chpl
var oneToTen: range = 1..10; // 1, 2, 3, ..., 10
var a = 1234, b = 5678;
var aToB: range = a..b;      // using variables
var twoToTenByTwo: range(stridable=true) = 2..10 by 2; // 2, 4, 6, 8, 10
var oneToInf = 1.. ;         // unbounded range
```

On the other hand, **domains** are multi-dimensional (including 1D) sets of integer indices that are
always bounded. To stress the difference between domain ranges and domains, domain definitions always
enclose their indices in curly brackets. Ranges can be used to define a specific dimension of a domain:

```chpl
var domain1to10: domain(1) = {1..10};                // 1D domain from 1 to 10 defined using the range 1..10
var twoDimensions: domain(2) = {-2..2, 0..2};        // 2D domain over a product of two ranges
var thirdDim: range = 1..16;                         // a range
var threeDims: domain(3) = {1..10, 5..10, thirdDim}; // 3D domain over a product of three ranges
for idx in twoDimensions do                          // cycle through all points in a 2D domain
  write(idx, ', ');
writeln();
for (x,y) in twoDimensions {                         // can also cycle using explicit tuples (x,y)
  write(x,",",y,"  ");
}
writeln();
```

Let us define an n^2 domain called `mesh`. It is defined by the single task in our code and is therefore
defined in memory on the same node (locale 0) where this task is running. For each of n^2 mesh points,
let us print out

1. m.locale.id = the ID of the locale holding that mesh point (should be 0)
1. here.id = the ID of the locale on which the code is running (should be 0)
1. here.maxTaskPar = the number of available cores (max parallelism with 1 task/core) (should be 3)

**Note**: We already saw some of these variables/functions: numLocales, Locales, here.id, here.name,
here.numPUs(), here.physicalMemory(), here.maxTaskPar.

```chpl
config const n = 8;
const mesh: domain(2) = {1..n, 1..n}; // a 2D domain defined in shared memory on a single locale
forall m in mesh do                   // go in parallel through all n^2 mesh points
  writeln(m, ' ', m.locale.id, ' ', here.id, ' ', here.maxTaskPar);
```
```
((7, 1), 0, 0, 3)
((1, 1), 0, 0, 3)
((7, 2), 0, 0, 3)
((1, 2), 0, 0, 3)
...
((6, 6), 0, 0, 3)
((6, 7), 0, 0, 3)
((6, 8), 0, 0, 3)
```

Now we are going to learn two very important properties of Chapel domains.

**First**, domains can be used to **define arrays of variables of any type on top of them**. For example, let
us define an n^2 array of real numbers on top of `mesh`:

```chpl
config const n = 8;
const mesh: domain(2) = {1..n, 1..n}; // a 2D domain defined in shared memory on a single locale
var T: [mesh] real;                   // a 2D array of reals defined in shared memory on a single locale (mapped onto this domain)
forall t in T do                      // go in parallel through all n^2 elements of T
  writeln(t, ' ', t.locale.id);
```
```sh
$ chpl test.chpl -o test
$ sbatch distributed.sh
$ cat solution.out
```
```
(0.0, 0)
(0.0, 0)
(0.0, 0)
(0.0, 0)
...
(0.0, 0)
(0.0, 0)
(0.0, 0)
```

By default, all n^2 array elements are set to zero, and all of them are defined on the same locale as the
underlying mesh. We can also cycle through all indices of T by accessing its domain:

```chpl
forall idx in T.domain
  writeln(idx, ' ', T(idx));   // idx is a tuple (i,j); also print the corresponding array element
```
```
(7, 1) 0.0
(1, 1) 0.0
(7, 2) 0.0
(1, 2) 0.0
...
(6, 6) 0.0
(6, 7) 0.0
(6, 8) 0.0
```

Since we use a paralell `forall` loop, the print statements appear in a random runtime order.

We can also define multiple arrays on the same domain:

```chpl
const grid = {1..100};          // 1D domain
const alpha = 5;                // some number
var A, B, C: [grid] real;       // local real-type arrays on this 1D domain
B = 2; C = 3;
forall (a,b,c) in zip(A,B,C) do // parallel loop
  a = b + alpha*c;              // simple example of data parallelism on a single locale
writeln(A);
```

The **second important property** of Chapel domains is that they can **span multiple locales** (nodes).

## Distributed domains

Domains are fundamental Chapel concept for distributed-memory data parallelism.

Let us now define an n^2 distributed (over several locales) domain `distributedMesh` mapped to locales in
blocks. On top of this domain we define a 2D block-distributed array A of strings mapped to locales in
exactly the same pattern as the underlying domain. Let us print out

(1) a.locale.id = the ID of the locale holding the element a of A
(2) here.name = the name of the locale on which the code is running
(3) here.maxTaskPar = the number of cores on the locale on which the code is running

Instead of printing these values to the screen, we will store this output inside each element of A as a string:
a = int + string + int
is a shortcut for
a = int:string + string + int:string

```
use BlockDist; // use standard block distribution module to partition the domain into blocks
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};
const distributedMesh: domain(2) dmapped Block(boundingBox=mesh) = mesh;
var A: [distributedMesh] string; // block-distributed array mapped to locales
forall a in A { // go in parallel through all n^2 elements in A
  // assign each array element on the locale that stores that index/element
  a = a.locale.id:string + '-' + here.name[0..4] + '-' + here.maxTaskPar:string + '  ';
}
writeln(A);
```

The syntax `boundingBox=mesh` tells the compiler that the outer edge of our decomposition coincides
exactly with the outer edge of our domain. Alternatively, the outer decomposition layer could include an
additional perimeter of *ghost points* if we specify

```chpl
const mesh: domain(2) = {1..n, 1..n};
const largerMesh: domain(2) dmapped Block(boundingBox=mesh) = {0..n+1,0..n+1};
```

but let us not worry about this for now.

Running our code on 3 locales, with 2 cores per locale, produces the following output:

```txt
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2
2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2
```

> If we were to run it on 4 locales, with 2 cores per locale, we might see something like this:
> ```txt
> 0-node1-2   0-node1-2   0-node1-2   0-node1-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
> 0-node1-2   0-node1-2   0-node1-2   0-node1-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
> 0-node1-2   0-node1-2   0-node1-2   0-node1-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
> 0-node1-2   0-node1-2   0-node1-2   0-node1-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
> 2-node4-2   2-node4-2   2-node4-2   2-node4-2   3-node3-2   3-node3-2   3-node3-2   3-node3-2
> 2-node4-2   2-node4-2   2-node4-2   2-node4-2   3-node3-2   3-node3-2   3-node3-2   3-node3-2
> 2-node4-2   2-node4-2   2-node4-2   2-node4-2   3-node3-2   3-node3-2   3-node3-2   3-node3-2
> 2-node4-2   2-node4-2   2-node4-2   2-node4-2   3-node3-2   3-node3-2   3-node3-2   3-node3-2
> ```

As we see, the domain `distributedMesh` (along with the string array `A` on top of it) was decomposed into 3x1
blocks stored on the three nodes, respectively. Equally important, for each element `a` of the array, the line
of code filling in that element ran on the same locale where that element was stored. In other words, this
code ran in parallel (`forall` loop) on 3 nodes, using up to two cores on each node to fill in the
corresponding array elements. Once the parallel loop is finished, the `writeln` command runs on locale 0
gathering remote elements from the other locales and printing them to standard output.

Now we can print the range of indices for each sub-domain by adding the following to our code:

```chpl
for loc in Locales do
  on loc do
    writeln(A.localSubdomain());
```

On 3 locales we should get:

```
{1..3, 1..8}
{4..6, 1..8}
{7..8, 1..8}
```

Let us count the number of threads by adding the following to our code:

```chpl
var count = 0;
forall a in A with (+ reduce count) { // go in parallel through all n^2 elements
  count = 1;
}
writeln("actual number of threads = ", count);
```

If our `n=8` is sufficiently large, there are enough array elements per node ($8*8/3\approx 21$ in our case)
to fully utilize the two available cores on each node, so our output should be

```sh
$ chpl test.chpl -o test
$ sbatch distributed.sh
$ cat solution.out
```
```
actual number of threads = 6
```

> ### <font style="color:blue">Exercise "Data.2"</font>
> Try reducing the array size `n` to see if that changes the output (fewer threads per locale), e.g.,
> setting n=3. Also try increasing the array size to n=20 and study the output. Does the output make sense?

So far we looked at the block distribution `BlockDist`. It will distribute a 2D domain among nodes either
using 1D or 2D decomposition (in our example it was 2D decomposition 2x2), depending on the domain size
and the number of nodes.

Let us take a look at another standard module for domain partitioning onto locales, called
CyclicDist. For each element of the array we will print out again

(1) a.locale.id = the ID of the locale holding the element a of A
(2) here.name = the name of the locale on which the code is running
(3) here.maxTaskPar = the number of cores on the locale on which the code is running

```chpl
use CyclicDist; // elements are sent to locales in a round-robin pattern
config const n = 8;
const mesh: domain(2) = {1..n, 1..n};  // a 2D domain defined in shared memory on a single locale
const m2: domain(2) dmapped Cyclic(startIdx=mesh.low) = mesh; // mesh.low is the first index (1,1)
var A2: [m2] string;
forall a in A2 {
  a = a.locale.id:string + '-' + here.name[1..5]:string + '-' + here.maxTaskPar:string + '  ';
}
writeln(A2);
```
```sh
$ chpl -o test test.chpl
$ sbatch distributed.sh
$ cat solution.out
```
```txt
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2   2-node3-2
0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2   0-node1-2
1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2   1-node2-2
```

> With 4 locales, we might see something like this:
> ```txt
> 0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2__
> 2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2__
> 0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2__
> 2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2__
> 0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2__
> 2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2__
> 0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2   0-node1-2   1-node4-2__
> 2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2   2-node2-2   3-node3-2__
> ```

As the name `CyclicDist` suggests, the domain was mapped to locales in a cyclic, round-robin pattern. We
can also print the range of indices for each sub-domain by adding the following to our code:

```chpl
for loc in Locales do
  on loc do
	writeln(A2.localSubdomain());
```
```txt
{1..8 by 3, 1..8}
{1..8 by 3 align 2, 1..8}
{1..8 by 3 align 0, 1..8}
```

In addition to BlockDist and CyclicDist, Chapel has several other predefined distributions: BlockCycDist,
ReplicatedDist, DimensionalDist2D, ReplicatedDim, BlockCycDim -- for details please see
https://chapel-lang.org/docs/primers/distributions.html.

## Heat transfer solver on distributed domains

Now let us use distributed domains to write a parallel version of our original heat transfer solver
code. We'll start by copying `baseSolver.chpl` into `parallel.chpl` and making the following
modifications to the latter:

(1) Add

```chpl
use BlockDist;
const mesh: domain(2) = {1..rows, 1..cols};   // local 2D domain
```

(2) Add a larger (n+2)^2 block-distributed domain `largerMesh` with a layer of *ghost points* on
*perimeter locales*, and define a temperature array T on top of it, by adding the following to our code:

```chpl
const largerMesh: domain(2) dmapped Block(boundingBox=mesh) = {0..rows+1, 0..cols+1};
```

(3) Change the definitions of T and Tnew (delete those two lines) to

```chpl
var T, Tnew: [largerMesh] real;   // block-distributed arrays of temperatures
```

<!-- Here we initialized an initial Gaussian temperature peak in the middle of the mesh. As we evolve our -->
<!-- solution in time, this peak should diffuse slowly over the rest of the domain. -->

<!-- > ## Question -->
<!-- > Why do we have   -->
<!-- > forall (i,j) in T.domain[1..n,1..n] {   -->
<!-- > and not   -->
<!-- > forall (i,j) in mesh -->
<!-- >> ## Answer -->
<!-- >> The first one will run on multiple locales in parallel, whereas the -->
<!-- >> second will run in parallel via multiple threads on locale 0 only, since -->
<!-- >> "mesh" is defined on locale 0. -->

<!-- The code above will print the initial temperature distribution: -->

<!-- ``` -->
<!-- 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0   -->
<!-- 0.0 2.36954e-17 2.79367e-13 1.44716e-10 3.29371e-09 3.29371e-09 1.44716e-10 2.79367e-13 2.36954e-17 0.0   -->
<!-- 0.0 2.79367e-13 3.29371e-09 1.70619e-06 3.88326e-05 3.88326e-05 1.70619e-06 3.29371e-09 2.79367e-13 0.0   -->
<!-- 0.0 1.44716e-10 1.70619e-06 0.000883826 0.0201158 0.0201158 0.000883826 1.70619e-06 1.44716e-10 0.0   -->
<!-- 0.0 3.29371e-09 3.88326e-05 0.0201158 0.457833 0.457833 0.0201158 3.88326e-05 3.29371e-09 0.0   -->
<!-- 0.0 3.29371e-09 3.88326e-05 0.0201158 0.457833 0.457833 0.0201158 3.88326e-05 3.29371e-09 0.0   -->
<!-- 0.0 1.44716e-10 1.70619e-06 0.000883826 0.0201158 0.0201158 0.000883826 1.70619e-06 1.44716e-10 0.0   -->
<!-- 0.0 2.79367e-13 3.29371e-09 1.70619e-06 3.88326e-05 3.88326e-05 1.70619e-06 3.29371e-09 2.79367e-13 0.0   -->
<!-- 0.0 2.36954e-17 2.79367e-13 1.44716e-10 3.29371e-09 3.29371e-09 1.44716e-10 2.79367e-13 2.36954e-17 0.0   -->
<!-- 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0   -->
<!-- ``` -->

Let us define an array of strings `message` with the same distribution over locales as T, by adding the
following to our code:

```chpl
var message: [largerMesh] string;
forall m in message do
  m = here.id:string;   // store ID of the locale on which the code is running
writeln(message);
halt();
```
```sh
$ chpl -o parallel parallel.chpl
$ ./parallel -nl 3 --rows=8 --cols=8   # run this from inside distributed.sh
```

The outer perimeter in the partition below are the *ghost points*, with the inner 8x8 array:

```txt
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 0 0 0
1 1 1 1 1 1 1 1 1 1
1 1 1 1 1 1 1 1 1 1
1 1 1 1 1 1 1 1 1 1
2 2 2 2 2 2 2 2 2 2
2 2 2 2 2 2 2 2 2 2
2 2 2 2 2 2 2 2 2 2
```

> With 4 locales, we might see something like this:
> ```txt
> 0 0 0 0 0 1 1 1 1 1
> 0 0 0 0 0 1 1 1 1 1
> 0 0 0 0 0 1 1 1 1 1
> 0 0 0 0 0 1 1 1 1 1
> 0 0 0 0 0 1 1 1 1 1
> 2 2 2 2 2 3 3 3 3 3
> 2 2 2 2 2 3 3 3 3 3
> 2 2 2 2 2 3 3 3 3 3
> 2 2 2 2 2 3 3 3 3 3
> 2 2 2 2 2 3 3 3 3 3
> ```

> ### <font style="color:blue">Exercise "Data.3"</font>
> In addition to here.id, also print the ID of the locale holding that value. Is it the same or different
> from `here.id`?

(4) Let's comment out this `message` part, and start working on the parallel solver.

(5) Move the linearly increasing boundary conditions (right/bottom sides) before the `while` loop.

(6) Replace the loop for computing *inner* `Tnew`:

```chpl
for i in 1..rows do {  // do smth for row i
  for j in 1..cols do {   // do smth for row i and column j
	Tnew[i,j] = 0.25 * (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]);
  }
}
```

with a parallel `forall` loop (**contains a mistake on purpose!**):

```chpl
forall (i,j) in mesh do
  Tnew[i,j] = 0.25 * (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]);
```

> ### <font style="color:blue">Exercise "Data.4"</font>
> Can anyone spot a mistake in this loop?

(7) Replace

```chpl
delta = 0;
for i in 1..rows do {
  for j in 1..cols do {
	tmp = abs(Tnew[i,j]-T[i,j]);
	if tmp > delta then delta = tmp;
  }
}
```

with

```chpl
delta = max reduce abs(Tnew[1..rows,1..cols]-T[1..rows,1..cols]);
```

(8) Replace

```chpl
T = Tnew;
```

with the **inner-only** update

```chpl
T[1..rows,1..cols] = Tnew[1..rows,1..cols];   // uses parallel `forall` underneath
```

## Benchmarking

Let's compile both serial and data-parallel versions using the same multi-locale compiler (and we will
need `-nl` flag when running both):

```sh
$ which chpl
/project/60303/shared/c3/chapel-1.24.1/bin/linux64-x86_64/chpl
$ chpl --fast baseSolver.chpl -o baseSolver
$ chpl --fast parallel.chpl -o parallel
```

First, let's try this on a smaller problem. Let's write two job submission scripts:

```sh
#!/bin/bash
# this is baseSolver.sh
#SBATCH --time=0:5:0         # walltime in d-hh:mm or hh:mm:ss format
#SBATCH --mem-per-cpu=1000   # in MB
#SBATCH --output=baseSolver.out
./baseSolver -nl 1 --rows=30 --cols=30 --niter=2000
```

```sh
#!/bin/bash
# this is parallel.sh
#SBATCH --time=0:5:0         # walltime in d-hh:mm or hh:mm:ss format
#SBATCH --mem-per-cpu=1000   # in MB
#SBATCH --nodes=3
#SBATCH --cpus-per-task=2
#SBATCH --output=parallel.out
./parallel -nl 3 --rows=30 --cols=30 --niter=2000
```

Let's run them both:

```sh
$ sbatch baseSolver.sh
$ sbatch parallel.sh
```

Wait for the jobs to finish and then check the results:

```sh
$ tail -3 baseSolver.out
Final temperature at the desired position [1,30] after 1148 iterations is: 2.58084
The largest temperature difference was 9.9534e-05
The simulation took 0.008524 seconds

$ tail -3 parallel.out
Final temperature at the desired position [1,30] after 1148 iterations is: 2.58084
The largest temperature difference was 9.9534e-05
The simulation took 193.279 seconds
```

As you can see, on the training VM cluster the parallel code on 4 nodes (with 2 cores each) ran ~22,675
times slower than a serial code on a single node ... What is going on here!? Shouldn't the parallel code
run ~8X faster, since we have 8X as many processors?

This is a **_fine-grained_** parallel code that needs lots of communication between tasks, and relatively
little computing. So, we are seeing the **communication overhead**. The training cluster has a very slow
network, so the problem is exponentially worse there ...

If we increase the problem size, there will be more computation (scaling O(n^2)) in between
communications (scaling O(n)), and at some point parallel code should catch up to the serial code and
eventually run faster. Let's try these problem sizes:

```
--rows=650 --cols=650 --niter=9500 --tolerance=0.002
Final temperature at the desired position [1,650] after 7750 iterations is: 0.125606
The largest temperature difference was 0.00199985

--rows=2000 --cols=2000 --niter=9500 --tolerance=0.002
Final temperature at the desired position [1,2000] after 9140 iterations is: 0.04301
The largest temperature difference was 0.00199989

--rows=8000 --cols=8000 --niter=9800 --tolerance=0.002
Final temperature at the desired position [1,8000] after 9708 iterations is: 0.0131638
The largest temperature difference was 0.00199974

./baseSolver -nl 1 --rows=16000 --cols=16000 --niter=9900 --tolerance=0.002
Final temperature at the desired position [1,16000] after 9806 iterations is: 0.00818861
The largest temperature difference was 0.00199975
```

#### On the training cluster

I switched both codes to single precision, to be able to accommodate larger arrays. The table below shows the
**slowdown** factor when going from serial to parallel. For each row correspondingly, I was running the
following:

```sh
$ ./baseSolver --rows=2000 --niter=200 --tolerance=0.002
$ ./parallel -nl 4 --rows=2000 --niter=200 --tolerance=0.002
$ ./parallel -nl 6 --rows=2000 --niter=200 --tolerance=0.002
```

| | 30^2 | 650^2 | 2,000^2 | 16,000^2 |
| ----- | ----- | ----- | ----- | ----- |
| --nodes=4 --cpus-per-task=2 | 32,324 | 176 | 27.78 | 4.13 |
| --nodes=6 --cpus-per-task=16 | | | 15.3 | 1/5.7 |

#### On Graham (faster interconnect):

| | 30^2 | 650^2 | 2,000^2 | 8,000^2 |
| ----- | ----- | ----- | ----- | ----- |
| --nodes=4 --cpus-per-task=2 | 5,170 | 14 | 2.9 | 1.25 |
| --nodes=4 --cpus-per-task=4 | | | | 1/1.56 |
| --nodes=8 --cpus-per-task=4 | | | | 1/2.72 |

<!-- 16,000^2 on Graham: baseSolver 41,482s; parallel --nodes=4 --cpus-per-task=2 61,052s -->

<!-- on Cedar at 650^2 we have ~60X slowdown: 27.5408 seconds and 1658.34 seconds, respectively (bad Chapel build over OmniPath?) -->

<!-- with Chapel 1.17.0 on Graham 650^2 took 21.1198 seconds and 907.352 seconds, respectively -->

<!-- on Cedar 2000^2 baseSolver: The simulation took 469.298 seconds -->

<!-- on Graham 5000^2 took 3697.65 seconds and 6015.98 seconds, respectively -->

## Final parallel code

Here is the final version of the entire code, minus the comments:

```chpl
use Time, BlockDist;
config const rows = 100, cols = 100;
config const niter = 500;
config const iout = 1, jout = cols, nout = 20;
config const tolerance = 1e-4: real;
var count = 0: int;
const mesh: domain(2) = {1..rows, 1..cols};
const largerMesh: domain(2) dmapped Block(boundingBox=mesh) = {0..rows+1, 0..cols+1};
var delta: real;
var T, Tnew: [largerMesh] real;   // a block-distributed array of temperatures
T[1..rows,1..cols] = 25;   // the initial temperature
writeln('Working with a matrix ', rows, 'x', cols, ' to ', niter, ' iterations or dT below ', tolerance);
for i in 1..rows do T[i,cols+1] = 80.0*i/rows;   // right-side boundary
for j in 1..cols do T[rows+1,j] = 80.0*j/cols;   // bottom-side boundary
writeln('Temperature at iteration ', 0, ': ', T[iout,jout]);
delta = tolerance*10;   // some safe initial large value
var watch: Timer;
watch.start();
while (count < niter && delta >= tolerance) do {
  count += 1;
  forall (i,j) in largerMesh[1..rows,1..cols] do
	Tnew[i,j] = 0.25 * (T[i-1,j] + T[i+1,j] + T[i,j-1] + T[i,j+1]);
  delta = max reduce abs(Tnew[1..rows,1..cols]-T[1..rows,1..cols]);
  T[1..rows,1..cols] = Tnew[1..rows,1..cols];
  if count%nout == 0 then writeln('Temperature at iteration ', count, ': ', T[iout,jout]);
 }
watch.stop();
writeln('Final temperature at the desired position [', iout,',', jout, '] after ', count, ' iterations is: ', T[iout,jout]);
writeln('The largest temperature difference was ', delta);
writeln('The simulation took ', watch.elapsed(), ' seconds');
```

This is the entire multi-locale, data-parallel, hybrid shared-/distributed-memory solver!

> ### <font style="color:blue">Exercise "Data.5"</font>
> Add printout to the code to show the total energy on the inner mesh [1..row,1..cols] at each
> iteration. Consider the temperature sum over all mesh points to be the total energy of the system. Is
> the total energy on the mesh conserved?

> ### <font style="color:blue">Exercise "Data.6"</font>
> Write a code to print how the finite-difference stencil [i,j], [i-1,j], [i+1,j], [i,j-1], [i,j+1] is
> distributed among nodes, and compare that to the ID of the node where T[i,i] is computed. Use problem
> size 8x8.

This produced the following output clearly showing the *ghost points* and the stencil distribution for
each mesh point:

```
empty empty empty empty empty empty empty empty empty empty
empty 000000   000000   000000   000001   111101   111111   111111   111111   empty
empty 000000   000000   000000   000001   111101   111111   111111   111111   empty
empty 000000   000000   000000   000001   111101   111111   111111   111111   empty
empty 000200   000200   000200   000201   111301   111311   111311   111311   empty
empty 220222   220222   220222   220223   331323   331333   331333   331333   empty
empty 222222   222222   222222   222223   333323   333333   333333   333333   empty
empty 222222   222222   222222   222223   333323   333333   333333   333333   empty
empty 222222   222222   222222   222223   333323   333333   333333   333333   empty
empty empty empty empty empty empty empty empty empty empty
```

* note that Tnew[i,j] is always computed on the same node where that element is stored
* note remote stencil points at the block boundaries

<!-- ## Periodic boundary conditions -->
<!-- Now let us modify the previous parallel solver to include periodic BCs. At the beginning of each time -->
<!-- step we need to set elements on the *ghost points* to their respective values on the *opposite ends*, by -->
<!-- adding the following to our code: -->
<!-- ``` -->
<!--   T[0,1..n] = T[n,1..n]; // periodic boundaries on all four sides; these will run via parallel forall -->
<!--   T[n+1,1..n] = T[1,1..n]; -->
<!--   T[1..n,0] = T[1..n,n]; -->
<!--   T[1..n,n+1] = T[1..n,1]; -->
<!-- ``` -->
<!-- Now total energy should be conserved, as nothing leaves the domain. -->

## I/O

Let us write the final solution to disk. Please note:

- here we'll write in ASCII (raw binary output is slightly more difficult to make portable) <!-- Chapel can also write
  binary data but nothing can read it (checked: not the endians problem!) -->
- a much better choice would be writing in NetCDF or HDF5 -- covered in our webinar
["Working with data files and external C libraries in Chapel"](https://westgrid.github.io/trainingMaterials/programming#working-with-data-files-and-external-c-libraries-in-chapel)
  - portable binary encoding (little vs. big endian byte order)
  - compression
  - random access
  - parallel I/O (partially implemented) -- see the HDF5 example in the webinar

Let's comment out all lines with `message` and `assert()`, and add the following at the end of our code to write ASCII:

```chpl
use IO;
var myFile = open('output.dat', iomode.cw);   // open the file for writing
var myWritingChannel = myFile.writer();   // create a writing channel starting at file offset 0
myWritingChannel.write(T);   // write the array
myWritingChannel.close();   // close the channel
```
```sh
$ chpl --fast parallel.chpl -o parallel
$ ./parallel -nl 3 --rows=8 --cols=8   # run this from inside distributed.sh
$ ls -l *dat
-rw-rw-r-- 1 razoumov razoumov 659 Mar  9 18:04 output.dat
```

The file *output.dat* should contain the 8x8 temperature array after convergence.

### Other I/O topics

* for binary I/O check https://chapel-lang.org/publications/ParCo-Larrosa.pdf
* writing arrays to NetCDF and HDF5 files from Chapel is covered in our
  [March 2020 webinar](https://bit.ly/3QnP1Pd)
<!-- * advanced take-home exercise: take a simple 2D or 3D non-linear problem, linearize it, implement a parallel -->
<!--   multi-locale linear solver entirely in Chapel -->
