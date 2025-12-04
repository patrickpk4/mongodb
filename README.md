# MongoDB Helm Chart

Este repositório contém um **Helm Chart completo para MongoDB**, projetado tanto para uso isolado quanto como **dependência** em aplicações (por exemplo: APIs .NET, Node.js, Go, Python).  
O chart fornece uma instalação **realista, configurável e extensível** do MongoDB em Kubernetes.

---

##  Funcionalidades

- StatefulSet com armazenamento persistente  
- Service ClusterIP (inclui opção headless)  
- Configuração de usuário e senha via **Secret**  
- Possibilidade de **usar Secret externo** (existingSecret pattern) 
- Storage configurável com PVC e `storageClass`  
- VolumeMount customizável  
- Autoescalonamento (HPA) opcional  
- NetworkPolicy opcional  
- RBAC (Role e RoleBinding) opcionais  
- Pode ser usado como **subchart (dependency)**  
- Suporte a customizações via `values.yaml`

---

##  Instalação

### 1. Adicionar o repositório

```bash
helm repo add mongo-dep https://patrickpk4.github.io/mongodb/
helm repo update
```

### 2. Instalar o chart

```bash
helm install my-mongo mongo-dep/mongodb
```

> Se estiver usando como dependência dentro de outro chart (por exemplo `api`), inclua no `Chart.yaml` do chart pai (veja seção *Uso como Dependência*).

---

##  Configuração (`values.yaml`)

Abaixo estão as principais opções disponíveis. Ajuste conforme sua necessidade.

### Controle de enable/disable
```yaml
mongodb:
  enabled: true
```

### Service / Rede
```yaml
service:
  type: ClusterIP
  port: 27017
```

### Volumes e Persistência
```yaml
volumeMounts:
  - name: mongodb-data
    mountPath: /data/db
    readOnly: false

volumeClaimTemplates:
  - metadata:
      name: mongodb-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: mongodb-nfs
      resources:
        requests:
          storage: 1Gi
```

> **Atenção:** use um `storageClassName` disponível no seu cluster, ou deixe em branco para usar a StorageClass padrão.

### Autoescalonamento (HPA)
```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

### RBAC (Role / RoleBinding)
```yaml
role:
  enabled: true

rolebinding:
  enabled: true
```

### NetworkPolicy
```yaml
networkpolicy:
  enabled: true
```

---

##  Configuração de Secret (existingSecret pattern)

Você pode **criar o Secret pelo chart** ou **usar um Secret existente**. O padrão recomendado é usar `existingSecret` quando estiver integrando com outros charts.

### Exemplo de `values.yaml` para Secret
```yaml
secret:
  create: true               # se true → cria o Secret pelo chart
  existingSecret: ""         # se preenchido → usa Secret existente e ignora create
  usernameKey: Mongo__User
  passwordKey: Mongo__Password
  username: mongouser        # usado somente quando create=true
  password: mongopwd         # usado somente quando create=true
```

### Template (secret.yaml)
```gotmpl
{{- if and (.Values.secret.create) (not .Values.secret.existingSecret) }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-secret
type: Opaque
stringData:
  {{ .Values.secret.usernameKey }}: {{ .Values.secret.username | quote }}
  {{ .Values.secret.passwordKey }}: {{ .Values.secret.password | quote }}
{{- end }}
```

### Como consumir no Deployment (exemplo)
```yaml
env:
  - name: MONGO_USERNAME
    valueFrom:
      secretKeyRef:
        name: {{ default (printf "%s-secret" .Release.Name) .Values.secret.existingSecret }}
        key: {{ .Values.secret.usernameKey }}

  - name: MONGO_PASSWORD
    valueFrom:
      secretKeyRef:
        name: {{ default (printf "%s-secret" .Release.Name) .Values.secret.existingSecret }}
        key: {{ .Values.secret.passwordKey }}
```

> Com isso, se `existingSecret` estiver preenchido, **nenhum Secret será criado** pelo chart e o chart usará o secret informado.

---

##  Uso como dependência em outro Chart (ex.: API)

No `Chart.yaml` do chart pai (API), adicione:
```yaml
dependencies:
  - name: mongodb
    version: "0.1.3"
    repository: "https://patrickpk4.github.io/mongodb/"
    condition: mongodb.enabled
```

No `values.yaml` do chart pai (API), configure como quer consumir o Secret:
```yaml
mongodb:
  enabled: true
  secret:
    create: false
    existingSecret: "my-external-mongo-secret"  # nome do secret a ser usado
    usernameKey: Mongo__User
    passwordKey: Mongo__Password
```

- Se `existingSecret` for definido → **o chart do MongoDB e o chart da API não criarão secrets adicionais** (desde que ambos estejam configurados para respeitar `existingSecret`).
- Se `existingSecret` estiver vazio e `create=true` no chart do MongoDB → o Mongo criará seu secret e a API poderá referenciá-lo.

---

##  Estrutura sugerida do repositório

```
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
```

---

##  Testes e verificação

Após instalar:

```bash
kubectl get pods
kubectl get svc
kubectl get pvc
kubectl logs statefulset/mongodb-statefulset -f
```

Para entrar no pod e usar o cliente Mongo (se disponível na imagem):

```bash
kubectl exec -it mongodb-statefulset-0 -- sh
# dentro do pod
mongo
# ou
mongosh
```

Comandos Mongo:
```js
show dbs
use admin
show collections
db.minhaColecao.find().pretty()
```

---

##  Contribuições

Pull requests, issues e sugestões são bem-vindas. Sinta-se à vontade para propor melhorias ou adicionar recursos.

---

Se quiser, eu gero este README.md no repositório (arquivo pronto) — quer que eu gere o arquivo agora?
