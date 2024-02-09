# 1: Containers from 30,000 feet

## The bad old days

- We could only run one application per server because the open-systems world of Windows and Linux didn't have the technologies to run multiple applications on the same server back in the days.
- When we need a new application, we need a new server.
- The performance requirement of the new application is unknown, IT department need to take a guess and ended up choosing a big fast servers that cost a lot of money, which is overpowered. Or, a underpowered servers that couldn't handle transactions and ended up losing customers.
- This is a waste of company capital and environmental resources.

## Hello VMware!

- VMware was born, a technology (virtual machine) that allowed us to run multiple business application on a single server safely.
- No more to oversized or underpowered server!
- VMs are great but they're far from perfect...
- VM requires its own OS (operating system) is a major flaw.
- Every OS consumes CPU, RAM and other resoursces.
- Every OS needs patching and monitoring.
- In some cases, OS requires a license.
- VM are too slow to boot, and portability isn't great - migrating and moving VM workloads between hypervisors and cloud platforms is harder than it needs to be.

> Hypervisor. A software that you can use to run multiple VMs on a single physical machine.

## Hello Containers!

- Container is roughly analogous to the VM.
- The major difference between them is that containers do not require their own full-blown OS.
- Containers on a single host share the host's OS. No need huge amounts of system resources such as CPU, RAM and storage.
- No licensing costs, and overhead of OS patching and other maintenance.
- Fast to start and ultra-portable.
- Despite all of this, containers remained complex and outside of reach of most organizations. It wasn't until Docker came along that containers were effectively democratized and accessizble to the masses.

## Windows containers vs Linux containers

- Container shares the kernel of the host it's running on. This means containerized Windows apps need a host with a Windows kernel, whereas containerized Linux apps need a host with a Linux kernel.
- It's possible to run Linux containers on Windows machines with the WSL 2 backend installed.

> There no such thing as Mac containers. But you can run Linux containers on your Mac using Docker Desktop.
