+++
title = "Parallel computing in Julia"
slug = "parallel_julia"
+++

{{<cor>}}Tuesday, June 14 &nbsp;&&nbsp; Thursday, June 16{{</cor>}}\
{{<cgr>}}9:30am–12:30pm Pacific Time{{</cgr>}}

This course will start at 9:30am Pacific Time and will run until 12:30pm Pacific Time. Its format will be a combination of
several interactive Zoom sessions and the reading materials in-between the Zoom sessions. Course materials will be added
here shortly before the start of the course.

---

Julia is a high-level programming language well suited for scientific computing and data science. Just-in-time
compilation, among other things, makes Julia really fast yet interactive. For heavy computations, Julia supports
multi-threaded and multi-process parallelism, both natively and via a number of external packages. It also supports
memory arrays distributed across multiple processes either on the same or different nodes. In this hands-on workshop, we
will start with a quick review of Julia's multi-threading features but will focus primarily on Distributed standard
library and its large array of tools. We will demo parallelization using two problems: a slowly converging series and a
Julia set. We will run examples on a multi-core laptop and an HPC cluster.

**Instructors**: Alex Razoumov (SFU) & Marie-Hélène Burle (SFU)

**Prerequisites:** working knowledge of serial Julia (covered in [our Julia course](../programming_julia)) and
familiarity with Compute Canada's HPC cluster environment, in particular, with the Slurm scheduler (covered in
[our HPC course](../basics_hpc)).


**Software**: All attendees will need a remote secure shell (SSH) client installed on their computer in order to
participate in the course exercises. On Windows we recommend
[the free Home Edition of MobaXterm](https://mobaxterm.mobatek.net/download.html). On Mac and Linux computers SSH is
usually pre-installed (try typing `ssh` in a terminal to make sure it is there). No need to install Julia on your
computer.





{{<cor>}}Zoom{{</cor>}} {{<s>}} {{<cgr>}}Day 1 - 9:30am-12:30pm Pacific{{</cgr>}} \
{{<linktitle url="../julia/julia-01-intro-language" text="Introduction to Julia language">}}\
{{<linktitle url="../julia/julia-02-intro-parallel" text="Intro to parallelism">}}\
{{<linktitle url="../julia/julia-03-threads-slow-series" text="Multi-threading with Base.Threads (slow series)">}} \
{{<linktitle url="../julia/julia-04-threadsx-slow-series" text="Multi-threading with ThreadsX (slow series)">}} \
{{<linktitle url="../julia/julia-05-threads-julia-set" text="Parallelizing the Julia set with Base.Threads">}}\
{{<linktitle url="../julia/julia-06-threadsx-julia-set" text="Parallelizing the Julia set with ThreadsX">}}

<!-- {{<cor>}}Zoom{{</cor>}} {{<s>}} {{<cgr>}}Day 1 - 9:30am-12:30pm Pacific{{</cgr>}} \ -->
<!-- {{<nolinktitle>}}Introduction to Julia language{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Intro to parallelism{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Multi-threading with Base.Threads (slow series){{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Multi-threading with ThreadsX (slow series){{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Parallelizing the Julia set with Base.Threads{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Parallelizing the Julia set with ThreadsX{{</nolinktitle>}} -->






<!-- {{<cor>}}Zoom{{</cor>}} {{<s>}} {{<cgr>}}Day 2 - 9:30am-12:30pm Pacific{{</cgr>}} \ -->
<!-- {{<linktitle url="../julia/julia-07-distributed1" text="Distributed.jl: basics">}}\ -->
<!-- {{<linktitle url="../julia/julia-08-distributed2" text="Distributed.jl: three scalable versions of the slow series">}} \ -->
<!-- {{<linktitle url="../julia/julia-09-distributed-arrays" text="DistributedArrays.jl">}}\ -->
<!-- {{<linktitle url="../julia/julia-10-distributed-julia-set" text="Parallelizing the Julia set with DistributedArrays">}}\ -->
<!-- {{<linktitle url="../julia/julia-11-shared-arrays" text="SharedArrays.jl">}}\ -->
<!-- {{<linkoptional url="../julia/julia-12-nbody" text="Parallelizing the N-body problem">}} (supplemental material)\ -->
<!-- {{<linkoptional url="../julia/julia-13-asm" text="Parallelizing the additive Schwarz method">}} (supplemental material) -->

{{<cor>}}Zoom{{</cor>}} {{<s>}} {{<cgr>}}Day 2 - 9:30am-12:30pm Pacific{{</cgr>}} \
{{<nolinktitle>}}Distributed.jl: basics{{</nolinktitle>}} \
{{<nolinktitle>}}Distributed.jl: three scalable versions of the slow series{{</nolinktitle>}} \
{{<nolinktitle>}}DistributedArrays.jl{{</nolinktitle>}} \
{{<nolinktitle>}}Parallelizing the Julia set with DistributedArrays{{</nolinktitle>}} \
{{<nolinktitle>}}SharedArrays.jl{{</nolinktitle>}} \
{{<nolinktitle>}}Parallelizing the N-body problem{{</nolinktitle>}} (supplemental material)\
{{<nolinktitle>}}Parallelizing the additive Schwarz method{{</nolinktitle>}} (supplemental material)




### External links

- ["Julia at Scale"](https://discourse.julialang.org/c/domain/parallel) forum
- Baolai Ge's (SHARCNET) November 2020 webinar
  ["Julia: Parallel computing revisited"](https://youtu.be/xTLFz-5a5Ec)
- WestGrid's March 2021 webinar ["Parallel programming in Julia"](https://youtu.be/2SafLn0xJKY)
- [Julia performance tips](https://docs.julialang.org/en/v1/manual/performance-tips)
- ["Think Julia: How to Think Like a Computer Scientist"](https://benlauwens.github.io/ThinkJulia.jl/latest/book.html)
  by Ben Lauwens and Allen Downey is a good introduction to basic Julia for beginners
