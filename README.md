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

## Usage

For cluster usage, configuration, security notes, and the upstream licence, see
the [eXact lab vHPC main page](https://github.com/exactlab/vhpc).

## Acknowledgements

This fork was developed by Tristan Montoya at the University of Saskatchewan
with funding support from the [Cooperative Institute for Research to Operations
in Hydrology (CIROH)](https://ciroh.ua.edu/).
