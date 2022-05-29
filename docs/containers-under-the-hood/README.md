# Containers under the hood

- [Containers under the hood](#containers-under-the-hood)
  - [Namespaces = isolation](#namespaces--isolation)
    - [PID](#pid)
    - [Mount](#mount)
    - [Network](#network)
    - [User](#user)
  - [Container isolation](#container-isolation)
  - [Links](#links)

![Comparison between Kernel space and User space](/img/4.png "Comparison between Kernel space and User space")

![Comparison between containers and virtual machines](/img/5.png "Comparison between containers and virtual machines")

## Namespaces = isolation

### PID

- isolates processes from each other
- one process can't see others
- ptocess ID 10 can exist multiple times, once in every namespace

### Mount

- restrict access to mounts or root filesystem

### Network

- only access to certain network devices
- firewall & routing rules & socket port numbers
- not able to see all traffic or contact all endpoints

### User

- different set of user ids used
- user(0) inside one namespace can be different from user (0) inside another
- don't use the host-root user(0) inside the container

## Container isolation

![Container isolation](/img/6.png "Container isolation")

## Links

- [What have containers done for you lately?](https://www.youtube.com/watch?v=MHv6cWjvQjM)
