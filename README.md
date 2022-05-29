# cks-notes

- [cks-notes](#cks-notes)
  - [Kubernetes Secure Architecture](#kubernetes-secure-architecture)
    - [Public Key Infrastructture (PKI)](#public-key-infrastructture-pki)
    - [[P] Find various Kubernetes certificates](#p-find-various-kubernetes-certificates)
    - [Links-1](#links-1)
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

## Kubernetes Secure Architecture

![Kubernetes Architecture](/img/1.png "Kubernetes Architecture")

### Public Key Infrastructture (PKI)

CA (Certificate Authority) is trusted root for all the certificates inside the cluster.

All cluster certificates are signed by the CA.

![CA scheme](/img/2.png "CA scheme")

Some of the components have only client certificate to communicate with API server, some of them have server certificates for be able to verify the client.

![Interaction between server and client certificates](/img/3.png "Interaction between server and client certificates")

### [P] Find various Kubernetes certificates

1. CA
2. API server cert
3. ETCD server cert
4. API -> ETCD
5. API -> kubelet
6. Scheduler -> API
7. controller-manager -> API
8. kubelet -> API
9. kubelet server cert

**Possible places:**

    ```
    # main certificates
    $ ls -l /etc/kubernetes/pki
    -rw-r--r-- 1 root root 1155 May 10 07:00 apiserver-etcd-client.crt
    -rw------- 1 root root 1679 May 10 07:00 apiserver-etcd-client.key
    -rw-r--r-- 1 root root 1164 May 10 07:00 apiserver-kubelet-client.crt
    -rw------- 1 root root 1675 May 10 07:00 apiserver-kubelet-client.key
    -rw-r--r-- 1 root root 1285 May 10 07:00 apiserver.crt
    -rw------- 1 root root 1675 May 10 07:00 apiserver.key
    -rw-r--r-- 1 root root 1099 May 10 07:00 ca.crt
    -rw------- 1 root root 1675 May 10 07:00 ca.key
    drwxr-xr-x 2 root root 4096 May 10 07:00 etcd
    -rw-r--r-- 1 root root 1115 May 10 07:00 front-proxy-ca.crt
    -rw------- 1 root root 1675 May 10 07:00 front-proxy-ca.key
    -rw-r--r-- 1 root root 1119 May 10 07:00 front-proxy-client.crt
    -rw------- 1 root root 1675 May 10 07:00 front-proxy-client.key
    -rw------- 1 root root 1675 May 10 07:00 sa.key
    -rw------- 1 root root  451 May 10 07:00 sa.pub

    # scheduler cert
    $ cat /etc/kubernetes/scheduler.conf
        ...
        client-certificate-data: LS0...
        client-key-data: LS0...

    # controller-manager cert
    $ cat /etc/kubernetes/controller-manager.conf
        ...
        client-certificate-data: LS0...
        client-key-data: LS0...

    # kubelet -> API & kubelet server
    $ cat /etc/kubernetes/kubelet.conf
        ...
        client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
        client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
    $ sudo ls -l /var/lib/kubelet/pki
    -rw------- 1 root root 2830 May 10 07:00 kubelet-client-2022-05-10-07-00-12.pem
    lrwxrwxrwx 1 root root   59 May 10 07:00 kubelet-client-current.pem -> /var/lib/kubelet/pki/kubelet-client-2022-05-10-07-00-12.pem
    -rw-r--r-- 1 root root 2266 May 10 07:00 kubelet.crt
    -rw------- 1 root root 1675 May 10 07:00 kubelet.key
    ```

### Links-1

- [All You Need to Know About Certificates in Kubernetes](https://www.youtube.com/watch?v=gXz4cq3PKdg)
- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components)
- [PKI certificates and requirements](https://kubernetes.io/docs/setup/best-practices/certificates)

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

Deny all egress traffic from specific group of pods

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
