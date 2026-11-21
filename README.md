# Response Surface Model + Test Particle Model (RSM+TPM)

Response Surface Model + Test Particle Model (RSM+TPM), supporting both local workstations (Linux/WSL) and High-Performance Computing (HPC) clusters via Singularity/Apptainer containers.


## General Prerequisites

* Ubuntu 22.04
* Python: 3.10 or higher
* Git

## General Setup

You can install the suite into your own WSL/local workstation, in your `$HOME` directory (using a new Python environment is highly recomended); or into your HPC workspace (e.g., `/scratch/$USER`)

The Response Surface Model (RSM) is written in Python and uses these Python packages:

* Numpy
* Matplotlib
* Pandas
* Scikit-learn
* H5py

The suite comes with a basic example [solid_cube.stl](/Inputs/STL_Files/solid_cube.stl), preconfigured in the simulation workflow, so the user can test the installation and simulation work. Please refer the [User Guide](User_Manual.pdf) for additional details.

### Cloning the repository (NOTE: Use the fix/inst_issues_1.0 branch)
```
git clone -b fix/inst_issues_1.0 https://github.com/ASSISTLaboratory/WVU_RSM_Suite.git
cd WVU_RSM_Suite
```
## A. Local installation (Ubuntu)

## B. Singularity container (Ubuntu)

### Prerequisites
* Singularity

During the build, the `tpm` C/C++ executable is compiled inside the container and installed globally to `/usr/local/bin/tpm`.

Choose one of the two definition files depending on your execution environment:

### Option 1: Standard Build — Local PC, WSL, or Single-Node HPC

Recommended for: Laptops, workstations, and single-node HPC jobs (utilizing all available CPU cores within a single server/node).

MPI Backend: Self-contained MPICH. Does not require loading any external HPC environment modules (`module load`).

1. Build the container (from `WVU_RSM_Suite` directory):

* On a PC / WSL (with `sudo`):
```
sudo singularity build Singularity/RSM+TPM_Ubuntu22.04.sif Singularity/RSM+TPM_Ubuntu22.04.def
```
* On an HPC cluster (without `sudo`, using `--fakeroot`):
```
singularity build --fakeroot Singularity/RSM+TPM_Ubuntu22.04.sif Singularity/RSM+TPM_Ubuntu22.04.def
```

2. Enter the container shell:
```
singularity shell Singularity/RSM+TPM_Ubuntu22.04.sif
```
Your terminal prompt will change to `Singularity>`, and your current working directory will remain `WVU_RSM_Suite`.

3. Run the simulation workflow with the **solid_cube example** using your local `Inputs` and saving results to your local `Outputs` directories:
```
./RSM_TPMC/rsm_run_script.py --tpm=/usr/local/bin/tpm
```

### Option 2: Multi-Node HPC Cluster Build

Recommended for: Large distributed HPC simulations spanning multiple compute nodes simultaneously (`#SBATCH --nodes=2` or more) via Slurm (`srun`).

MPI Backend: Requires matching the MPI runtime inside the container with the host cluster's MPI module.

1. Build the container (from `WVU_RSM_Suite` directory):

| MPI runtime | Command |
| :--- | :--- |
| Intel MPI | `singularity build --fakeroot Singularity/RSM+TPM_Ubuntu22.04.sif Singularity/RSM+TPM_Ubuntu22.04.def` |
| OpenMPI 4.1.x | `singularity build --fakeroot Singularity/RSM+TPM_Ubuntu22.04_OpenMPI.sif Singularity/RSM+TPM_Ubuntu22.04_OpenMPI.def` |

2. Load the HPC MPI module :

| MPI runtime | Load the Cluster's module |
| :--- | :--- |
| Intel MPI | e.g., `module load intel-mpi` |
| OpenMPI 4.1.x | e.g., `module load openmpi` |

> **Tip:** Run `module avail mpi` or `module avail openmpi` or ask your HPC administrator to check the exact module name for supporting MPI for your Multi-Node HPC Cluster

3. Enter the container shell:

| MPI runtime | Command |
| :--- | :--- |
| Intel MPI | `singularity shell Singularity/RSM+TPM_Ubuntu22.04.sif` |
| OpenMPI 4.1.x | `singularity shell Singularity/RSM+TPM_Ubuntu22.04_OpenMPI.sif` |

Your terminal prompt will change to `Singularity>`, and your current working directory will remain `WVU_RSM_Suite`.

4. Run the simulation workflow with the **solid_cube example** using your local `Inputs` and saving results to your local `Outputs` directories:
```
./RSM_TPMC/rsm_run_script.py --mpiexec=mpiexec --tpm=/usr/local/bin/tpm
```
