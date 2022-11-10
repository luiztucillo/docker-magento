# Magento Docker

## Uso

1 - Crie um arquivo chamado .env usando de base o arquivo .env.sample

2 - Defina o usuário neste arquivo para o mesmo do seu sistema

3 - Defina a URL neste arquivo para o domínio locao

4 - No arquivo docker-compose, comente a linha 47 que aponta para o plugin do Mercado Pago

5 - Prepare o ambiente executando:
```
mkdir src && \
mkdir sessions && \
mkdir credentials && \
mkdir logs && \
touch logs/php-fpm-access.log && \
touch logs/php-fpm-error.log
```

6 - `docker-compose up -d --build` para executar os containers

7 - Faça o download do Magento

8 - Instale o Magento

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
sudo chmod -R 777 logs && \
sudo chmod -R 777 sessions && \
sudo chmod -R 777 src && \
sudo chmod -R 777 credentials && \
sudo chmod -R 777 database
```

# Install Magento
Execute o comando abaixo, alterando os dados para o correto
```
docker exec -it magento_php php ./bin/magento setup:install \
    --base-url=https://magento.local \
    --db-host=mysql \
    --db-name=magento \
    --db-user=root \
    --db-password=1234qwer \
    --db-prefix=mg_ \
    --admin-firstname=Nome \
    --admin-lastname=Sobrenome \
    --admin-email=meu@email.com \
    --admin-user=admin \
    --admin-password=1234qwer \
    --elasticsearch-host=elasticsearch \
    --language=pt_BR \
    --currency=BRL \
    --timezone=America/Sao_Paulo \
    --use-rewrites=1 \
    --session-save=redis \
    --session-save-redis-host=redis \
    --session-save-redis-db=2
```
# Test Elasticsearch
```
curl -X GET "localhost:9200/_cat/nodes?v=true&pretty"
```

# Disable 2FA in Admin
```
docker exec magento_php bin/magento module:disable Magento_TwoFactorAuth && docker exec magento_php bin/magento cache:flush
```

# RUN Mysql docker to connect client
```
docker run --interactive --tty --rm --network="magento_network" mysql bash
```

# Install MP plugin
```
composer require mercadopago/magento2-plugin && \
    bin/magento setup:upgrade && \
    bin/magento cache:clean
```

# Install SMTP extension
```bash
docker exec -it magento_php composer require mageplaza/module-smtp && \
    docker exec -it magento_php bin/magento setup:upgrade && \
    docker exec -it magento_php bin/magento cache:clean
```

# Install sample data
```
docker exec -it magento_php bin/magento sampledata:deploy && \
    docker exec -it magento_php bin/magento setup:upgrade && \
    docker exec -it magento_php bin/magento cache:clean
```

# XDEBUG
Após instalar o plugin do Mercado Pago no padrão composer, configure o xdebug conforme abaixo.
A pasta relativa ao módulo do Mercado Pago deve ser ajustada para onde clonou o repositório.
## Path mappings
```
{
  "name": "Listen for Magento Docker XDebug",
  "type": "php",
  "request": "launch",
  "port": 9003,
  "pathMappings": {
      "/var/www/html/vendor/mercadopago/magento2-plugin/src/MercadoPago": "${workspaceRoot}/src/vendor/mercadopago/magento2-plugin/src/MercadoPago",
      "/var/www/html/vendor/magento": "${workspaceRoot}/src/vendor/magento",
      "/var/www/html/pub/index.php": "${workspaceRoot}/src/pub/index.php"
  },
  "log": true
}
```
