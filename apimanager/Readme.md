# Configuração e Apply do APIManager

## 🔐 1. Login via CLI (`oc`)

Alterne para o terminal e faça login no cluster usando o token de acesso:

```bash
oc login \
  --token=XXX \
  --server=https://XXX:6443
```

Selecione o projeto `3scale`:

```bash
oc project 3scale
```

---

## 🧩 2. Configuração e Apply do APIManager

### 📄 Arquivo `apimanager.yml`

```yaml
kind: APIManager
apiVersion: apps.3scale.net/v1alpha1
metadata:
  name: apimanager-sample
  namespace: 3scale
spec:
  wildcardDomain: apps.XXX
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

