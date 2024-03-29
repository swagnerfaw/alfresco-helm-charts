# Alfresco Process services architecture

Below is a representation of a fully in-cluster Kuerbenetes deployment of APS.

```mermaid
%%{init: {'theme':'neutral'}}%%
flowchart TB

classDef alf fill:#0b0,color:#fff
classDef thrdP fill:#bb,color:#000
classDef k8s fill:#326ce5,stroke:#326ce5,stroke-width:2px,color:#fff

subgraph legend
  direction TB
  a[Alfresco workloads]:::alf
  t[Third party workloads]:::thrdP
  k[Kubernetes resources]:::k8s
end

legend ~~~ client[Client]
client -- "HTTP/S request /activiti-app" --> ingressAPS
client -- "HTTP/S request /activiti-admin" --> ingressAdmin

subgraph Kubernetes
    ingressAPS{{Ingress: release-name-ingress-aps}}
    ingressAPS -- Routes request --> serviceAPS[Service: release-name-service-aps]
    serviceAPS -- Directs request --> deploymentAPS[Deployment: release-name-aps]
    deploymentAPS -- Runs on --> podAPS[Pod: release-name-aps]
    deploymentAPS -- Uses --> configMapAPS>ConfigMap: release-name-configmap-aps]
    deploymentAPS -- Uses --> secretAPS>Secret: release-name-database-aps]
    podAPS -- Connects to --> servicePostgres[Service: pg-postgresql-aps]
    servicePostgres -- Directs request --> statefulSetPostgres[StatefulSet: pg-postgresql-aps]
    statefulSetPostgres -- Runs on --> podPostgres[Pod: pg-postgresql-aps]
    ingressAdmin{{Ingress: release-name-ingress-admin}}
    ingressAdmin -- Routes request --> serviceAdmin[Service: release-name-service-admin]
    serviceAdmin -- Directs request --> deploymentAdmin[Deployment: release-name-admin]
    deploymentAdmin -- Runs on --> podAdmin[Pod: release-name-admin]
    deploymentAdmin -- Uses --> configMapAdmin>ConfigMap: release-name-configmap-admin]
    deploymentAdmin -- Uses --> secretAdmin>Secret: release-name-database-admin]
    podAdmin -- Connects to --> servicePostgres
    class podAPS,podAdmin,deploymentAPS,deploymentAdmin alf
    class statefulSetPostgres,podPostgres thrdP
    class ingressAPS,serviceAPS,configMapAPS,secretAPS k8s
    class ingressAdmin,serviceAdmin,configMapAdmin,secretAdmin,servicePostgres k8s
    class ingressAPS,serviceAPS,configMapAPS,secretAPS k8s
    class ingressAdmin,serviceAdmin,configMapAdmin,secretAdmin,servicePostgres k8s
end
```

## AWS example deployment

Below is a representation of a deployment of APS which leverages AWS services.

```mermaid
%%{init: {'theme':'neutral'}}%%
flowchart TB

classDef alf fill:#0b0,color:#fff
classDef aws fill:#fa0,color:#fff
classDef k8s fill:#326ce5,stroke:#326ce5,stroke-width:2px,color:#fff

subgraph legend
  direction TB
  a[Alfresco workload]:::alf
  t[AWS resources]:::aws
  k[Kubernetes resources]:::k8s
end

legend ~~~ client[Client]

subgraph EKS
  ingressAPS{{Ingress: release-name-ingress-aps}}
  ingressAdmin{{Ingress: release-name-ingress-admin}}
  ingressAPS -- Routes request --> serviceAPS[Service: release-name-service-aps]
  serviceAPS -- Directs request --> deploymentAPS[Deployment: release-name-aps]
  deploymentAPS -- Runs on --> podAPS[Pod: release-name-aps]
  deploymentAPS -. Uses .-> configMapAPS>ConfigMap: release-name-configmap-aps]
  deploymentAPS -. Uses .-> secretAPS>Secret: release-name-database-aps]
  ingressAdmin -- Routes request --> serviceAdmin[Service: release-name-service-admin]
  serviceAdmin -- Directs request --> deploymentAdmin[Deployment: release-name-admin]
  deploymentAdmin -- Runs on --> podAdmin[Pod: release-name-admin]
  deploymentAdmin -. Uses .-> configMapAdmin>ConfigMap: release-name-configmap-admin]
  deploymentAdmin -. Uses .-> secretAdmin>Secret: release-name-database-admin]
  class ingressAPS,serviceAPS,configMapAPS,secretAPS k8s
  class ingressAdmin,serviceAdmin,configMapAdmin,secretAdmin k8s
  class podAPS,podAdmin,deploymentAPS,deploymentAdmin alf
end

subgraph AWS public network
  ELB[Elastic Load Balancer]-- HTTP/S requests --> ingressAPS
  ELB[Elastic Load Balancer]-- HTTP/S requests --> ingressAdmin
  class ELB aws
end

client -- "HTTP/S request /activiti-app" --> ELB
client -- "HTTP/S request /activiti-admin" --> ELB

subgraph AWS private network
  podAPS -- Connects to --> RDS[(AWS RDS instance)]
  podAdmin -- Connects to --> RDS[(AWS RDS instance)]
  podAPS -- Connects to --> ES[(AWS ElasticSearch instance)]
  class ES,RDS aws
end
```
