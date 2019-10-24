# Troubleshooting Containers
## David Raila
## NCSA Software Meeting
## Mon October 3, 2016

# Overview
- Containers and container runtimes - How they work
- Challenges in troubleshooting
- Approaches to troubleshooting
- Details
-- In-container, sidecar, host, remote
- Troubleshooting at scale in cluster orchestration systems
- Resources

# Optimal Containers (12-factor)
- Encapsulated software service with minimal dependencies
- Single purpose - does one thing
- Immutable - code inside is static
- Stateless - data/state stored externally
- Configuration-free - configuration delivered at run-time
- Versioned - updates generate new versions
- Fungible - swappable for equivalent implementation
-- livestock vs pets

# What is a container?

# JUST A PROCESS

# A container is just a process living on the kernel with all the others
- Setup by the runtime in a particular way
- Using regular kernel facilities
-- Namespaces - a private view of the kernel
-- Control Groups - fine grained resource controls
-- Capabilities - fine-grained privileges

# Namespaces - isolated kernel view
```
cgroup -> 'cgroup:[4026531835]'
ipc -> 'ipc:[4026531839]'
mnt -> 'mnt:[4026531840]'
net -> 'net:[4026531963]'
pid -> 'pid:[4026531836]'
user -> 'user:[4026531837]'
uts -> 'uts:[4026531838]'
```

# Namespace behaviors
- Default - inherit from parent
- Creation - process clone() args
- Runtimes create per container 
- Command-line tools: nsenter, unshare

# Control Groups - resource management
- Metering, and limiting resources
- Memory, I/O, network, devices
- Some access control - devices
- Soft/hard limits 
- Managed under /sys/fs/cgroup

# Capabilities - Fine grained privileges
- Network, signals, mounts, etc.
- Enables root with limits
- Container engine defaults
-- reduce unsafe root capabilities
-- cgroups, mount, devices

# What a container engine does (simplified)
- Setup filesystem copy - (mkdir, tar -x)
- Create namespace, cgroup, net-interfaces - (unshare, ip link)
- Setup mounts, filesystem, /proc, cgroups - (mount, umount)
- Setup security - (selinux, apparmor)
- Drop capabilities - (getcap, setcap)
- 'jump' into configured container - (chroot)

# Why so many container runtimes?
- docker, rkt, lxc/lxd, runC, OCID
-- All use the same building blocks
-- namespaces, cgroups, capabilities, filesystem
-- All perform the same
- Differences are in the ecosystems, tooling, etc.
-- More/less support for building/distributing images

# DIY Containers - learning the hard way
- Not tremendously difficult ~ 25 lines of shellscript
-- nsenter, unshare, tar, mount -o bind, umount, chroot.
- It is almost guaranteed to be insecure
-- But you'll learn exactly what a container runtime does
-- Don't blame me if you start a fire
-- I just offered the matches, extinguisher is your job

# Troubleshooting Options from inside-out
- In-container
-- Jump into container, setup/install
- Sidecar container
--  Deploy container with tooling in same namespace
- From container host
-- Setup tooling in host and work
- Remote debugging
-- Setup tooling remotely, work over network

# Challenges with In-container Troubleshooting
- No in-container tooling
-- Should(may) not even have OS or package installation tools
- Troubleshooting may exceed resource constraints
-- Internal tooling in same cgroups

# Challenges with sidecar co-containers
- Need non-default namespace setup
-- Join some target namespaces, but not all
-- Typically IPC, PID, Network
- Need to build tooling container
-- 

# Issues with Host OS Troubleshooting
- Troubleshooting using OS tools from host-system
-- Many tools not container-aware
-- Container optimized OS's lack tooling
-- Network ports and volumes are typically fixed

# Troubleshooting from debug containers
- Put tools in debug container
- Start debug container in target's namespace
-- Not target cgroup


# Container Do's
- Log to stdout
- Use a volume for configuration
-- inotify can respond to changes without restart
- Only include essential dependencies
-- package autoremove build tools
- Use debug flag in configuration
-- Turn on tracing to stdout
- Use static executables
- Add tiny web API for control/debug

# Container Dont's
- Don't install at run-time
-- package installs, npm install, etc.
- Don't ship tools and/or source
- 







