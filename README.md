#  NextPark – Sprint 3 (Java Advanced + DevOps)


| Nome | RM | Função |
|------|----|--------|
| Raphaela Oliveira Tatto | RM554983 | 
| Tiago Ribeiro Capela | RM558021 | 

---


## Dificuladades no Deploy
Professor, boa noite.

Não conseguimos efetuar o deploy, apesar de termos tentado tanto em Java quanto em .NET.
Abaixo está o exemplo dos scripts que estávamos utilizando.

Acredito que a maior dificuldade que enfrentamos foi em relação ao banco de dados, pois nas aplicações em Java e .NET utilizamos o banco da FIAP, enquanto na sua matéria era necessário utilizar um banco na nuvem.

Chegamos a considerar duas alternativas:

- Manter o banco da FIAP para uso local e configurar o banco em nuvem apenas para produção;

- Ou remover completamente o banco da FIAP e utilizar somente o SQL Server em nuvem.

Infelizmente, ao tentar a segunda opção, a aplicação Java — que já estava funcionando corretamente — começou a apresentar erros, e o deploy também não concluía, ficando apenas no status “starting”.

Apesar disso, conseguimos criar os grupos de recursos, o banco de dados e o App Service no Azure, mas não conseguimos finalizar o deploy.

Pedimos desculpas por não ter sido possível concluir nesta sprint.
Na próxima entrega vamos nos preparar melhor e garantir que tudo esteja funcionando!

---
##  Scripts de Deploy – Azure CLI

```bash
az login

RESOURCE_GROUP=rg_sprint3
LOCATION=eastus2
GITHUB_REPO=raphatatto/Nextpark-sprint3-Java
BRANCH=main
DB_SERVER_NAME=sprint3-java
DB_NAME=sprint3
DB_USER=raphatatto
DB_PASS="SenhaForte123@"
APP_NAME=sprint3-java-app-unique

az group create --name $RESOURCE_GROUP --location $LOCATION

az provider register --namespace Microsoft.Sql
az provider register --namespace Microsoft.Web

az sql server create --resource-group $RESOURCE_GROUP --name $DB_SERVER_NAME --location $LOCATION     --admin-user $DB_USER --admin-password $DB_PASS --enable-public-network true

az sql db create --resource-group $RESOURCE_GROUP --server $DB_SERVER_NAME --name $DB_NAME     --backup-storage-redundancy Local --zone-redundant false

az sql server firewall-rule create --resource-group $RESOURCE_GROUP --server $DB_SERVER_NAME     --name AllowAzureIP --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

az appservice plan create --name $APP_NAME-plan --resource-group $RESOURCE_GROUP --sku F1 --is-linux

az webapp create --resource-group $RESOURCE_GROUP --plan $APP_NAME-plan --name $APP_NAME --runtime "JAVA|17-java17"

az resource update --resource-group $RESOURCE_GROUP --namespace Microsoft.Web   --resource-type basicPublishingCredentialsPolicies --name scm --parent sites/$APP_NAME --set properties.allow=true

az sql server firewall-rule create --resource-group $RESOURCE_GROUP --server $DB_SERVER_NAME   --name AllowAzureServices --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

az webapp config appsettings set   --resource-group $RESOURCE_GROUP   --name $APP_NAME   --settings     SPRING_DATASOURCE_USERNAME="$DB_USER@$DB_SERVER_NAME"     SPRING_DATASOURCE_PASSWORD="$DB_PASS"     SPRING_DATASOURCE_URL="jdbc:sqlserver://$DB_SERVER_NAME.database.windows.net:1433;database=$DB_NAME;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"

az webapp restart --name $APP_NAME --resource-group $RESOURCE_GROUP

az webapp deployment github-actions add --name $APP_NAME --resource-group $RESOURCE_GROUP --repo $GITHUB_REPO --branch $BRANCH --login-with-github

az webapp show --resource-group $RESOURCE_GROUP --name $APP_NAME
az sql server show --resource-group $RESOURCE_GROUP --name $DB_SERVER_NAME
```

---

##  Script DDL – Banco de Dados

Arquivo `script_bd.sql`:

```sql
CREATE TABLE USUARIOS (
    ID BIGINT IDENTITY(1,1) PRIMARY KEY,
    USERNAME NVARCHAR(80) NOT NULL UNIQUE,
    PASSWORD NVARCHAR(200) NOT NULL,
    ROLE NVARCHAR(20) NOT NULL CHECK (ROLE IN ('CLIENTE','GERENTE'))
);

CREATE TABLE VAGAS (
    ID BIGINT IDENTITY(1,1) PRIMARY KEY,
    CODIGO NVARCHAR(20) NOT NULL UNIQUE,
    STATUS NVARCHAR(10) NOT NULL CHECK (STATUS IN ('LIVRE','OCUPADA'))
);

CREATE TABLE MOTOS (
    ID BIGINT IDENTITY(1,1) PRIMARY KEY,
    PLACA NVARCHAR(10) NOT NULL UNIQUE,
    MODELO NVARCHAR(80) NOT NULL,
    STATUS NVARCHAR(15) NOT NULL CHECK (STATUS IN ('ALOCADA','MANUTENCAO','VISTORIA','DESALOCADA')),
    OWNER_USER_ID BIGINT NOT NULL,
    VAGA_ID BIGINT,
    CONSTRAINT FK_MOTOS_OWNER FOREIGN KEY (OWNER_USER_ID) REFERENCES USUARIOS(ID),
    CONSTRAINT FK_MOTOS_VAGA FOREIGN KEY (VAGA_ID) REFERENCES VAGAS(ID)
);

CREATE TABLE HIST_MOV (
    ID BIGINT IDENTITY(1,1) PRIMARY KEY,
    MOTO_ID BIGINT NOT NULL,
    ORIGEM_VAGA_ID BIGINT,
    DESTINO_VAGA_ID BIGINT,
    ACAO NVARCHAR(20) NOT NULL,
    USUARIO NVARCHAR(80) NOT NULL,
    CREATED_AT DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
    CONSTRAINT FK_HIST_MOTO FOREIGN KEY (MOTO_ID) REFERENCES MOTOS(ID) ON DELETE CASCADE,
    CONSTRAINT FK_HIST_ORIGEM FOREIGN KEY (ORIGEM_VAGA_ID) REFERENCES VAGAS(ID),
    CONSTRAINT FK_HIST_DEST FOREIGN KEY (DESTINO_VAGA_ID) REFERENCES VAGAS(ID)
);
```


![Texto alternativo](https://github.com/raphatatto/sprint-devops-Nextpark-3/blob/main/img1.png)
![Texto alternativo](https://github.com/raphatatto/sprint-devops-Nextpark-3/blob/main/img2.png)
![Texto alternativo](https://github.com/raphatatto/sprint-devops-Nextpark-3/blob/main/img3.png)

