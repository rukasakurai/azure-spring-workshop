# azure-spring-workshop
## 構成
- Compute: Azure App Serice
- Database: Azure Database for PostgreSQL server
- Code repository: GitHub
- CI/CD: GitHub Actions

https://docs.microsoft.com/en-us/learn/modules/deploy-java-spring-boot-app-service-mysql/5-exercise-deploy

## 前提条件
- Azureサブスクリプション上でのContributor権限
- Java JDK (1.8 以降)、Maven (3.0 以降)、Azure CLI (2.12 以降)、Bash、Curl のローカルでのインストール

### 参考: コンテンツ作成者の環境
- Windows 10
- Bashのターミナル:
  - WSL2 (確認方法: wsl -l -v)
　- GitBash (CurlのPOSTがWLS2だと実行できなかったので)
- java version "1.8.0_301" (確認方法: java -version)
- (確認方法: mvn -v)
- (確認方法: az -v)

# こちらのサンプルコードの展開方法
## レポジトリをFork
https://github.com/rukasakurai/azure-spring-workshop

## Azure上でのリソース作成
```
AZ_RESOURCE_GROUP=rg-springapp
AZ_LOCATION=japaneast
AZ_LOCAL_IP_ADDRESS=$(curl http://whatismyip.akamai.com/)
```
## App Service
Web Appを作成。Java
## DB
DBを作成。PostgreSQL or MySQL
### PostgreSQL
```
Creating PostgresSQL
https://docs.microsoft.com/en-us/azure/postgresql/quickstart-create-server-database-azure-cli

AZ_RESOURCE_GROUP=rg-20211001-temp
AZ_POSTGRE_NAME=postgresql-20211001-temp
AZ_LOCATION=japaneast
AZ_POSTGRESQL_USERNAME=spring
AZ_POSTGRESQL_PASSWORD=P@ssword123
AZ_LOCAL_IP_ADDRESS=217.178.23.128

az postgres server create --resource-group $AZ_RESOURCE_GROUP --name $AZ_POSTGRE_NAME  --location $AZ_LOCATION --admin-user $AZ_POSTGRESQL_USERNAME --admin-password $AZ_POSTGRESQL_PASSWORD --sku-name GP_Gen5_2

az postgres server firewall-rule create --resource-group $AZ_RESOURCE_GROUP --server $AZ_POSTGRE_NAME --name AllowMyIP --start-ip-address $AZ_LOCAL_IP_ADDRESS --end-ip-address $AZ_LOCAL_IP_ADDRESS

az postgres server show --resource-group $AZ_RESOURCE_GROUP --name $AZ_POSTGRE_NAME -o yaml

Alllow access to Azure services: Yes

psql --host=postgresql-20211001-temp.postgres.database.azure.com --port=5432 --username=spring@postgresql-20211001-temp --dbname=postgres


CREATE TABLE todo (
  id int, description varchar(255), details varchar(255), done bit, PRIMARY KEY (id)
);
```

### MySQL
```
AZ_DATABASE_NAME=db-20211001-temp
AZ_MYSQL_USERNAME=spring
AZ_MYSQL_PASSWORD=<Password>

az group create \
    --name $AZ_RESOURCE_GROUP \
    --location $AZ_LOCATION \
    | jq

az mysql server create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME \
    --location $AZ_LOCATION \
    --sku-name B_Gen5_1 \
    --storage-size 5120 \
    --admin-user $AZ_MYSQL_USERNAME \
    --admin-password $AZ_MYSQL_PASSWORD \
    | jq

az mysql server firewall-rule create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name $AZ_DATABASE_NAME-database-allow-local-ip \
    --server-name $AZ_DATABASE_NAME \
    --start-ip-address $AZ_LOCAL_IP_ADDRESS \
    --end-ip-address $AZ_LOCAL_IP_ADDRESS \
    | jq

az mysql server firewall-rule create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name allAzureIPs \
    --server-name $AZ_DATABASE_NAME \
    --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0 \
    | jq

az mysql db create \
    --resource-group $AZ_RESOURCE_GROUP \
    --name demo \
    --server-name $AZ_DATABASE_NAME \
    | jq

## GitHub上のコード変更
GitHub上のコードを変更すると、GitHub Actionsが走り、新しいコードが自動的に展開される

## アプ
### GET
http://<web app name>.azurewebsites.net/
### POST
curl --header "Content-Type: application/json" \
    --request POST \
    --data '{"description":"configuration","details":"congratulations, you have set up your Spring Boot application correctly!","done": "true"}' \
    https://<web app name>.azurewebsites.net

