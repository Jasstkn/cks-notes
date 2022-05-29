# cks-notes

- [cks-notes](#cks-notes)
  - [Kubernetes secure architecture](#kubernetes-secure-architecture)
  - [Containers under the hood](#containers-under-the-hood)
    - [Namespaces = isolation](#namespaces--isolation)
      - [PID](#pid)
      - [Mount](#mount)
      - [Network](#network)
      - [User](#user)
    - [Container isolation](#container-isolation)
    - [Links-2](#links-2)
  - [Network policies](#network-policies)
    - [Examples](#examples)
    - [Multiple NetworkPolicies](#multiple-networkpolicies)

## [Kubernetes secure architecture](docs/k8s-secure-architecture/README.md)

## Containers under the hood

![Comparison between Kernel space and User space](/img/4.png "Comparison between Kernel space and User space")

![Comparison between containers and virtual machines](/img/5.png "Comparison between containers and virtual machines")

### Namespaces = isolation

#### PID

- isolates processes from each other
- one process can't see others
- ptocess ID 10 can exist multiple times, once in every namespace

#### Mount

- restrict access to mounts or root filesystem

#### Network

- only access to certain network devices
- firewall & routing rules & socket port numbers
- not able to see all traffic or contact all endpoints

#### User

- different set of user ids used
- user(0) inside one namespace can be different from user (0) inside another
- don't use the host-root user(0) inside the container

### Container isolation

![Container isolation](/img/6.png "Container isolation")

### Links-2

- [What have containers done for you lately?](https://www.youtube.com/watch?v=MHv6cWjvQjM)

## Network policies

- Firewall rules in Kubernetes
- Implemented by the Nwtwork plugins CNI (Calico / Weave / etc)
- Namespace level
- Restrict the ingress and/or Egress for a group of Pods based on certain rules and conditions

Without NetworkPolicies:

- by default every pod can access every pod
- pods aren't isolated by default

### Examples

Deny all egress(=outgoing) traffic from a group of pods(selector id=frontend)

<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=./manifests/networkpolicy-deny-egress.yaml) -->
<!-- The below code snippet is automatically added from ./manifests/networkpolicy-deny-egress.yaml -->
```yaml
api: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-egress-fronted
  namespace: default
spec:
  podSelector:
    matchLabels:
      id: frontend
  policyTypes:
  - Egress
```
<!-- MARKDOWN-AUTO-DOCS:END -->

Allow egress traffic from a group of pods(namespace=default, selector id=frontend) to

1. namespace with label id=ns1 **AND** port 80
2. a group of pods(id=backend) within the same namespace because there is no namespace selector.

<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=./manifests/networkpolicy-2-egress.yaml) -->

### Multiple NetworkPolicies

- it's possible to have multiple NetworkPolicies selecting the same group of pods
- If a pod has more than one NetworkPolicy:
  - then the union of all NetworkPolicies is applied
  - order doesn't affect policy result
