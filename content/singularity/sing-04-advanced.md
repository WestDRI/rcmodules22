+++
title = "Running advanced containers"
slug = "sing-04-advanced"
weight = 4
+++

### Running MPI programs from within a container

You need:

- OpenMPI installed inside your Singularity container (version 3 or later)
- high-performance interconnect package (libpsm2 for OmniPath, UCX for Infiniband) inside your container
- to compile your MPI program with the same version of OpenMPI as inside your container

https://docs.alliancecan.ca/wiki/Singularity#Running_MPI_programs_from_within_a_container



module load singularity openmpi  # openmpi does not need to match the MPI version inside the container







### Running GPU programs (Using CUDA)




### Moving from Singularity to Apptainer
