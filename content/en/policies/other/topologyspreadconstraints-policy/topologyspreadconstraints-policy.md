---
title: "Spread Pods Across Nodes & Zones"
category: Sample
version: 1.8.0
subject: Deployment, StatefulSet
policyType: "validate"
description: >
    Deployments to a Kubernetes cluster with multiple availability zones often need to distribute those replicas to align with those zones to ensure site-level failures do not impact availability. This policy ensures topologySpreadConstraints are defined,  to spread pods over nodes and zones. Deployments or Statefulsets with leass than 3  replicas are skipped.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/topologyspreadconstraints-policy/topologyspreadconstraints-policy.yaml" target="-blank">/other/topologyspreadconstraints-policy/topologyspreadconstraints-policy.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: topologyspreadconstraints-policy
  annotations:
    policies.kyverno.io/title: Spread Pods Across Nodes & Zones
    kyverno.io/kubernetes-version: "1.22-1.23"
    kyverno.io/kyverno-version: 1.8.0
    policies.kyverno.io/category: Sample
    policies.kyverno.io/description: >-
      Deployments to a Kubernetes cluster with multiple availability zones often need to
      distribute those replicas to align with those zones to ensure site-level failures
      do not impact availability. This policy ensures topologySpreadConstraints are defined, 
      to spread pods over nodes and zones. Deployments or Statefulsets with leass than 3 
      replicas are skipped.
    policies.kyverno.io/minversion: 1.8.0
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Deployment, StatefulSet
    
spec:
  background: true
  failurePolicy: Ignore
  validationFailureAction: Audit
  rules:
    - name: spread-pods
      match:
        any:
          - resources:
              kinds:
                - Deployment
                - StatefulSet
      preconditions:
        all:
          - key: "{{ request.object.spec.replicas }}"
            operator: GreaterThanOrEquals
            value: 3
      validate:
        message: "topologySpreadConstraint for kubernetes.io/hostname & topology.kubernetes.io/zone are required"
        deny:
          conditions:
            any:
              - key: "{{request.object.spec.template.spec.topologySpreadConstraints[?topologyKey=='kubernetes.io/hostname' || topologyKey=='topology.kubernetes.io/zone'] || `[]` | length(@) }}"
                operator: NotEquals
                value: 2

```
