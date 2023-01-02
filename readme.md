# Magento Docker

## Uso

1 - Crie um arquivo chamado .env usando de base o arquivo .env.sample  

2 - Defina o usuário neste arquivo para o mesmo do seu sistema

3 - Defina a URL neste arquivo para o seu domínio

4 - Defina o path dos arquivos de SSL neste arquivo

5 - No arquivo docker-compose, comente o bloco do mysql caso não precise do banco de dados rodando local

6 - Prepare o ambiente executando: \
```
mkdir src && \
mkdir sessions && \
mkdir credentials && \
mkdir logs && \
touch logs/php-fpm-access.log && \
touch logs/php-fpm-error.log && \
sudo chown -R $USER: /etc/letsencrypt/live/
```

7 - `docker-compose up -d --build` para executar os containers

# Download Magento
1 - Busque as chaves de autenticação no [Magento Commerce](https://marketplace.magento.com/customer/accessKeys/) \
2 - Baixe a última versão do Magento \
```
docker exec -it magento_php composer create-project -vvv --repository-url=https://repo.magento.com/ magento/project-community-edition .
```
__Para instalar uma versão específica do magento, adicionar `:x.x.x` ao nome do repo. Exemplo:__
```
docker exec -it magento_php composer create-project -vvv --repository-url=https://repo.magento.com/ magento/project-community-edition:2.3.2 .
```
Se ocorrerem erros de permissão, execute: \
```
sudo chown -R $USER: src && \
sudo chown -R $USER: sessions && \
sudo chown -R $USER: credentials && \
sudo chown -R $USER: /etc/letsencrypt/live/
```

# Install Magento
Execute o comando abaixo, alterando os dados para o correto
```
docker exec -it magento_php php ./bin/magento setup:install \
    --base-url=https://ltucillo.ppolimpo.io \
    --db-host=mysqlhost.com \
    --db-name=dbname \
    --db-user=username \
    --db-password=dbpass \
    --db-prefix=245_ \
    --admin-firstname=Nome \
    --admin-lastname=Sobrenome \
    --admin-email=meu@email.com \
    --admin-user=adminuser \
    --admin-password=adminpass \
    --elasticsearch-host=127.0.0.1 \
    --elasticsearch-index-prefix=elprefix \
    --language=pt_BR \
    --currency=BRL \
    --timezone=America/Sao_Paulo \
    --use-rewrites=1 \
    --session-save=redis \
    --session-save-redis-host=redis \
    --session-save-redis-db=2
```

# Setting Redis as session storage
https://devdocs.magento.com/guides/v2.4/config-guide/redis/redis-session.html
docker exec -it magento_php bin/magento setup:config:set --session-save=redis --session-save-redis-host=redis --session-save-redis-log-level=4 --session-save-redis-db=2 \
    && docker exec -it magento_php ./bin/magento cache:clean


# Test Elasticsearch
curl -X GET "localhost:9200/_cat/nodes?v=true&pretty"

#/admin_qdl39h

# Disable 2FA in Admin
`docker exec magento_php bin/magento module:disable Magento_TwoFactorAuth && docker exec magento_php bin/magento cache:flush`

# RUN Mysql docker to connect client
`docker run --interactive --tty --rm mysql bash`

# Install MP plugin
```
composer require mercadopago/magento2-plugin && \
    bin/magento setup:upgrade && \
    bin/magento cache:clean
```

# Install SMTP extension
```bash
composer require mageplaza/module-smtp && \
    bin/magento setup:upgrade && \
    bin/magento cache:clean
```

# Install sample data
```bash
docker exec -it magento_php bin/magento sampledata:deploy && \
    docker exec -it magento_php bin/magento setup:upgrade && \
    docker exec -it magento_php bin/magento cache:clean
```
