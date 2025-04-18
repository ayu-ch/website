---
title: "Enforce etcd encryption in OpenShift in CEL expressions"
category: OpenShift
version: 1.11.0
subject: APIServer
policyType: "validate"
description: >
    Encryption at rest is a security best practice. This policy ensures encryption is enabled for etcd in OpenShift clusters.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//openshift-cel/enforce-etcd-encryption/enforce-etcd-encryption.yaml" target="-blank">/openshift-cel/enforce-etcd-encryption/enforce-etcd-encryption.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: enforce-etcd-encryption
  annotations:
    policies.kyverno.io/title: Enforce etcd encryption in OpenShift in CEL expressions
    policies.kyverno.io/category: OpenShift
    policies.kyverno.io/severity: high
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: APIServer
    policies.kyverno.io/description: >-
      Encryption at rest is a security best practice. This policy ensures encryption is enabled for etcd in OpenShift clusters.
spec:
  validationFailureAction: Enforce
  background: true
  rules:
  - name: check-etcd-encryption
    match:
      any:
      - resources:
          kinds:
          - config.openshift.io/v1/APIServer
          operations:
          - CREATE
          - UPDATE
    validate:
      cel:
        expressions:
          - expression: "has(object.spec.encryption)"
            message: >-
              Encryption should be enabled for etcd


```
