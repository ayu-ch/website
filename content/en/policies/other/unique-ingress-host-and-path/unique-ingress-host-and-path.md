---
title: "Unique Ingress Host and Path"
category: Sample
version: 1.6.0
subject: Ingress
policyType: "validate"
description: >
    Similar to the ability to check the uniqueness of hosts and paths independently, it is possible to check for uniqueness of them both together across a cluster. This policy ensures that no Ingress can be created or updated unless it is globally unique with respect to host plus path combination.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/unique-ingress-host-and-path/unique-ingress-host-and-path.yaml" target="-blank">/other/unique-ingress-host-and-path/unique-ingress-host-and-path.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: unique-ingress-host-and-path
  annotations:
    policies.kyverno.io/title: Unique Ingress Host and Path
    policies.kyverno.io/category: Sample
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Ingress
    kyverno.io/kyverno-version: 1.7.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.23"
    policies.kyverno.io/description: >-
      Similar to the ability to check the uniqueness of hosts and paths independently,
      it is possible to check for uniqueness of them both together across a cluster.
      This policy ensures that no Ingress can be created or updated unless it is
      globally unique with respect to host plus path combination.
spec:
  validationFailureAction: Audit
  background: false
  rules:
    - name: check-host-path-combo
      match:
        any:
        - resources:
            kinds:
              - Ingress
      preconditions:
        all:
        - key: "{{ request.operation || 'BACKGROUND' }}"
          operator: NotEquals
          value: DELETE
      context:
        - name: rules
          apiCall:
            urlPath: "/apis/networking.k8s.io/v1/ingresses"
            jmesPath: "items[].spec.rules[]"
      validate:
        message: "The Ingress host and path combination must be unique across the cluster."
        foreach:
        - list: "request.object.spec.rules[]"
          deny:
            conditions:
              all:
              - key: "{{ element.http.paths[].path }}"
                operator: AnyIn
                value: "{{ rules[?host=='{{element.host}}'][].http.paths[].path }}"
```
