# Magento Docker

## Uso

1. Crie um arquivo chamado .env usando de base o arquivo .env.sample  

2. Defina o usuário neste arquivo para o mesmo do seu sistema

3. Defina a URL neste arquivo para o seu domínio

4. Defina o path dos arquivos de SSL neste arquivo

5. No arquivo docker-compose, comente o bloco do mysql caso não precise do banco de dados rodando local

6. Prepare o ambiente executando: \
```
mkdir src && \
mkdir sessions && \
mkdir credentials && \
mkdir logs && \
touch logs/php-fpm-access.log && \
touch logs/php-fpm-error.log && \
sudo chown -R $USER: /etc/letsencrypt/live/
```

7. Execute os containers
```
docker-compose up -d --build
```

8. Pegue as informações de autenticação do repositório do Magento em [Magento Commerce](https://marketplace.magento.com/customer/accessKeys/)

9. Download Magento
```
docker exec -it magento_php composer create-project -vvv --repository-url=https://repo.magento.com/ magento/project-community-edition:2.4.5-p2 .
```

10. Instale o Magento \
> Se precisar configurar o mysql, execute:
> ```
> docker run --interactive --tty --rm mysql bash
> ```

```
docker exec -it magento_php php ./bin/magento setup:install \
    --base-url=https://ltucillo.ppolimpo.io \
    --db-host=mysqlhost.com \
    --db-name=dbname \
    --db-user=username \
    --db-password=dbpass \
    --db-prefix=m_ \
    --admin-firstname=Nome \
    --admin-lastname=Sobrenome \
    --admin-email=meu@email.com \
    --admin-user=admin \
    --admin-password=1234qwer \
    --backend-frontname=admin \
    --elasticsearch-host=172.31.17.80 \
    --elasticsearch-index-prefix=m_ \
    --language=pt_BR \
    --currency=BRL \
    --timezone=America/Sao_Paulo \
    --use-rewrites=1
```
#### No magento 2.4.6+ adicione:
```
--search-engine=elasticsearch7 \
--elasticsearch-enable-auth=false \
```

11. Atualize a URL do admin no arquivo `src/app/etc/env.php`

12. Desabilite o 2FA no Admin
#### Magento 2.4.5 -
```
docker exec magento_php bin/magento module:disable Magento_TwoFactorAuth && \
docker exec magento_php bin/magento cache:flush
```
#### Magento 2.4.6 +
```
docker exec magento_php bin/magento module:disable Magento_AdminAdobeImsTwoFactorAuth && \
docker exec magento_php bin/magento module:disable Magento_TwoFactorAuth && \
docker exec magento_php bin/magento cache:flush
```

13. Instale sample data
```bash
docker exec -it magento_php bin/magento sampledata:deploy && \
docker exec -it magento_php bin/magento setup:upgrade && \
docker exec -it magento_php bin/magento cache:clean
```

# Tradução
https://github.com/rafaelstz/traducao_magento2_pt_br

# Comandos Úteis
```bash
docker exec -it magento_php bin/magento setup:upgrade && \
docker exec -it magento_php rm -rf var/cache var/page_cache pub/static/frontend && \
docker exec -it magento_php bin/magento setup:di:compile && \
docker exec -it magento_php bin/magento setup:static:deploy --area frontend -f -j16 && \
docker exec -it magento_php bin/magento cache:clean
```

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

# Setting Redis as session storage
https://devdocs.magento.com/guides/v2.4/config-guide/redis/redis-session.html
```
docker exec -it magento_php bin/magento setup:config:set --session-save=redis --session-save-redis-host=redis --session-save-redis-log-level=4 --session-save-redis-db=2 \
    && docker exec -it magento_php ./bin/magento cache:clean
```

# Test Elasticsearch
```
curl -X GET "localhost:9200/_cat/nodes?v=true&pretty"
```

# RUN Mysql docker to connect client
```
docker run --interactive --tty --rm mysql bash
```

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
