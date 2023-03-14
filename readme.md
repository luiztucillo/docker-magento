# Magento Docker

## Uso

1 - Crie um arquivo chamado .env usando de base o arquivo .env.sample

2 - Defina o usuário neste arquivo para o mesmo do seu sistema

3 - Defina a URL neste arquivo para o domínio locao

6 - Prepare o ambiente executando:
```
mkdir src && \
chmod -R 777 src && \
mkdir sessions && \
chmod -R 777 sessions && \
mkdir credentials && \
chmod -R 777 credentials && \
mkdir logs && \
chmod -R 777 logs && \
touch logs/php-fpm-access.log && \
touch logs/php-fpm-error.log
```

7 - `docker-compose up -d --build` para executar os containers

8 - Faça o download do Magento

9 - Instale o Magento

# Download Magento
1 - Busque as chaves de autenticação no [Magento Commerce](https://marketplace.magento.com/customer/accessKeys/) \
2 - Baixe o Magento \
```bash

docker exec -it magento_php composer create-project -vvv --repository-url=https://repo.magento.com/ magento/project-community-edition .
```
__Para instalar uma versão específica do magento, adicionar `:x.x.x` ao nome do repo.__
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
    --search-engine=elasticsearch7 \
    --elasticsearch-host=elasticsearch \
    --elasticsearch-port=9200 \
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
docker exec magento_php bin/magento module:disable Magento_AdminAdobeImsTwoFactorAuth && \
    docker exec magento_php bin/magento module:disable Magento_TwoFactorAuth && \
    docker exec magento_php bin/magento cache:flush
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

# Utils
## Set developer mode
```bash
rm -rf var/di/* var/generation/* && \
docker exec -it magento_php bin/magento deploy:mode:set developer
```
## Clean all
```bash
docker exec -it magento_php bin/magento setup:upgrade && \
docker exec -it magento_php rm -rf var/cache var/page_cache pub/static/frontend && \
docker exec -it magento_php bin/magento setup:di:compile && \
docker exec -it magento_php bin/magento setup:static:deploy --area frontend -f -j16 && \
docker exec -it magento_php bin/magento cache:clean
```

# Desenvolvimento do plugin fora do docker
1 - Faça o clone do plugin em algum diretório na máquina \
2 - No `docker-compose.yml`, configure o volume do plugin para apontar para interno do docker, adicionando o caminho dentro do container `php`, na seção `volumes`
```yml
php:
...
volumes:
      ...
      - ./payment-magento-plugin:/var/www/html/app/code/MercadoPago/PaymentMagento
      ...
...
```
O formato é: `-[caminho_do_diretório_local]:[caminho_dentro_do_docker]`
## Para o antigo plugin, mapear: \
`- ./[caminho_local]/cart-magento2:/var/www/html/vendor/mercadopago/magento2-plugin` \
## Para o novo plugin, mapear: \
`- ./[caminho_local]:/var/www/html/app/code/MercadoPago/PaymentMagento`
3 - Reinicie o docker com `docker-compose down && docker-compose up -d --build`

# Debugging
### VSCODE
1 - Instalar a extensão oficial do xdebug após conectar na máquina remota. Buscar por: `xdebug.php-debug`\
2 - No menu superior, clique em `Run` depois `Add configuration` e selecione `PHP`\
3 - Crie a seguinte configuração no json que abre:
```
{
  "name": "Listen for Magento Remote Debug",
  "type": "php",
  "request": "launch",
  "port": 9003,
  "pathMappings": {
      "/var/www/html": "${workspaceRoot}/src",
  },
  "log": true
},
```
4 - Coloque o caminho específico do plugin no `pathMappings` \
Ex.: \
   ```json
   "pathMappings": {
      "/var/www/html": "${workspaceRoot}/src",
      "/var/www/html/app/code/MercadoPago/PaymentMagento": "${workspaceRoot}/payment-magento-plugin"
   },
   ```
5 - Na aba `Run and Debug` clique em `Run`
