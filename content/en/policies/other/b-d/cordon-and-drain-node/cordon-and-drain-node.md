---
title: "Cordon and Drain Node"
category: other
version: 1.10.0
subject: Node
policyType: "mutate"
description: >
    There are cases where either an operations or security incident may occur and Nodes should be evacuated and placed in an unused state for further analysis. For example, a Node is found to be running a vulnerable version of a CRI engine or kernel and to minimize chances of a compromise may need to be decommissioned so another can be built. This policy shows how to use Kyverno to both cordon and drain a given Node and uses a hypothetical label being written to it called `testing=drain` to illustrate the point. For production use, the match block should be modified to trigger on the appropriate condition.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/b-d/cordon-and-drain-node/cordon-and-drain-node.yaml" target="-blank">/other/b-d/cordon-and-drain-node/cordon-and-drain-node.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: cordon-and-drain-node
  annotations:
    policies.kyverno.io/title: Cordon and Drain Node
    policies.kyverno.io/category: other
    policies.kyverno.io/subject: Node
    kyverno.io/kyverno-version: 1.10.1
    policies.kyverno.io/minversion: 1.10.0
    kyverno.io/kubernetes-version: "1.26"
    policies.kyverno.io/description: >-
      There are cases where either an operations or security incident may occur and Nodes
      should be evacuated and placed in an unused state for further analysis. For example,
      a Node is found to be running a vulnerable version of a CRI engine or kernel and to
      minimize chances of a compromise may need to be decommissioned so another can be built.
      This policy shows how to use Kyverno to both cordon and drain a given Node and uses a
      hypothetical label being written to it called `testing=drain` to illustrate the point.
      For production use, the match block should be modified to trigger on the appropriate
      condition.
spec:
  rules:
    - name: mutate-node
      match:
        any:
        - resources:
            kinds:
            - Node
            operations:
            - UPDATE
            selector:
              matchLabels:
                testing: drain
      mutate:
        targets:
        - apiVersion: v1
          kind: Node
          name: "{{request.object.metadata.name}}"
        patchStrategicMerge:
          spec:
            unschedulable: true
            taints:
            - effect: NoExecute
              key: kyverno-evicted
              timeAdded: "{{ time_now_utc() }}" 

```
