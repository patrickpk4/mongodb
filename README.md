# MongoDB Helm Chart (Dependency)

Este repositório contém um Helm Chart do **MongoDB** projetado para ser
utilizado como **dependência** em aplicações, como por exemplo a sua
API.

## 📦 Objetivo do Chart

Fornecer uma forma simples e consistente de: - Implantar uma instância
MongoDB em Kubernetes - Ser usado como dependência (`charts/`) em outros
Helm Charts - Permitir configuração via `values.yaml` da aplicação
principal

## 🧩 Como usar este Chart como dependência

### 1. Adicione a seção `dependencies` no `Chart.yaml` da sua aplicação:

``` yaml
dependencies:
  - name: mongodb
    version: 0.1.0
    repository: "https://github.com/patrickpk4/mongodb"
```

### 2. Configure os valores do MongoDB no `values.yaml` da sua aplicação:

``` yaml
mongodb:
  image:
    tag: "7.0"
  auth:
    enabled: true
    rootUser: admin
    rootPassword: "MinhaSenha123"
```

### 3. Atualize as dependências:

``` bash
helm dependency update
```

Agora o MongoDB será incluído automaticamente no deploy da sua
aplicação.

## 📁 Estrutura do repositório

    mongodb-chart/
      ├── Chart.yaml
      ├── values.yaml
      ├── templates/
      │     ├── deployment.yaml
      │     ├── service.yaml
      │     ├── secret.yaml
      │     └── pvc.yaml

## 🛠 Configurações principais

-   **Autenticação habilitada**
-   **PVC para persistência**
-   **Service para comunicação interna**
-   **Configuração compatível para uso como dependência Helm**

## ✨ Contribuições

Sinta-se à vontade para enviar PRs, abrir issues ou deixar sugestões.

------------------------------------------------------------------------

Feito com ☕ e Kubernetes.
