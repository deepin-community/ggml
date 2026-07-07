# ggml CUDA backend built against the Nvidia-provided repositories ###

This package provides a ggml CUDA backend which has been built against a
specific major.minor version of the CUDA packages from Nvidia's official
repository, rather than the CUDA packages of the target distribution within
Debian.

It can be useful to use Nvidia's repository in order to support recent graphics 
cards or CUDA versions, and also when running a cloud kernel (since the
Debian-provided Nvidia river requires a full-fledged kernel).

What follows are abbreviated instructions for configuring the system to use
these backends. This involves three steps:
  * Enabling the 'contrib' component for your distribution (Debian only)
  * Enabling the Debian maintainers' private repository
  * Enabling Nvidia's repository for CUDA.

## Enabling the 'contrib' component for your distribution

*This step can be skipped on Ubuntu.*

First, locate the sources entry for your distribution. This will be either the
file `/etc/apt/sources.list`, or one of the file ins `/etc/apt/sources.list.d`.

If the file ends in `.list`, then ensure that your distribution has the contrib
section enabled, after main. For example:

```
deb https://deb.debian.org/debian trixie main contrib [possibly other components]
```

If the file ends in `.sources`, modify the `Components` line:

```
Types: deb
URIs: https://deb.debian.org/debian
Suites: trixie
Components: main contrib [possibly other components]
Signed-By: /usr/share/keyrings/debian-archive-keyring.pgp
```

## Enabling the Debian maintainer's private repository

The backend built against Nvidia's official CUDA cannot be shipped from within
the official Debian Archive, so instead, the ggml maintainers ship it in their
PPA.

To use the backend from this PPA, you first need to install the archive key:

```
$ sudo wget -O /usr/share/keyrings/ai-archive-keyring.gpg \
        https://apt.ai.debian.net/debian/ai-archive-keyring.gpg
```

Then, you need to create a sources file for the PPA. Create the file
`/etc/apt/sources.list.d/llama.cpp-dev.sources` with the following contents:

```
Types: deb deb-src
URIs: https://apt.ai.debian.net/llama.cpp-dev/debian
Suites: unstable
Components: vendor-nvidia
Signed-By: /usr/share/keyrings/ai-archive-keyring.gpg
```

On Ubuntu, use the above but replace the URI line with this:

```
URIs: https://apt.ai.debian.net/llama.cpp-dev/ubuntu
```

To use these packages, you must also enable the Nvidia repository, as per the
next step.

## Nvidia repository configuration ##

If not needed, it is recommended to disable the 'non-free' component, in
order to avoid conflicts with Nvidia software provided there by Debian.

Download and install the GPG keys and repository configuration for your
distribution and architecture. Note that Nvidia doesn't use the official
distribution codenames, but rather {OS}{version}, e.g.: debian13 or ubuntu2404.

For example:

```
$ export distro=debian13
$ export arch=x86_64 # or arch=sbsa for arm64
$ wget https://developer.download.nvidia.com/compute/cuda/repos/${distro}/${arch}/cuda-keyring_1.1-1_all.deb
$ sudo dpkg -i cuda-keyring_1.1-1_all.deb
$ sudo apt update
```

The full instructions are provided by Nvidia at:

    https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/debian.html

## Nvidia driver installation ##

Install the Linux kernel headers matching your kernel:

```
$ sudo apt install linux-headers[-cloud]-<arch>
```

In a cloud environment such as AWS, the default is a lightweight "cloud" kernel, 
while on a workstation with a graphical user interface it will be a regular
full-fledged kernel. Run `uname -a` in order to find out.

For example, on an amd64 workstation:
```
$ sudo apt install linux-headers-amd64
```
or, for example on an arm64 cloud instance:
```
$ sudo apt install linux-headers-cloud-arm64
```

It is then recommended by the vendor to pin a driver version, for example:
```
$ sudo apt install nvidia-driver-pinning-595
```

Then install either the compute-only Nvidia driver (typically in a cloud):
```
$ sudo apt install nvidia-driver-cuda nvidia-kernel-dkms
```
or the graphics Nvidia driver (typically on a workstation) which will pull
more packages as dependencies:
```
$ sudo apt install nvidia-driver nvidia-kernel-dkms
```

The installation will take some time as the driver kernel modules
are actually built from source via the dkms mechanism.

Then reboot and check that the driver has been properly installed:
```
$ sudo apt install nvidia-driver-cuda nvidia-kernel-dkms
$ sudo shutdown -r
$ sudo lsmod | grep nvidia
```
