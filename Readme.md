# 📘 Instalação do 3scale no OpenShift – Guia Completo


Este projeto tem como objetivo agilizar a disponibilização e configuração de ambientes do Red Hat 3scale API Management no OpenShift, padronizando etapas de instalação, configuração e troubleshooting.

A proposta é reduzir o tempo de setup, evitar erros comuns e fornecer um guia prático e reutilizável para implantação do 3scale em diferentes ambientes.


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

## ⚙️ 2. Itens para execução

- [Instalação do Operador](operador/)
- [API Manager](apimanager/)
- [Troubleshooting](ts/)

