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
Choose one of the two definition files depending on your execution environment:

## A. Local installation (Ubuntu)

## B. Singularity container (Ubuntu)

### Prerequisites
* Singularity

During the build, the `tpm` C/C++ executable is compiled inside the container and installed globally to `/usr/local/bin/tpm`.

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

MPI Backend: OpenMPI, compatible with HPC OpenMPI modules.

1. Build the container (from `WVU_RSM_Suite` directory):
```
singularity build --fakeroot Singularity/RSM+TPM_Ubuntu22.04_OpenMPI.sif Singularity/RSM+TPM_Ubuntu22.04_OpenMPI.def
```
(Alternatively, build on your local PC with sudo and transfer the .sif file to the cluster via `scp` or `rsync`.)

2. Load the HPC OpenMPI module :
```
module purge
module load parallel/openmpi/4.1.4_gcc93
```
Refer with your HPC administrator which module to load for supporting OpenMPI for your Multi-Node HPC Cluster
* Intel Compilers and Intel MPI: `compiler/2023.1.0 mpi/2021.9.0`
* GCC Compilers 9.3 and OpenMPI 4.1.4: `parallel/openmpi/4.1.4_gcc93`
* GCC Compilers 12.2 and OpenMPI 4.1.5: `parallel/openmpi/4.1.5_gcc122`
* NVIDIA HPC 23.3 and OpenMPI 3.1.5: `lang/nvidia/nvhpc/23.3`

3. Enter the container shell:
```
singularity shell Singularity/RSM+TPM_Ubuntu22.04_OpenMPI.sif
```
Your terminal prompt will change to `Singularity>`, and your current working directory will remain `WVU_RSM_Suite`.

4. Run the simulation workflow with the **solid_cube example** using your local `Inputs` and saving results to your local `Outputs` directories:
```
./RSM_TPMC/rsm_run_script.py --tpm=/usr/local/bin/tpm
```

<!--


You can install these dependencies using the packages on the Linux distribution.
On Ubuntu 22.04.2 LTS (Jammy Jellyfish) you can install most dependencies by executing the command:

```
sudo apt-get install python3 python3-numpy python3-matplotlib python3-pandas python3-sklearn python3-pip
```

An exception here is h5py. The package provided by Ubuntu depends on OpenMPI.
To install the serial version use pip:

```
sudo python3 -m pip install h5py
```

The same packages can be used in the case of Ubuntu 20.04.6 LTS (Focal Fossa)



## Compiling TPM

See the file `RSM_TPMC/tpm/README.md` for instructions on how to compile tpm before using RSM  

## Execute RSM

The main script for RSM is `rsm_run_script.py`.
You can execute this script with two options `--mpiexec` and `--tpm`. These options will select which will be the MPI execution command and the location of the tpm executable.
The values for default are equivalent to using this command:

```
./rsm_run_script.py --mpiexec=mpiexec --tpm=tpm/src/tpm
```

# Running RSM+TPMC on an HPC Cluster

The main development of RSM+TPMC is done on a HPC cluster at West Virginia University.
The cluster is called Thorny Flat.

The folder `ARCH` contains scripts to load the corresponding modules and definition files for conteinarize the code using Singularity.

## Loading Environment Modules

The script `load_modules_Thorny_Flat.sh` load a selection of modules used for the development and testing of the code.
To activate the modules in the current shell session the script must be the source, otherwise, the modules will not be loaded in the current shell.

```
$> source ./ARCH/load_modules_Thorny_Flat.sh 

Missing argument for choosing the compiler to be used.
Select one of the following options:

 * intel22 : Intel Compilers and Intel MPI
 * gcc93   : GCC Compilers 9.3 and OpenMPI 4.1.4
 * gcc122  : GCC Compilers 12.2 and OpenMPI 4.1.5
 * nvhpc   : NVIDIA HPC 23.3 and OpenMPI 3.1.5
```

For example, if you want to use RSM+TPMC with the GCC 12.2 compiler, execute:

```
$> source ./ARCH/load_modules_Thorny_Flat.sh gcc122
```

This will load the following modules

```
Loading gcc version 12.2.0 : lang/gcc/12.2.0
Loading hdf5 version 1.14.0_gcc122 : libs/hdf5/1.14.0_gcc122
Loading python version cpython_3.11.3_gcc122 : lang/python/cpython_3.11.3_gcc122
Loading gcc version 12.2.0 : lang/gcc/12.2.0
Loading openmpi version 4.1.5_gcc122 : parallel/openmpi/4.1.5_gcc122
```

These modules will provide all the libraries, compilers, and Python installed with the necessary packages for running the code.

## Singularity containers

On the folder ARCH there are 3 examples of Singularity definition files used to create containers for RSM+TPM


 * `RSM+TPM_RockyLinux_9.def`   
 * `RSM+TPM_Ubuntu_focal.def`  
 * `RSM+TPM_Ubuntu_jammy.def`

To create the singularity image install Singularity on your Linux machine and as a superuser execute the following command:

```
singularity build RSM+TPM_RockyLinux_9.sif RSM+TPM_RockyLinux_9.def
```
-->
