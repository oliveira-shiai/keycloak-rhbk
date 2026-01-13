# Red Hat Build of Keycloak (RHBK) 26.2 no OpenShift - Laboratório POC

[cite_start]Este repositório contém os guias e manifestos para a implementação do **Red Hat Build of Keycloak (RHBK) 26.2** via Operator no OpenShift[cite: 12]. [cite_start]O objetivo desta POC é configurar um ambiente com Alta Disponibilidade (HA), automação via CRDs e estratégia de Disaster Recovery (DR)[cite: 13, 14].

---

## 🏗️ Arquitetura da Solução

[cite_start]A implementação contempla os seguintes pilares[cite: 14]:
* [cite_start]**Cluster RHBK:** Dois PODs operando em Alta Disponibilidade (HA) com compartilhamento de cache via Infinispan[cite: 15, 54].
* [cite_start]**Banco de Dados:** Instância PostgreSQL para persistência de dados[cite: 17, 36].
* [cite_start]**Infraestrutura como Código (IaC):** Utilização do CRD `KeycloakRealmImport` para provisionamento automático[cite: 16, 92].
* [cite_start]**Disaster Recovery (DR):** Ambiente em modo Cold-Standby configurado em RHEL[cite: 18, 150].

---

## 📂 Estrutura de Arquivos

| Arquivo | Função |
| :--- | :--- |
| `1-postgredb_all.yaml` | [cite_start]Manifestos de Secret, PVC, Service e Deployment do PostgreSQL[cite: 37, 42]. |
| `2-criar_certificado_keycloak-tls.txt` | [cite_start]Script para gerar certificados autoassinados e a Secret TLS no cluster[cite: 49]. |
| `3-keycloak_Instance_CR.yaml` | [cite_start]Definição da instância do Keycloak (CR) integrada ao banco e TLS[cite: 51, 52]. |
| `4-acesso_admin.txt` | [cite_start]Comandos para obter a senha do `temp-admin` e criar o usuário administrador permanente[cite: 78, 83]. |
| `5-keycloak-realm-apps.yaml` | [cite_start]Importação do Realm "apps", incluindo o client `spring-artemis-producer-client`[cite: 93, 94]. |
| `6-consultar_token_realm-apps.txt` | [cite_start]Script de teste para validação de autenticação e obtenção de Access Token[cite: 129, 130]. |

---

## 🚀 Guia de Implementação

### 1. Preparação e Banco de Dados
[cite_start]Crie o namespace `rhbk` e implante o PostgreSQL utilizando o arquivo `1-postgredb_all.yaml`[cite: 23, 42].

### 2. Segurança (TLS)
[cite_start]Gere o par de chaves e a secret para garantir a comunicação HTTPS[cite: 48]. [cite_start]Os comandos estão disponíveis em `2-criar_certificado_keycloak-tls.txt`[cite: 49].

### 3. Deploy do Keycloak (HA)
[cite_start]Aplique o manifesto `3-keycloak_Instance_CR.yaml`[cite: 52]. [cite_start]Aguarde até que os 2 pods estejam no estado `Running` e `Ready`[cite: 53]. [cite_start]O Keycloak estará acessível via hostname configurado[cite: 190, 191].

### 4. Gestão de Realm e Clientes
[cite_start]Para provisionar o Realm "apps" de forma automática, aplique o arquivo `5-keycloak-realm-apps.yaml`[cite: 94]. [cite_start]Isso criará o cliente de serviço com suporte a `serviceAccountsEnabled`[cite: 227].

---

## 🛡️ Disaster Recovery (DR) - Cold-Standby

[cite_start]Para o cenário de DR em servidores RHEL[cite: 149]:
* [cite_start]**Hardware:** Recomendado RHEL 8/9 com 8GB RAM e 4vCPU[cite: 156, 158].
* [cite_start]**Estratégia:** Instalação estrutural onde Realms e usuários são sincronizados via banco de dados[cite: 151].
* [cite_start]**Configuração de Cache:** O arquivo `keycloak.conf` no DR deve ser ajustado com `cache=local`[cite: 165, 194].
* [cite_start]**Ativação:** O serviço só deve ser iniciado (`kc.sh start --optimized`) em caso de failover real[cite: 201, 202].

---

## 🔗 Links de Referência
* [cite_start][Guia do Operator RHBK 26.2](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/26.2/html-single/operator_guide/index)[cite: 23, 45].
* [cite_start][Configuração de Banco de Dados](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/26.2/html-single/server_configuration_guide/index#db-installing-the-oracle-database-driver)[cite: 46, 207].

---
[cite_start]**Elaborado por:** Joel Oliveira[cite: 5, 214].
