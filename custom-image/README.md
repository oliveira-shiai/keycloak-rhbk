# RHBK - Imagem Customizada

Este repositório contém os arquivos necessários para criar uma imagem customizada do **Red Hat Build of Keycloak (RHBK)** e publicá-la no **OpenShift Internal Registry**.

## Pré-requisitos

- OpenShift CLI (`oc`)
- Podman
- Acesso ao cluster OpenShift
- Credenciais do Red Hat Customer Portal (ou Service Account)
- Projeto `rhbk` criado no OpenShift

---

# 1. Preparação do Ambiente

## Login no OpenShift

```bash
oc login -u kubeadmin -p <PASSWORD> https://<API_CLUSTER>:6443
oc project rhbk
```

> **Observação:** Substitua `<PASSWORD>` e `<API_CLUSTER>` pelos valores do seu ambiente.

---

## Habilitar a rota do Image Registry (caso ainda não esteja habilitada)

```bash
oc patch configs.imageregistry.operator.openshift.io/cluster \
  --patch '{"spec":{"defaultRoute":true}}' \
  --type=merge
```

---

## Autenticação nos registries

### Registry da Red Hat

Necessário para realizar o pull da imagem base do RHBK.

```bash
podman login registry.redhat.io
```

Utilize suas credenciais do **Red Hat Customer Portal** ou uma **Service Account**.

---

### Registry interno do OpenShift

Obtenha a URL do registry:

```bash
REGISTRY_URL=$(oc get route default-route \
  -n openshift-image-registry \
  --template='{{ .spec.host }}')
```

Faça o login utilizando o token da sessão atual:

```bash
podman login \
  -u kubeadmin \
  -p $(oc whoami -t) \
  $REGISTRY_URL
```

---

# 2. Criação dos Containerfiles

Crie os arquivos de build conforme o ambiente desejado:

- `Containerfile.dev`
- `Containerfile.prd`

Esses arquivos serão utilizados para gerar as imagens customizadas do RHBK.

---

# 3. Build e Push da Imagem

## Ambiente DEV/HML

### Build

```bash
podman build \
  -f Containerfile.dev \
  -t ${REGISTRY_URL}/rhbk/rhbk-custom:26.4.19-dev .
```

### Push

```bash
podman push \
  ${REGISTRY_URL}/rhbk/rhbk-custom:26.4.19-dev
```

---

## Ambiente PRD

### Build

```bash
podman build \
  -f Containerfile.prd \
  -t ${REGISTRY_URL}/rhbk/rhbk-custom:26.4.19-prd .
```

### Push

```bash
podman push \
  ${REGISTRY_URL}/rhbk/rhbk-custom:26.4.19-prd
```

---

# 4. Atualização da Custom Resource do Keycloak

Após publicar a imagem, atualize a CR do Keycloak para utilizar a nova versão.

Exemplo:

```yaml
spec:
  image: image-registry.openshift-image-registry.svc:5000/rhbk/rhbk-custom:26.4.19-dev

  additionalOptions:
    - name: metrics-enabled
      value: "true"
```

---

# Estrutura do Repositório

```text
.
├── Containerfile.dev
├── Containerfile.prd
└── README.md
```

---

# Fluxo Resumido

```text
Login no OpenShift
        │
        ▼
Login nos Registries
        │
        ▼
Criar Containerfile
        │
        ▼
Build da Imagem
        │
        ▼
Push para o Registry do OpenShift
        │
        ▼
Atualizar a CR do Keycloak
        │
        ▼
Operator realiza o rollout da nova imagem
```
