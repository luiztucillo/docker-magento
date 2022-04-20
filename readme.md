# Magento Docker

## Uso

1 - Crie um arquivo chamado .env usando de base o arquivo .env.sample  

2 - Defina o usuário neste arquivo para o mesmo do seu sistema

3 - Defina a URL neste arquivo para o seu domínio

4 - Defina o path dos arquivos de SSL neste arquivo

5 - No arquivo docker-compose, comente o bloco do mysql caso não precise do banco de dados rodando local

6 - `docker-compose up -d --build` para executar os containers

# Download Magento
1 - Busque as chaves de autenticação no [Magento Commerce](https://marketplace.magento.com/customer/accessKeys/)
2 - Execute `docker exec -it magento_php composer create-project -vvv --repository-url=https://repo.magento.com/ magento/project-community-edition .`

# Install Magento
Execute o comando abaixo, alterando os dados para o correto
```
docker exec -it magento_php php ./bin/magento setup:install \
    --base-url=$DOMAIN \
    --db-host=mysql.ppolimpo.io \
    --db-name=ltucillo_magento \
    --db-user=admin \
    --db-password=1234qwer \
    --admin-firstname=Luiz \
    --admin-lastname=Tucillo \
    --admin-email=luiz.tucillo@mercadolivre.com \
    --admin-user=admin \
    --admin-password=1234qwer \
    --elasticsearch-host=es1 \
    --language=pt_BR \
    --currency=BRL \
    --timezone=America/Sao_Paulo \
    --use-rewrites=1
```

# Test Elasticsearch
curl -X GET "localhost:9200/_cat/nodes?v=true&pretty"

#/admin_qdl39h

# Disable 2FA in Admin
`bin/magento module:disable Magento_TwoFactorAuth`
`bin/magento cache:flush`

# RUN Mysql docker to connect client
`docker run --interactive --tty --rm mysql bash`


