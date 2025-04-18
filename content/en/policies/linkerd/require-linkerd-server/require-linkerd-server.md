---
title: "Require Linkerd Server"
category: Linkerd
version: 
subject: Deployment, Server
policyType: "validate"
description: >
    In Linkerd 2.11, a Server resource selects ports on a set of Pods in the same Namespace and is used to deny traffic which then must be authorized later. Ensuring that Linkerd policy is enforced on Pods in the mesh is important to maintaining a secure environment. This policy, requiring Linkerd 2.11+, has two rules designed to check Deployments (exposing ports) and Services to ensure a corresponding Server resource exists first.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//linkerd/require-linkerd-server/require-linkerd-server.yaml" target="-blank">/linkerd/require-linkerd-server/require-linkerd-server.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-linkerd-server
  annotations:
    policies.kyverno.io/title: Require Linkerd Server
    policies.kyverno.io/category: Linkerd
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Deployment, Server
    kyverno.io/kyverno-version: "1.8.0"
    kyverno.io/kubernetes-version: "1.24"
    policies.kyverno.io/description: >-
      In Linkerd 2.11, a Server resource selects ports on a set of Pods in the
      same Namespace and is used to deny traffic which then must be authorized later.
      Ensuring that Linkerd policy is enforced on Pods in the mesh is important to maintaining
      a secure environment. This policy, requiring Linkerd 2.11+, has two rules designed to check
      Deployments (exposing ports) and Services to ensure a corresponding Server resource
      exists first.
spec:
  validationFailureAction: Audit
  background: true
  rules:
  - name: check-deployment-has-server
    match:
      any:
      - resources:
          kinds:
          - Deployment
    preconditions:
      all:
      - key: "{{ request.object.spec.template.spec.containers[].ports[] || `[]` | length(@) }}"
        operator: GreaterThanOrEquals
        value: 1
    context:
    - name: server_count
      apiCall:
        urlPath: "/apis/policy.linkerd.io/v1beta1/namespaces/{{request.namespace}}/servers"
        jmesPath: "items[?label_match(spec.podSelector.matchLabels, `{{request.object.spec.template.metadata.labels}}`)] | length(@)"
    validate:
      message: "Every Deployment declaring ports requires a matching Server."
      deny:
        conditions:
          any:
          - key: "{{server_count}}"
            operator: LessThan
            value: 1
  - name: check-service-has-server
    match:
      any:
      - resources:
          kinds:
          - Service
    context:
    - name: server_count
      apiCall:
        urlPath: "/apis/policy.linkerd.io/v1beta1/namespaces/{{request.namespace}}/servers"
        jmesPath: "items[?label_match(spec.podSelector.matchLabels, `{{request.object.spec.selector}}`)] | length(@)"
    validate:
      message: "Every Service requires a matching Server."
      deny:
        conditions:
          any:
          - key: "{{server_count}}"
            operator: LessThan
            value: 1
```
