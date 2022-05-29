# Network policies

- [Network policies](#network-policies)
  - [Examples](#examples)
  - [Multiple NetworkPolicies](#multiple-networkpolicies)

- Firewall rules in Kubernetes
- Implemented by the Nwtwork plugins CNI (Calico / Weave / etc)
- Namespace level
- Restrict the ingress and/or Egress for a group of Pods based on certain rules and conditions

Without NetworkPolicies:

- by default every pod can access every pod
- pods aren't isolated by default

## Examples

Deny all egress(=outgoing) traffic from a group of pods(selector id=frontend)

<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=../../manifests/networkpolicy-deny-egress.yaml) -->
<!-- The below code snippet is automatically added from ../../manifests/networkpolicy-deny-egress.yaml -->
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

<!-- MARKDOWN-AUTO-DOCS:START (CODE:src=../../manifests/networkpolicy-2-egress.yaml) -->
<!-- The below code snippet is automatically added from ../../manifests/networkpolicy-2-egress.yaml -->
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
  egress:
  - to:
    - namespaceSelector:
      matchLabels:
        id: ns1
    ports:
    - protocol: TCP
      port: 80
  - to:
    - podSelector:
        matchLabels:
          id: backend
```
<!-- MARKDOWN-AUTO-DOCS:END -->

## Multiple NetworkPolicies

- it's possible to have multiple NetworkPolicies selecting the same group of pods
- If a pod has more than one NetworkPolicy:
  - then the union of all NetworkPolicies is applied
  - order doesn't affect policy result
