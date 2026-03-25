# 📘 Instalação do 3scale no OpenShift – Guia Completo

> **Ambiente utilizado:** OpenShift Container Platform **4.18.35**  
> **Operator:** 3scale API Management `0.10.1` — channel `threescale-2.13`  
> **Namespace:** `3scale`

---

## 🧱 1. Provisionamento do Ambiente

### 🎯 Objetivo
Criar um cluster com recursos suficientes para suportar o 3scale.

### ⚙️ Sizing recomendado (mínimo funcional)

| Componente    | CPU      | Memória    |
|---------------|----------|------------|
| Worker Nodes  | ≥ 3      | ≥ 8GB cada |
| Total cluster | ≥ 6 vCPU | ≥ 16GB     |

### ⚠️ Observações

- Ambientes pequenos causam:
  - `Insufficient CPU`
  - Pods não agendados
- APIcast e System são sensíveis a recursos

---

## ⚙️ 2. Instalação do Operator via OperatorHub

<a href="3scale/operador/Readme.md" target="_blank">Guia do Operador</a>


---

## 🔐 3. Login via CLI (`oc`)

Alterne para o terminal e faça login no cluster usando o token de acesso:

```bash
oc login \
  --token=sha256~eCwlVjJFg3vXYzujKDfbAYe3ORmtnkT6nlXX9awyQsM \
  --server=https://api.itz-ot6ru4.infra01-lb.syd05.techzone.ibm.com:6443
```

Selecione o projeto `3scale`:

```bash
oc project 3scale
```

---

## 🧩 4. Configuração e Apply do APIManager

### 📄 Arquivo `apimanager.yml`

```yaml
kind: APIManager
apiVersion: apps.3scale.net/v1alpha1
metadata:
  name: apimanager-sample
  namespace: 3scale
spec:
  wildcardDomain: apps.itz-ot6ru4.infra01-lb.syd05.techzone.ibm.com
  system:
    fileStorage:
      persistentVolumeClaim:
        storageClassName: ocs-external-storagecluster-cephfs
  apicast:
    productionSpec:
      configurationLoader: lazy
      resources:
        requests:
          memory: 256Mi
          cpu: 100m
        limits:
          memory: 512Mi
          cpu: 500m
    stagingSpec:
      resources:
        requests:
          memory: 256Mi
          cpu: 100m
        limits:
          memory: 512Mi
          cpu: 500m
```

### 📌 Pontos importantes do manifesto

| Campo | Valor | Observação |
|-------|-------|------------|
| `wildcardDomain` | `apps.itz-ot6ru4...` | Deve corresponder ao domínio real do cluster |
| `storageClassName` | `ocs-external-storagecluster-cephfs` | CephFS — compatível com **ReadWriteMany (RWX)** |
| `configurationLoader` | `lazy` | APIcast carrega configuração sob demanda |
| `memory limit` | `512Mi` | Evita `OOMKilled` em ambientes com pouca RAM |

### ▶️ Aplicar o manifesto

```bash
oc apply -f apimanager.yml
```

---

## 👀 5. Acompanhar a Subida dos Pods

Monitore o estado dos pods em tempo real:

```bash
watch oc get pods -n 3scale
```

Ou de forma pontual:

```bash
oc get pods -n 3scale
```

### ✅ Estado esperado (todos `Running`)

```
NAME                               READY   STATUS    RESTARTS
apicast-production-xxx             1/1     Running   0
apicast-staging-xxx                1/1     Running   0
backend-cron-xxx                   1/1     Running   0
backend-listener-xxx               1/1     Running   0
backend-worker-xxx                 1/1     Running   0
system-app-xxx                     3/3     Running   0
system-memcache-xxx                1/1     Running   0
system-sidekiq-xxx                 1/1     Running   0
zync-xxx                           1/1     Running   0
zync-database-xxx                  1/1     Running   0
zync-que-xxx                       1/1     Running   0
```

---

## 💾 6. Storage — Problema e Solução

### 🔴 Erro encontrado

PVC `system-storage` em `Pending`

### 📌 Causa

StorageClass padrão incompatível com:

```yaml
accessModes:
  - ReadWriteMany
```

### ✅ Solução aplicada

Especificar explicitamente a StorageClass **CephFS** (compatível com RWX) no manifesto:

