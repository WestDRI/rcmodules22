+++
title = "Parallel programming in Chapel"
slug = "parallel_chapel"
+++

{{<cor>}}Tuesday, June 28 &nbsp;&&nbsp; Thursday, June 30{{</cor>}}\
{{<cgr>}}9:30am–12:30pm Pacific Time{{</cgr>}}

This course will start at 9:30am Pacific Time and will run until 12:30pm Pacific Time. Its format will be a
combination of several interactive Zoom sessions and the reading materials in-between the Zoom
sessions. Course materials will be added here shortly before the start of the course.

---

This course is a general introduction to the main principles of parallel programming, using Chapel programming
language to illustrate the basic concepts and ideas. Chapel is a relatively new language for both shared- and
distributed-memory programming, with easy-to-use, high-level abstractions for both task and data parallelism
that make it ideal for learning parallel programming for a novice HPC user. Chapel is incredibly intuitive,
striving to merge the ease-of-use of Python and the performance of traditional compiled languages such as C
and Fortran. Parallel constructs that typically take tens of lines of MPI code can be expressed in only a few
lines of Chapel code. Chapel is open source and can run on any Unix-like operating system, with hardware
support from laptops to large HPC systems.

**Instructor**: Alex Razoumov (SFU)

**Prerequisites:** working knowledge of the Linux Bash shell and familiarity with Compute Canada's HPC cluster
  environment, in particular, with the Slurm scheduler (covered in [our HPC course](../basics_hpc)).

**Software**: All attendees will need a remote secure shell (SSH) client installed on their computer in order
to participate in the course exercises. On Windows we recommend
[the free Home Edition of MobaXterm](https://mobaxterm.mobatek.net/download.html). On Mac and Linux computers
SSH is usually pre-installed (try typing `ssh` in a terminal to make sure it is there). No need to install
Chapel on your computer.






<!-- {{<cor>}}Zoom{{</cor>}} {{<s>}} {{<cgr>}}Day 1 - 9:30am-12:30pm Pacific{{</cgr>}} \ -->
<!-- {{<chap>}}Part 1: basic language features{{</chap>}} \ -->
<!-- {{<nolinktitle>}}Introduction to Chapel{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Basic syntax and variables{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Ranges and arrays{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Conditional statements{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Getting started with loops{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Measuring code performance{{</nolinktitle>}} \ -->
<!-- {{<chap>}}Part 2: task parallelism{{</chap>}} \ -->
<!-- {{<nolinktitle>}}Intro to parallel computing{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Fire-and-forget tasks{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Synchronization of threads{{</nolinktitle>}} \ -->
<!-- {{<nolinktitle>}}Task-parallelizing the heat transfer solver{{</nolinktitle>}} -->

{{<cor>}}Zoom{{</cor>}} {{<s>}} {{<cgr>}}Day 1 - 9:30am-12:30pm Pacific{{</cgr>}} \
{{<chap>}}Part 1: basic language features{{</chap>}} \
{{<linktitle url="../chapel/chapel-01-intro" text="Introduction to Chapel">}} \
{{<linktitle url="../chapel/chapel-02-variables" text="Basic syntax and variables">}} \
{{<linktitle url="../chapel/chapel-03-ranges-and-arrays" text="Ranges and arrays">}} \
{{<linktitle url="../chapel/chapel-04-conditions" text="Conditional statements">}} \
{{<linktitle url="../chapel/chapel-05-loops" text="Getting started with loops">}} \
{{<linktitle url="../chapel/chapel-06-command-line-arguments" text="Using command-line arguments">}} \
{{<linktitle url="../chapel/chapel-07-timing" text="Measuring code performance">}} \
{{<chap>}}Part 2: task parallelism{{</chap>}} \
{{<linktitle url="../chapel/chapel-08-intro-parallel" text="Intro to parallel computing">}} \
{{<linktitle url="../chapel/chapel-09-fire-and-forget-tasks" text="Fire-and-forget tasks">}} \
{{<linktitle url="../chapel/chapel-10-synchronising-threads" text="Synchronization of threads">}} \
{{<linktitle url="../chapel/chapel-11-task-parallel-heat-transfer" text="Task-parallelizing the heat transfer solver">}}







{{<cor>}}Zoom{{</cor>}} {{<s>}} {{<cgr>}}Day 2 - 9:30am-12:30pm Pacific{{</cgr>}} \
{{<chap>}}Part 3: data parallelism{{</chap>}} \
{{<nolinktitle>}}Single-locale data parallelism{{</nolinktitle>}} \
{{<nolinktitle>}}Parallelizing the Julia set problem{{</nolinktitle>}} \
{{<nolinktitle>}}Multi-locale Chapel{{</nolinktitle>}} \
{{<nolinktitle>}}Domains and data parallelism{{</nolinktitle>}}

<!-- {{<cor>}}Zoom{{</cor>}} {{<s>}} {{<cgr>}}Day 2 - 9:30am-12:30pm Pacific{{</cgr>}} \ -->
<!-- {{<chap>}}Part 3: data parallelism{{</chap>}} \ -->
<!-- {{<linktitle url="../chapel/chapel-12-single-locale-data-parallel" text="Single-locale data parallelism">}} \ -->
<!-- {{<linktitle url="../chapel/chapel-13-julia-set" text="Parallelizing the Julia set problem">}} \ -->
<!-- {{<linktitle url="../chapel/chapel-14-multi-locale-chapel" text="Multi-locale Chapel">}} \ -->
<!-- {{<linktitle url="../chapel/chapel-15-domains-and-data-parallel" text="Domains and data parallelism">}} -->






### Solutions

You can find the solutions [here](../../solutions-chapel).




### Links

- {{<a "https://chapel-lang.org" "Chapel homepage">}}
- {{<a "https://developer.hpe.com/platform/chapel/home" "What is Chapel?">}} (HPE Developer Portal)
- {{<a "https://chapel-for-python-programmers.readthedocs.io/basics.html" "Getting started guide for Python programmers">}}
- {{<a "https://learnxinyminutes.com/docs/chapel" "Learn X=Chapel in Y minutes">}}
- {{<a "https://stackoverflow.com/questions/tagged/chapel" "Chapel on StackOverflow">}}
- Watch {{<a "https://youtu.be/0DjIdRJIqRY" "Chapel: Productive, Multiresolution Parallel Programming talk">}} by Brad Chamberlain
- WestGrid's April 2019 webinar {{<a "https://bit.ly/39iRSbx" "Working with distributed unstructured data in Chapel">}}
- WestGrid's March 2020 webinar {{<a "https://bit.ly/3QnP1Pd" "Working with data files and external C libraries in Chapel">}} discusses writing arrays to NetCDF and HDF5 files from Chapel

&nbsp;





<!-- * Binary I/O: check https://chapel-lang.org/publications/ParCo-Larrosa.pdf -->

<!-- * advanced: take a simple 2D or 3D non-linear problem, linearize it, implement a parallel multi-locale -->
<!--   linear solver entirely in Chapel -->




{{< figure src="/img/solveMulti.gif" >}}
