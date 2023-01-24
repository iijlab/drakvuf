# Build instructions for Debian packages

## Build

### Prerequisite

- [Docker](https://www.docker.com/)
- [GNU Make](https://www.gnu.org/software/make/)

Clone the repository:

```
$ cd ${working_directory}
$ git clone https://github.com/iijlab/drakvuf.git
```

Import submodules:

```
$ cd drakvuf
$ git submodule update --init
```

### Build packages

```
$ cd ./package
$ make deb T=ubuntu=20.04-arm64
```

#### Result

```
out/
└── ubuntu
    └── 20.04
        └── arm64
            ├── drakvuf-bundle-1.x-gitxxxxxxxxxxxxxx+xxxxxxx-1-generic-arm64.deb
            └── xen-hypervisor-4.xx.x-generic-arm64.deb
```

#### Help

```
$ make help
Build the Debian binary package with container
--------
$ make <target> [T=ubuntu-20.04-amd64]
$ make deb [T=ubuntu-20.04-amd64]
$ make sh [T=ubuntu-20.04-amd64]

<target>
--------
builder              Create a new builder instance
builder-clean        Remove docker builder instance
builder-run          Launch builder container
cache-clean          Remove docker builder cache
cache                Show disk usage for docker builder cache
clean                Remove the debian packages
deb                  Create the debian packages
distclean            Remove the debian packages and docker image
help                 Display available targets
image                Create docker image
image-allclean       Remove all docker images
image-clean          Remove docker image
realclean            Remove the all debian packages, docker images and builder instance
sh                   Enter interactive mode and start shell in container
--------
```

## Install

Deploy the created package to the target host.

Target host:

```
OS  : Ubuntu 20.04 LTS
Arch: ARM64
```

### Install packages

Drakvuf and Xen:

```
$ sudo apt install \
 ./drakvuf-bundle-*-1-generic-arm64.deb \
 ./xen-hypervisor-*-generic-arm64.deb
```

Required packages:

```
$ sudo apt install libc++1-12 libc++abi1-12
```

See this for details:

[DRAKVUF® Black-box Binary Analysis System](https://drakvuf.com/)
