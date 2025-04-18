---
title: "Require unique host names in OpenShift routes"
category: OpenShift
version: 1.6.0
subject: Route
policyType: "validate"
description: >
    An Route host is a URL at which services may be made available externally. In most cases, these hosts should be unique across the cluster to ensure no routing conflicts occur. This policy checks an incoming Route resource to ensure its hosts are unique to the cluster.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//openshift/unique-routes/unique-routes.yaml" target="-blank">/openshift/unique-routes/unique-routes.yaml</a>

```yaml
---
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: unique-routes
  annotations:
    policies.kyverno.io/title: Require unique host names in OpenShift routes
    policies.kyverno.io/category: OpenShift
    policies.kyverno.io/severity: high
    kyverno.io/kyverno-version: 1.6.0
    policies.kyverno.io/minversion: 1.6.0
    kyverno.io/kubernetes-version: "1.20"
    policies.kyverno.io/subject: Route
    policies.kyverno.io/description: >-
      An Route host is a URL at which services may be made available externally. In most cases,
      these hosts should be unique across the cluster to ensure no routing conflicts occur.
      This policy checks an incoming Route resource to ensure its hosts are unique to the cluster.
spec:
  validationFailureAction: Enforce
  background: false
  rules:
    - name: require-unique-routes
      match:
        any:
          - resources:
              kinds:
                - route.openshift.io/v1/Route
      context:
        - name: hosts
          apiCall:
            urlPath: "/apis/route.openshift.io/v1/Routes"
            jmesPath: "items[].spec.host"
      preconditions:
        all:
          - key: "{{ request.operation || 'BACKGROUND' }}"
            operator: NotEquals
            value: "DELETE"
      validate:
        message: >-
          The Route host name must be unique.
        deny:
          conditions:
            all:
              - key: "{{ request.object.spec.host }}"
                operator: AnyIn
                value: "{{ hosts }}"

```
