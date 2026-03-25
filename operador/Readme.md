# ⚙️ 2. Instalação do Operator via OperatorHub

### 📌 Passo 1 — Acessar o OperatorHub

No console do OpenShift, navegue até **Operators → OperatorHub**.  
No campo de busca, pesquise por `3scale`.

> 🖼️ *Print: OperatorHub exibindo o card "3scale API Management" fornecido pela Red Hat — versão Community Operator `0.10.1`, channel `threescale-2.13`.*
![OperatorHub - 3scale](../images/01-OperatorHub%20com%20card%203scale.png)

---

### 📌 Passo 2 — Selecionar o Operator

Clique no card **3scale API Management**.  
O painel lateral exibirá as informações do operator:

- **Channel:** `threescale-2.13`
- **Version:** `0.10.1`
- **Provider:** Red Hat
- **Source:** Community
- **Capability level:** Basic Install, Seamless Upgrades, Full Lifecycle, Deep Insights

> ⚠️ **Nota:** Trata-se de um **Community Operator** — não verificado diretamente pela Red Hat. Use com atenção em produção.

Clique em **Install** para prosseguir.
![Selecionar Operador](../images/02-Painel%20do%20operator%20detalhes-Install.png)

---

### 📌 Passo 3 — Criar o Projeto `3scale`

Na tela de instalação do Operator, em **Installed Namespace**, clique em **Create Project**.

Preencha:

- **Name:** `3scale`

Clique em **Create**.

> 🖼️ *Print: Modal "Create Project" com o campo Name preenchido como `3scale`.*
![Modal "Create Project"](../images/03-Modal%20Create%20Project.png)

---

### 📌 Passo 4 — Configurar e instalar o Operator

Com o projeto criado, configure as opções de instalação:

| Campo               | Valor selecionado                     |
|---------------------|---------------------------------------|
| Update channel      | `threescale-2.13`                     |
| Version             | `0.10.1`                              |
| Installation mode   | All namespaces on the cluster         |
| Installed Namespace | `3scale`                              |
| Update approval     | Automatic                             |

> 🖼️ *Print: Tela "Install Operator" com as configurações acima preenchidas e namespace `3scale` selecionado.*
![Install Operator](../images/04-Tela%20Install%20Operator%20configurada.png)

Clique em **Install** para iniciar a instalação.

---

### 📌 Passo 5 — Confirmar o Operator instalado

Após a instalação, acesse **Operators → Installed Operators** e selecione o projeto `3scale`.

O operator **3scale API Management 0.10.1** deve aparecer como instalado.  
Clique nele e acesse a aba **APIManager** para criar o operand.

> 🖼️ *Print: Tela "Operator details" mostrando a aba APIManager com "No operands found" — aguardando a criação do APIManager.*
![Validação do Operador Instalado](../images/05-Operator%20instalado%20aba%20APIManager.png)

---

