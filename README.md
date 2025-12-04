MongoDB Helm Chart

Este repositório contém um Helm Chart completo para MongoDB, projetado tanto para uso isolado quanto como dependência em aplicações como APIs .NET, Node.js, Go, Python etc.

O objetivo deste chart é fornecer uma instalação realista, configurável e extensível do MongoDB dentro de um cluster Kubernetes.

 Funcionalidades

StatefulSet com armazenamento persistente

Service ClusterIP configurável

Autoescalonamento (HPA) opcional

NetworkPolicy opcional

RBAC (Role e RoleBinding) opcionais

Configuração de usuário e senha via Secret

Suporte a Secret externo

Pode ser usado como subchart (dependency)

Storage configurável com PVC

VolumeMount customizável

 Instalação
1. Adicionar o repositório
helm repo add mongo-dep https://patrickpk4.github.io/mongodb/
helm repo update

2. Instalar o chart
helm install my-mongo mongo-dep/mongodb

 Configurações Disponíveis (values.yaml)

Abaixo estão todas as funcionalidades incluídas, com explicação.

 Configuração do Secret
Criar Secret automaticamente
secret:
  create: true
  existingSecret: ""
  usernameKey: Mongo__User
  passwordKey: Mongo__Password
  username: mongouser
  password: mongopwd

Usar Secret já existente
secret:
  create: false
  existingSecret: "mongodb-secret"
  usernameKey: Mongo__User
  passwordKey: Mongo__Password

 Habilitar/Desabilitar o Chart como Dependência
mongodb:
  enabled: true

 Configuração de Rede
service:
  type: ClusterIP
  port: 27017

 Volumes e Persistência
VolumeMount
volumeMounts:
  - name: mongodb-data
    mountPath: /data/db
    readOnly: false

PVC com storageClassNfs
volumeclaimtemplates:
  - metadata:
      name: mongodb-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: mongodb-nfs
      resources:
        requests:
          storage: 1Gi

 Autoescalonamento (HPA)
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

 RBAC (Role e RoleBinding)
role:
  enabled: true

rolebinding:
  enabled: true

 NetworkPolicy
networkpolicy:
  enabled: true

 Uso como Dependência

No Chart.yaml da API:

dependencies:
  - name: mongodb
    version: "0.1.3"
    repository: "https://patrickpk4.github.io/mongodb/"


No values.yaml da API:

mongodb:
  enabled: true
  secret:
    create: false
    existingSecret: "mongodb-secret"
    usernameKey: Mongo__User
    passwordKey: Mongo__Password

 Estrutura do Repositório
mongodb/
 ├── charts/
 ├── templates/
 │    ├── statefulset.yaml
 │    ├── service.yaml
 │    ├── secret.yaml
 │    ├── role.yaml
 │    ├── rolebinding.yaml
 │    ├── networkpolicy.yaml
 │    ├── hpa.yaml
 │    └── _helpers.tpl
 ├── Chart.yaml
 ├── values.yaml
 └── README.md

 Testes
kubectl get pods
kubectl logs statefulset/mongodb-statefulset -f
kubectl exec -it mongodb-statefulset-0 -- mongosh

 Contribuições

Contribuições são bem-vindas!
Pull Requests, Issues e sugestões são aceitas.
