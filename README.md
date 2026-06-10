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
- [mizuRoute](https://github.com/ESCOMP/mizuRoute) `v3.1.1` as `mizuRoute.exe`
- [OSTRICH](https://github.com/DOI-BOR/ostrich) `v21.03.16` as `Ostrich`
  and `OstrichMPI`

They are installed under `/opt/hydrotools/bin`, which is included in `PATH` on
the base, headnode, and worker images. Build metadata is written to
`/opt/hydrotools/versions.env` inside the image.

## Images

Published images use the repository name as their package prefix:

- `ghcr.io/tristanmontoya/vhpc-hydrotools-base:v0.5.0`
- `ghcr.io/tristanmontoya/vhpc-hydrotools-headnode:v0.5.0`
- `ghcr.io/tristanmontoya/vhpc-hydrotools-worker:v0.5.0`

The GitHub Actions workflow publishes `linux/amd64` and `linux/arm64` manifests
for tag builds and manual workflow dispatches.

## HydroLearn HPC assignment workflow

The base image clones
[`tristanmontoya/hydrolearn-hpc`](https://github.com/tristanmontoya/hydrolearn-hpc)
during the image build. The clone source is controlled by the build arguments
`HYDROLEARN_HPC_REPO` and `HYDROLEARN_HPC_REF`, which default to the public
GitHub repository and `main`.

At startup, the headnode seeds the shared workspace volume from the baked image
copy when `/workspace/hydrolearn-hpc` is empty. Runtime dependencies for the
assignment are installed in the image. `packages.yml` is only needed for
optional runtime additions.

The user-facing Python environment is `/opt/venv`, which is built from
Python 3.12 and is first on `PATH`. Both `python` and `python3` resolve to this
environment inside the base, headnode, and worker images. Rocky Linux's system
Python remains available only through its full path for operating system tools.

## Usage

Before beginning, ensure Docker Compose and Git are installed. Docker provides
[installation instructions for Docker Compose][compose-install], and the Git
project provides [Git installation instructions][git-install].

To start the cluster and SSH into the head node:

```sh
git clone https://github.com/tristanmontoya/vhpc-hydrotools.git
cd vhpc-hydrotools
docker compose up -d
ssh -p 2222 user@localhost
```

Use the password `password` when prompted. The head node SSH service is mapped
to `127.0.0.1:2222` by `docker-compose.yml`.

For more complete cluster usage instructions, configuration information,
security notes, and the upstream licence, see the
[eXact lab vHPC main page](https://github.com/exactlab/vhpc).

[compose-install]: https://docs.docker.com/compose/install/
[git-install]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git

## Acknowledgements

This fork was developed by Tristan Montoya at the University of Saskatchewan
with funding support from the [Cooperative Institute for Research to Operations
in Hydrology (CIROH)](https://ciroh.ua.edu/).
