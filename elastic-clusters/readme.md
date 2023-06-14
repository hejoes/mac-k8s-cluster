# Provisioning elastic, kibana and prometheus exporter

[[_TOC_]]

## General

https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-elasticsearch.html
https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-kibana.html

## Setup

1. Create a new helm values file to the environment folder (for example /test/values-ee-test-content-api-v3.yaml) with the following content:

  ```yaml
  users: dGVzdHVzZXI6JDJhJDEwJHZqZldVd2FDY2ljN2lnZmx2SHhCQ2VXQkQ2TnJVbURKZDRQaUdYS2JvdHdrdXdjbFB6aVZtCnRlc3R1c2VyMTokMmEkMTAkakFXNnhDUnZDV0Q1VzlPaXp4bC9DT0U5bTRTQ1BnR2RUQXppdno5SVJKaGtUcjBvSFVpd08K
  user_roles: c3VwZXJ1c2VyOnRlc3R1c2VyLHRlc3R1c2VyMQo=

  elastic:
    url: "ee-test-content-api-v3.kibana.test.delfi.ee.elastic.test.delfi.net"
    resources:
      storage: 10Gi
      memory: 2Gi
      cpu:
        request: 500m
        limit: 4

  kibana:
    url: "ee-test-content-api-v3.kibana.test.delfi.ee.kibana.test.delfi.ee"
    resources:
      memory: 1Gi
      cpu:
        request: 200m
        limit: 1
  exporter:
    extraEnvSecrets:
      ES_PASSWORD:
        secret: ee-test-content-api-v3-es-elastic-user
    es: # Must be http://{{ .Release.name }}-es-http:9200 where release.name = argoApplication name
      uri: http://ee-test-content-api-v3.kibana.test.delfi.ee-es-http:9200

  ```

2. Create a new argoApplication `ee-test-content-api-v3.yaml` file into [ArgoCD application templates folder](https://gitlab.delfi.net/eg/infra/argocd/-/tree/master/argocd/templates/applications) repository pointing to the newly created values file:

  ```yaml
  apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: ee-test-content-api-v3
    namespace: argocd
  spec:
    destination:
      namespace: elastic
      server: https://kubernetes.default.svc
    project: infra
    source:
      helm:
        valueFiles:
        - values-global.yaml
        - values-ee-test-content-api-v3.yaml
      path: elastic-clusters/{{ .Values.enviroment }}
      repoURL: https://gitlab.delfi.net/eg/infra/infrastructure-helm.git
      targetRevision: eck-operator
    ignoreDifferences: 
    - group: elasticsearch.k8s.elastic.co
      kind: Elasticsearch
      jqPathExpressions:
      - .spec.nodeSets[0].volumeClaimTemplates[0].status
      - .spec.nodeSets[0].podTemplate.metadata
  ```

3. Sync the `argo-cd` app in the corresponding ArgoCD interface for detecting the previously added argoApplication.

4. Sync the added argoApplication in the corresponding ArgoCD interface.

5. Login (username: elastic) to kibana via port-forwarding or via DNS according to the config in values file (i.e. https://ee-test-content-api-v3.kibana.test.delfi.ee). For password use:

  ```bash
  export CLUSTERNAME=<replace-with-the-new-clustername>
  kubectl get secret $CLUSTERNAME-es-elastic-user -n elastic -o=jsonpath='{.data.elastic}' | base64 --decode; echo
  ```

6. Create a snapshot repository via kibana console:
  
  ```json
  POST _snapshot/my_s3_repository
  {
    "type": "s3",
    "settings": {
      "bucket": "eg-test-elastic-snapshot",
      "path_style_access": true,	
      "base_path": "$CLUSTERNAME", 
      "endpoint": "https://s3.teliahybridcloud.com/" 
    }
  }
  ```

7. Create a snapshot policy via kibana UI (reccomended) or customise the request and via console. See examples in snapshots/readme.md

  ```json
  PUT /_slm/policy/daily-snapshots
  {
    "schedule": "0 30 1 * * ?", 
    "name": "<daily-snap-{now/d}>", 
    "repository": "my_s3_repository", 
    "config": { 
      "indices": ["data-*", "important"], 
      "ignore_unavailable": false,
      "include_global_state": false
    },
    "retention": { 
      "expire_after": "30d", 
      "min_count": 5, 
      "max_count": 50 
    }
  }
  ```

## Users

See `./users/readme.md`