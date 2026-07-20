# Backup e Restore do Banco de Dados PostgreSQL (RHBK)

Este documento descreve o procedimento para realizar **backup (dump)** e **restore** do banco de dados PostgreSQL utilizado pelo **Red Hat Build of Keycloak (RHBK)**.

---

## Backup do Banco de Dados (Dump)

Execute o comando abaixo para gerar um backup completo do banco e salvá-lo na sua máquina local.

```bash
oc exec postgredb-759cb7f55-x846j -n rhbk -- \
bash -c 'pg_dump -U $POSTGRESQL_USER -d $POSTGRESQL_DATABASE -F c' \
> backup-keycloakdb.dump
```

O arquivo `backup-keycloakdb.dump` será criado no diretório local onde o comando foi executado.

---

# Restore do Banco de Dados

## 1. Escalar o Keycloak para 0 réplicas

Antes do restore, interrompa o Keycloak para evitar conexões durante a restauração.

```bash
oc patch keycloak keycloak -n rhbk \
--type=merge \
-p '{"spec":{"instances":0}}'
```

Acompanhe até que o Pod seja removido:

```bash
oc get pods -n rhbk -w
```

---

## 2. Copiar o backup para o Pod do PostgreSQL

Obtenha o nome do Pod do banco:

```bash
POD_DB=$(oc get pods -n rhbk \
-l app=postgredb \
-o jsonpath='{.items[0].metadata.name}')
```

Copie o arquivo de backup para o diretório `/tmp` do Pod:

```bash
oc cp backup-keycloakdb.dump \
rhbk/${POD_DB}:/tmp/backup-keycloakdb.dump
```

---

## 3. Executar o Restore

Execute o restore dentro do Pod do PostgreSQL:

```bash
oc exec -it ${POD_DB} -n rhbk -- \
bash -c 'pg_restore -U $POSTGRESQL_USER \
-d $POSTGRESQL_DATABASE \
-1 \
-c \
/tmp/backup-keycloakdb.dump'
```

Após a conclusão, remova o arquivo temporário (opcional):

```bash
oc exec -it ${POD_DB} -n rhbk -- \
rm /tmp/backup-keycloakdb.dump
```

---

## 4. Escalar o Keycloak novamente

Após a restauração do banco, retorne o Keycloak para a quantidade desejada de réplicas.

```bash
oc patch keycloak keycloak -n rhbk \
--type=merge \
-p '{"spec":{"instances":1}}'
```

---

# Fluxo Resumido

```text
Backup
│
├── pg_dump
└── backup-keycloakdb.dump

Restore
│
├── Scale Down do Keycloak
├── Copiar backup para o Pod do PostgreSQL
├── Executar pg_restore
├── Remover arquivo temporário (opcional)
└── Scale Up do Keycloak
```