```yaml
system:
  fileStorage:
    persistentVolumeClaim:
      storageClassName: ocs-external-storagecluster-cephfs
```

> ⚠️ **RBD (ReadWriteOnce)** não é suportado pelo `system-storage` — use sempre CephFS ou equivalente RWX.

---

## 🧠 7. Problemas de Scheduling

### 🔴 Erro

```
0/6 nodes are available: Insufficient cpu
```

### ✅ Solução

- Migrar para cluster com mais recursos
- Ajustar `requests` e `limits` nos specs do APIManager (já aplicado no manifesto acima)

---

## 🧪 8. Problema de Image Pull Secret

### 🔴 Erro

```
Unable to retrieve image pull secrets
```

### ✅ Solução

- Validar o `pull-secret` do cluster
- Garantir acesso aos registries:

```
registry.redhat.io
registry.connect.redhat.com
```

---

## 💥 9. CrashLoopBackOff no APIcast

### 🔍 Sintomas

- Pods reiniciando continuamente
- Logs com:

```
missing configuration
failed to load configuration
```

### 🧠 Causa 1 — Falta de configuração publicada no System

APIcast depende de uma configuração publicada no portal 3scale.

#### ✅ Solução

- Acessar o portal admin
- Criar API/service
- Publicar a configuração

### 🧠 Causa 2 — Memória insuficiente (`OOMKilled`)

#### ✅ Ajuste aplicado no manifesto

```yaml
apicast:
  productionSpec:
    resources:
      requests:
        memory: 256Mi
        cpu: 100m
      limits:
        memory: 512Mi
        cpu: 500m
  stagingSpec:
    resources:
      requests:
        memory: 256Mi
        cpu: 100m
      limits:
        memory: 512Mi
        cpu: 500m
```

---

## 🔄 10. Reinicialização dos Pods

Caso necessário, forçar rollout manual:

```bash
oc rollout latest dc/apicast-production -n 3scale
oc rollout latest dc/apicast-staging -n 3scale
```

---

## 🌐 11. Checagem das Rotas

```bash
oc get routes -n 3scale
```

### 📋 Resultado obtido

| Nome          | URL                                         | Serviço            |
|---------------|---------------------------------------------|--------------------|
| backend       | `backend-3scale...`                         | backend-listener   |
| api (prod)    | `api-3scale-apicast-production...`          | apicast-production |
| api (staging) | `api-3scale-apicast-staging...`             | apicast-staging    |
| master        | `master.apps...`                            | system-master      |
| admin         | `3scale-admin...`                           | system-provider    |
| portal dev    | `3scale.apps...`                            | system-developer   |

---

## 🧭 12. Explicação das Rotas

### 🔹 `system-provider` — Admin Portal

```
https://3scale-admin.apps.itz-ot6ru4.infra01-lb.syd05.techzone.ibm.com
```

> Interface administrativa do 3scale — onde você cria APIs, aplicações e planos.

---

### 🔹 `system-developer` — Developer Portal

```
https://3scale.apps.itz-ot6ru4.infra01-lb.syd05.techzone.ibm.com
```

> Portal para desenvolvedores — cadastro de apps e consumo de APIs.

---

### 🔹 `system-master`

```
https://master.apps.itz-ot6ru4.infra01-lb.syd05.techzone.ibm.com
```

> API interna do 3scale — usada pelo APIcast para baixar configuração.

---

### 🔹 `apicast-production`

```
https://api-3scale-apicast-production.apps.itz-ot6ru4.infra01-lb.syd05.techzone.ibm.com
```

> Gateway de APIs em produção — tráfego real passa por aqui.

---

### 🔹 `apicast-staging`

```
https://api-3scale-apicast-staging.apps.itz-ot6ru4.infra01-lb.syd05.techzone.ibm.com
```

> Gateway de homologação — testes antes de publicar em produção.

---

### 🔹 `backend`

> Comunicação interna entre componentes — não exposto para uso externo direto.

---

## ✅ 13. Estado Final Esperado

- Todos os pods em `Running`
- APIcast sem `CrashLoopBackOff`
- PVCs todos em `Bound`
- Rotas acessíveis via browser
- Portal admin funcionando em `https://3scale-admin.apps...`
