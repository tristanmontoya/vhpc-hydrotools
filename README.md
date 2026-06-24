# Virtual HPC cluster with hydrologic modeling tools

The `vhpc-hydrotools` virtual HPC cluster is a hydrologic modeling fork of
[eXact lab vHPC](https://github.com/exactlab/vhpc). The original project
provides a Docker Compose-based virtual HPC cluster with Slurm, OpenMPI, shared
storage, and SSH access on Rocky Linux 9.

This fork keeps that cluster layout and adds two project-specific changes:

- **Cross-architecture images**: images are published for `linux/amd64` and
  `linux/arm64`, so the same tags can run on x86_64 machines and ARM64 systems
  such as Apple Silicon Macs.
- **Precompiled hydrologic tools**: the base image includes SUMMA, mizuRoute,
  and OSTRICH on `PATH` for use from the login and worker nodes.

The upstream vHPC project remains the source for the Slurm cluster structure.
Use this fork when you want that environment with hydrologic modeling tools
already available in both supported container architectures.

## Included hydrologic tools

The tools are built from pinned upstream releases in `Containerfile.slurm-base`:

- [SUMMA](https://github.com/CH-Earth/summa) `v3.3.0` as `summa.exe`
- [mizuRoute](https://github.com/ESCOMP/mizuRoute) `v1.2.3` as `mizuRoute.exe`
- [OSTRICH](https://github.com/DOI-BOR/ostrich) `v21.03.16` as `Ostrich`
  and `OstrichMPI`

They are installed under `/opt/hydrotools/bin`, which is included in `PATH` on
the base, head (login) node, and worker images. Build metadata is written to
`/opt/hydrotools/versions.env` inside the image.

## Images

Published images use the repository name as their package prefix:

- `ghcr.io/tristanmontoya/vhpc-hydrotools-base:v0.6.5`
- `ghcr.io/tristanmontoya/vhpc-hydrotools-headnode:v0.6.5`
- `ghcr.io/tristanmontoya/vhpc-hydrotools-worker:v0.6.5`

The GitHub Actions workflow publishes `linux/amd64` and `linux/arm64` manifests
for tag builds and manual workflow dispatches.

## HydroLearn HPC Material

The base image clones
[`tristanmontoya/hydrolearn-hpc`](https://github.com/tristanmontoya/hydrolearn-hpc)
during the image build. The clone source is controlled by the build arguments
`HYDROLEARN_HPC_REPO` and `HYDROLEARN_HPC_REF`, which default to the public
GitHub repository and `main`.

At startup, the headnode seeds the shared workspace volume from the baked image
copy when `/workspace/hydrolearn-hpc` is empty. Python runtime dependencies for
the assignment are installed from the baked `hydrolearn-hpc/requirements.txt`.
`packages.yml` is only needed for optional runtime additions.

The user-facing Python environment is `/opt/venv`, which is built from
Python 3.12 and is first on `PATH`. Both `python` and `python3` resolve to this
environment inside the base, headnode, and worker images. Rocky Linux's system
Python remains available only through its full path for operating system tools.

## Usage

### Using Docker

Before beginning, ensure [Docker Compose](https://docs.docker.com/compose/install/) and [Git](https://git-scm.com/install/) are installed following the official installation instructions. To run Docker, follow the instructions for your operating system below.

#### MacOS

Open the terminal and start Docker Desktop from the command line:

```sh
docker desktop start
```
or 
```sh
open -a Docker
```

#### Windows

Start Docker Desktop from the Start menu, then open PowerShell.

#### Linux

Start the Docker daemon:

```sh
sudo systemctl start docker
```
If you want Docker to start automatically when booting your Linux machine, use the following command:

```sh
sudo systemctl enable docker
``` 

### Starting the cluster
Use Git to clone the repository and change into the project directory:

```sh 
git clone https://github.com/tristanmontoya/vhpc-hydrotools.git
cd vhpc-hydrotools
```

Before we set up the cluster, make sure that Docker is running:

```sh
docker info
```

Then, start up the cluster with the following command:

```sh
docker compose up -d
```

### Logging in

Log in to the head node:

```sh
ssh -p 2222 user@localhost
```

Enter the default password (`password`) when prompted. Once logged in, the preinstalled software tools are available in `/workspace/hydrolearn-hpc`. If the cluster has been rebuilt, SSH may reject the connection with an error like:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
Host key verification failed.
```
This can happen when the head node container has been removed and recreated. Remove the stale SSH host key and reconnect:

```sh
ssh-keygen -R "[localhost]:2222"
ssh -p 2222 user@localhost
```

### Exiting, stopping, and restarting the cluster
Exit the cluster by typing `exit` or pressing `Ctrl+D` in the SSH session. The cluster will continue running in the background. To "turn off" the virtual HPC system, simply stop the containers:

```sh
docker compose stop
```

You can then restart the same containers later using `docker compose start`.

### Removing the cluster containers

Use the following command to stop *and remove* the cluster containers and network:

```sh
docker compose down
```

Note that this does not remove the named Docker volumes, so files in `/workspace`, `/home`, and `/scratch` will persist across cluster rebuilds. You can check this by starting the cluster again:

```sh
docker compose up -d
```

Because `docker compose down` removes containers, the SSH host key may change the next time the cluster is recreated. If that happens, remove the stale SSH key as described above. 

### Destroying the cluster state

If you'd like to start fresh and destroy the cluster state completely, including named Docker volumes, use the following command (but make sure you've backed up any important files in the volumes first):

```sh
docker compose down --volumes
```

For more complete cluster usage instructions, configuration information,
security notes, and the upstream licence, see the
[eXact lab vHPC main page](https://github.com/exactlab/vhpc).

## Acknowledgements

This fork was developed by Tristan Montoya at the University of Saskatchewan
with funding support from the [Cooperative Institute for Research to Operations
in Hydrology (CIROH)](https://ciroh.ua.edu/).
