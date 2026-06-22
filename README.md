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
the base, head (login) node, and worker images. Build metadata is written to
`/opt/hydrotools/versions.env` inside the image.

## Images

Published images use the repository name as their package prefix:

- `ghcr.io/tristanmontoya/vhpc-hydrotools-base:v0.6.3`
- `ghcr.io/tristanmontoya/vhpc-hydrotools-headnode:v0.6.3`
- `ghcr.io/tristanmontoya/vhpc-hydrotools-worker:v0.6.3`

The GitHub Actions workflow publishes `linux/amd64` and `linux/arm64` manifests
for tag builds and manual workflow dispatches.

## HydroLearn HPC Material

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

Before beginning, ensure [Docker Compose](https://docs.docker.com/compose/install/) and [Git](https://git-scm.com/install/) are installed. Docker provides [installation instructions for Docker Compose](https://docs.docker.com/compose/install/), and the Git project provides [Git installation instructions](https://git-scm.com/install/).

### Starting Docker

Make sure Docker is installed and running before starting the cluster.

On macOS, start Docker Desktop with:

```sh
docker desktop start
```

If that command is not available, use:

```sh
open -a Docker
```

On Windows, start Docker Desktop from the Start menu. Then open PowerShell or Windows Terminal for the commands below.

If you are using Docker Engine directly on Linux, start the Docker daemon with:

```sh
sudo systemctl start docker
```

Check that Docker is running:

```sh
docker info
```

### Starting the cluster

From the repository root, start the cluster:

```sh
docker compose up -d
```

The image registry and version have defaults in `docker-compose.yml`. Advanced
users may override them with a local `.env` file or exported shell variables.

### Logging in

Log in to the head node:

```sh
ssh -p 2222 user@localhost
```

The default password is:

```text
password
```

Once logged in, the preinstalled software tools are installed in the following workspace:

```sh
/workspace/hydrolearn-hpc
```

### If SSH reports that the host key has changed

If the cluster has been rebuilt, SSH may reject the connection with an error like:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
Host key verification failed.
```

This can happen when the head-node container has been removed and recreated. Remove the stale SSH host key and reconnect:

```sh
ssh-keygen -R "[localhost]:2222"
ssh -p 2222 user@localhost
```

### Exiting the cluster

To leave the SSH session and return to your host terminal, run:

```sh
exit
```

### Stopping and restarting the cluster

For routine shutdown, stop the cluster without removing the containers:

```sh
docker compose stop
```

Restart the same containers later with:

```sh
docker compose start
```

### Removing the cluster containers

To stop and remove the cluster containers and network:

```sh
docker compose down
```

Note that this does not remove the named Docker volumes, so files in `/workspace`, `/home`, and `/scratch` will persist across cluster rebuilds. You can check this by starting the cluster again:

```sh
docker compose up -d
```

Because `docker compose down` removes containers, the SSH host key may change the next time the cluster is recreated. If that happens, remove the stale SSH key as described above.

To destroy the cluster state completely, including named Docker volumes:

```sh
docker compose down --volumes
```
This will remove all files in the shared workspace and reset the cluster to a clean state. Use this command with caution.

For more complete cluster usage instructions, configuration information,
security notes, and the upstream licence, see the
[eXact lab vHPC main page](https://github.com/exactlab/vhpc).

## Acknowledgements

This fork was developed by Tristan Montoya at the University of Saskatchewan
with funding support from the [Cooperative Institute for Research to Operations
in Hydrology (CIROH)](https://ciroh.ua.edu/).
