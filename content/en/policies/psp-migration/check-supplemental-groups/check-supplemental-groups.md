---
title: "Check supplementalGroups"
category: PSP Migration
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Supplemental groups control which group IDs containers add and can coincide with restricted groups on the host. Pod Security Policies (PSP) allowed a range of these group IDs to be specified which were allowed. This policy ensures any Pod may only specify supplementalGroup IDs between 100-200 or 500-600.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//psp-migration/check-supplemental-groups/check-supplemental-groups.yaml" target="-blank">/psp-migration/check-supplemental-groups/check-supplemental-groups.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: psp-check-supplemental-groups
  annotations:
    policies.kyverno.io/title: Check supplementalGroups
    policies.kyverno.io/category: PSP Migration
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Supplemental groups control which group IDs containers add and can coincide with
      restricted groups on the host. Pod Security Policies (PSP) allowed a range of
      these group IDs to be specified which were allowed. This policy ensures any Pod
      may only specify supplementalGroup IDs between 100-200 or 500-600.
spec:
  background: false
  validationFailureAction: Audit
  rules:
  - name: supplementalgroup-ranges
    match:
      any:
      - resources:
          kinds:
            - Pod
    preconditions:
      all:
      - key: "{{ request.operation || 'BACKGROUND' }}"
        operator: AnyIn
        value:
        - CREATE
        - UPDATE
    validate:
      message: Any supplementalGroup ID must be within the range 100-200 or 500-600.
      pattern:
        spec:
          =(securityContext):
            =(supplementalGroups): 100-200 | 500-600
```
