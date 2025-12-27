# 🐡

This repository contains a number of Containerfiles for creating OCI images useful for running OpenBSD applications in a Linux container runtime environment.

## Usage

The way to achieve this is to use virtualization, and using Linux KVM to mitigate performance degradation. To ensure that containers running virtual machines can build and execute without direct user input, [cloud-init](https://github.com/canonical/cloud-init) is used in the installed operating system image.

The images contain files only in a `scratch` container and are not executable on their own.

* `openbsd-amd64` and `openbsd-arm64`: contains a 32GB qcow2 disk image with automatic partitioning scheme in file `/usr/local/share/openbsd/os.qcow2`
* `cloud-init-amd64` and `cloud-init-arm64`: contains an installation set containing cloud-init binaries in `/usr/local/share/site/siteXY.tgz` (see install.site(5))

You will likely want to create a container with QEMU (or similar tool) installed and use for example `COPY --from=ghcr.io/3405691582/1f421/openbsd-arm64 /usr/local/share/openbsd/os.qcow2 ...` in your Containerfile.

For a concrete example, see https://github.com/3405691582/swift-bootstrap.

## Building

cloud-init uses Python with a number of Python dependencies. To ensure the images are built with minimal dependencies, the build uses pyinstaller, which generates single-file archives which do not require additional dependencies to be installed. These do however require specific description of the base system dependencies and may need to be updated as the base operating system does.

pxeboot is used to ensure automated installation on amd64 and require a local httpd to serve the cloud-init site file set during installation. When building, you must ensure that the containerized httpd can bind to port 80. One way to do this is by setting the relevant sysctl on your build host, i.e.,

```
$ sudo sysctl -w net.ipv4.ip_unprivileged_port_start=80
```

netboot does not work fully on arm64. For automated installation, the build uses an existing `openbsd-amd64` image with rdsetroot(8) and vnconfig(8) to extract the installer ramdisk and configure it per autoinstall(8). This means in order to build `openbsd-arm64`, you must build `openbsd-amd64` first (with cloud-init). Additionally, some arm64 bootloaders do not appear to correctly netboot; a specific working version of u-boot needs to be used.

The cloud-init images need only be built when building the OpenBSD images. You should either supply a `siteXY.tgz` set in the local context during the container build process which will be installed into the image, or edit the Containerfile to refer to an existing data-only container image to provide the file set.

You may need to update `install.conf` to match the `siteXY.tgz` file, if it updates.
