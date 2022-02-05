---
title: "Disallow Host Process"
category: Pod Security Standards (Baseline)
version: 
subject: Pod
policyType: "validate"
description: >
    Windows pods offer the ability to run HostProcess containers which enables privileged access to the Windows node. Privileged access to the host is disallowed in the baseline policy. HostProcess pods are an alpha feature as of Kubernetes v1.22. This policy ensures the `hostProcess` field, if present, is set to `false`.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/disallow_hostprocess/disallow_hostprocess.yaml" target="-blank">/other/disallow_hostprocess/disallow_hostprocess.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-host-process
  annotations:
    policies.kyverno.io/category: Pod Security Standards (Baseline)
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    kyverno.io/kyverno-version: 1.5.2
    kyverno.io/kubernetes-version: "1.22"
    policies.kyverno.io/description: >-
      Windows pods offer the ability to run HostProcess containers which enables privileged
      access to the Windows node. Privileged access to the host is disallowed in the baseline
      policy. HostProcess pods are an alpha feature as of Kubernetes v1.22. This policy ensures
      the `hostProcess` field, if present, is set to `false`.
spec:
  validationFailureAction: audit
  background: true
  rules:
  - name: host-process-containers
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: >-
        HostProcess containers are disallowed. The fields spec.securityContext.windowsOptions.hostProcess,
        spec.containers[*].securityContext.windowsOptions.hostProcess, spec.initContainers[*].securityContext.windowsOptions.hostProcess,
        and spec.ephemeralContainers[*].securityContext.windowsOptions.hostProcess must either be undefined
        or set to `false`.
      pattern:
        spec:
          =(ephemeralContainers):
            - =(securityContext):
                =(windowsOptions):
                  =(hostProcess): "false"
          =(initContainers):
            - =(securityContext):
                =(windowsOptions):
                  =(hostProcess): "false"
          containers:
            - =(securityContext):
                =(windowsOptions):
                  =(hostProcess): "false"
```