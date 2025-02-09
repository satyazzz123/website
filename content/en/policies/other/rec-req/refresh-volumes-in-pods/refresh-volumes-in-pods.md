---
title: "Refresh Volumes in Pods"
category: Other
version: 1.9.0
subject: Pod,ConfigMap
policyType: "mutate"
description: >
    Although ConfigMaps and Secrets mounted as volumes to a Pod, when the contents change, will eventually propagate to the Pods mounting them, this process may take between 60-90 seconds. In order to reduce that time, a modification made to downstream Pods will cause the changes to take effect almost instantly. This policy watches for changes to ConfigMaps which have been marked for this quick reloading process which contain the label `kyverno.io/watch=true` and will write an annotation to any Pods which mount them as volumes causing a fast refresh in their contents. See the related policy entitled "Refresh Environment Variables in Pods" for a similar reloading process when ConfigMaps and Secrets are consumed as environment variables instead. Use of this policy may require providing the Kyverno ServiceAccount with permission to update Pods.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/rec-req/refresh-volumes-in-pods/refresh-volumes-in-pods.yaml" target="-blank">/other/rec-req/refresh-volumes-in-pods/refresh-volumes-in-pods.yaml</a>

```yaml
apiVersion: kyverno.io/v2beta1
kind: ClusterPolicy
metadata:
  name: refresh-volumes-in-pods
  annotations:
    policies.kyverno.io/title: Refresh Volumes in Pods
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod,ConfigMap
    kyverno.io/kyverno-version: 1.9.0
    policies.kyverno.io/minversion: 1.9.0
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/description: >-
      Although ConfigMaps and Secrets mounted as volumes to a Pod, when the contents change,
      will eventually propagate to the Pods mounting them, this process may take between 60-90 seconds.
      In order to reduce that time, a modification made to downstream Pods will cause the changes
      to take effect almost instantly. This policy watches for changes to ConfigMaps which have been
      marked for this quick reloading process which contain the label `kyverno.io/watch=true` and
      will write an annotation to any Pods which mount them as volumes causing a fast refresh in their
      contents. See the related policy entitled "Refresh Environment Variables in Pods" for a similar
      reloading process when ConfigMaps and Secrets are consumed as environment variables instead.
      Use of this policy may require providing the Kyverno ServiceAccount with permission
      to update Pods.
spec:
  mutateExistingOnPolicyUpdate: false
  rules:
  - name: refresh-from-configmap-volume
    match:
      any:
      - resources:
          kinds:
          - ConfigMap
          selector:
            matchLabels:
              kyverno.io/watch: "true"
    preconditions:
      all:
      - key: "{{ request.operation }}"
        operator: Equals
        value: UPDATE
    mutate:
      targets:
        - apiVersion: v1
          kind: Pod
          namespace: "{{ request.namespace }}"
      patchStrategicMerge:
        metadata:
          annotations:
            corp.org/random: "{{ random('[0-9a-z]{8}') }}"
        spec:
          volumes:
          - configMap:
              <(name): "{{ request.object.metadata.name }}"
```
