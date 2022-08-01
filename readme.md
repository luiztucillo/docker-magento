# Magento Docker

## Uso

1 - Crie um arquivo chamado .env usando de base o arquivo .env.sample  

2 - Defina o usuário neste arquivo para o mesmo do seu sistema

3 - Defina a URL neste arquivo para o seu domínio

4 - Defina o path dos arquivos de SSL neste arquivo

5 - No arquivo docker-compose, comente o bloco do mysql caso não precise do banco de dados rodando local

6 - `docker-compose up -d --build` para executar os containers

# Download Magento
1 - Busque as chaves de autenticação no [Magento Commerce](https://marketplace.magento.com/customer/accessKeys/) \
2 - Execute `docker exec -it magento_php composer create-project -vvv --repository-url=https://repo.magento.com/ magento/project-community-edition .` \
If you got permission error run outside docker but inside docker folder: `sudo chown -R $USER: src && sudo chown -R $USER: sessions && sudo chown -R $USER: credentials`
Also, run: `sudo chown -R $USER: /etc/letsencrypt/live/`

# Install Magento
Execute o comando abaixo, alterando os dados para o correto
```
docker exec -it magento_php php ./bin/magento setup:install \
    --base-url=https://ltucillo.ppolimpo.io \
    --db-host=mysqlhost.com \
    --db-name=dbname \
    --db-user=username \
    --db-password=dbpass \
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
`bin/magento module:disable Magento_TwoFactorAuth`
`bin/magento cache:flush`

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
bin/magento sampledata:deploy && \
    bin/magento setup:upgrade && \
    bin/magento cache:clean
