# Red Hat Build of Keycloak (RHBK) 26.x no OpenShift - Laboratório POC

Este repositório contém os guias e manifestos para a implementação do Red Hat Build of Keycloak (RHBK) 26.2 via Operator no OpenShift. O objetivo desta POC é configurar um ambiente com Alta Disponibilidade (HA), automação via CRDs e estratégia de Disaster Recovery (DR).

---

## 🏗️ Arquitetura da Solução

A implementação contempla os seguintes pilares:
* **Cluster RHBK:** Dois PODs operando em Alta Disponibilidade (HA) com compartilhamento de cache via Infinispan.
* **Banco de Dados:** Instância PostgreSQL para persistência de dados.
* **Infraestrutura como Código (IaC):** Utilização do CRD `KeycloakRealmImport` para provisionamento automático.
* **Disaster Recovery (DR):** Ambiente em modo Cold-Standby configurado em RHEL.

---

## 📂 Estrutura de Arquivos

| Arquivo | Função |
| :--- | :--- |
| `1-postgredb_all.yaml` | Manifestos de Secret, PVC, Service e Deployment do PostgreSQL. |
| `2-criar_certificado_keycloak-tls.txt` | Script para gerar certificados autoassinados e a Secret TLS no cluster. |
| `3-keycloak_Instance_CR.yaml` | Definição da instância do Keycloak (CR) integrada ao banco e TLS. |
| `4-acesso_admin.txt` | Comandos para obter a senha do `temp-admin` e criar o usuário administrador permanente. |
| `5-keycloak-realm-apps.yaml` | Importação do Realm "apps", incluindo o client `spring-artemis-producer-client`. |
| `6-consultar_token_realm-apps.txt` | Script de teste para validação de autenticação e obtenção de Access Token. |

---

## 🚀 Guia de Implementação

### 1. Preparação e Banco de Dados
Crie o namespace `rhbk` e implante o PostgreSQL utilizando o arquivo `1-postgredb_all.yaml`.

### 2. Segurança (TLS)
Gere o par de chaves e a secret para garantir a comunicação HTTPS. Os comandos estão disponíveis em `2-criar_certificado_keycloak-tls.txt`.

### 3. Deploy do Keycloak (HA)
Aplique o manifesto `3-keycloak_Instance_CR.yaml`. O Keycloak estará acessível via hostname configurado.

### 4. Gestão de Realm e Clientes
Para provisionar o Realm "apps" de forma automática, aplique o arquivo `5-keycloak-realm-apps.yaml`. Isso criará o cliente de serviço com suporte a `serviceAccountsEnabled`.

---

## 🛡️ Disaster Recovery (DR) - Cold-Standby

Para o cenário de DR em servidores RHEL:
* **Hardware:** Recomendado RHEL 8/9 com 8GB RAM e 4vCPU.
* **Estratégia:** Instalação onde Realms e usuários são sincronizados via banco de dados.
* **Configuração de Cache:** O arquivo `keycloak.conf` no DR deve ser ajustado com `cache=local`.
* **Ativação:** O serviço só deve ser iniciado em caso de failover real.

---

