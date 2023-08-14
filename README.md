# Flatcar Container Linux AMD Driver

docker build --build-arg FLATCAR_VERSION=5.15.119 --build-arg AMD_DRIVER_VERSION=latest .

https://wiki.archlinux.org/title/AMDGPU
http://repo.radeon.com
https://amdgpu-install.readthedocs.io/en/latest/install-prereq.html#downloading-the-installer-package

---

# OLD
# Flatcar Container Linux Nvidia Driver

Leveraging NVIDIA GPUs on Container Linux involves the following steps:
* compiling the NVIDIA kernel modules;
* loading the kernel modules on demand;
* creating NVIDIA device files; and
* loading NVIDIA libraries

Compounding this complexity further is the fact that these steps have to be executed whenever the Container Linux system updates since the modules may no longer be compatible with the new kernel.

Forklift takes care of automating all of these steps and ensures that kernel modules are up-to-date for the host's kernel.


## Installation for Systemd

### Requirements
First, make sure you have the [Forklift code available](https://github.com/mediadepot/flatcar-forklift) on your Flatcar Container Linux machine and that the `forklift` service is installed.

### Getting Started
Enable and start the `forklift` template unit file with the desired driver:
```sh
sudo systemctl enable forklift@nvidia
sudo systemctl start forklift@nvidia
```

This service takes care of automatically compiling, installing, backing up, and loading the NVIDIA kernel modules as well as creating the NVIDIA device files.

Compiling the NVIDIA kernel modules can take between 10-15 minutes depending on your Internet speed, CPU, and RAM. To check the progress of the compilation, run:
```sh
journalctl -fu forklift@nvidia
```

## Verify
Once Forklift has successfully run, the host should have NVIDIA device files and kernel modules loaded. To verify that the kernel modules were loaded, run:
```sh
lsmod | grep nvidia
```

This should return something like:
```sh
nvidia_uvm            626688  2
nvidia              12267520  35 nvidia_uvm
...
```

Verify that the devices were created with:
```sh
ls /dev/nvidia*
```

This should produce output like:
```sh
/dev/nvidia-uvm  /dev/nvidia0  /dev/nvidiactl
```

Finally, try running the NVIDIA system monitoring interface (SMI) command, `nvidia-smi`, to check the status of the connected GPU:
```sh
/opt/drivers/nvidia/bin/nvidia-smi
```

If your GPU is connected, this command will return information about the model, temperature, memory usage, GPU utilization etc.

## Leveraging NVIDIA GPUs in Containers
Now that the kernel modules are loaded, devices are present, and libraries have been created, the connected GPU can be utilized in containerized applications.

In order to give the container access to the GPU, the device files must be explicitly loaded in the namespace, and the NVIDIA libraries and binaries must be mounted in the container. Consider the following command, which runs the `nvidia-smi` command inside of a Docker container:
```sh
docker run -it \
--device=/dev/nvidiactl \
--device=/dev/nvidia-uvm \
--device=/dev/nvidia0 \
--volume=/opt/drivers/nvidia:/usr/local/nvidia:ro \
--entrypoint=nvidia-smi \
nvidia/cuda:9.1-devel
```

# References

- https://github.com/squat/modulus/tree/master/nvidia
- https://github.com/kinvolk/coreos-overlay/blob/main/x11-drivers/nvidia-drivers/files/bin/setup-nvidia
- https://raw.githubusercontent.com/kinvolk/manifest/v2605.9.0/main.xml
- https://github.com/BugRoger/coreos-nvidia-driver
- https://docs.flatcar-linux.org/os/kernel-modules/
- https://github.com/paroque28/nvidia-driver/blob/c14c9cec01048936c995e19a705fa967ba1c9a5c/coreos/nvidia-driver





