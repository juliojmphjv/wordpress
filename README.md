1. Criar os secrets
```sh
openssl rand -base64 20 | docker secret create wp_mysql_password -
openssl rand -base64 20 | docker secret create wp_mysql_root_password -
```

2. Deploy
```sh
docker stack deploy -c stack.yml wordpress
```

3. Acesse http://127.0.0.1:30000 e configure o wordpress

4. Alterar o password da base de dados

```sh
openssl rand -base64 20 | docker secret create wp_mysql_password_v2 -

docker service update --secret-rm wp_mysql_password wordpress_db

docker service update \
    --secret-add source=wp_mysql_password,target=old_wp_mysql_password \
    --secret-add source=wp_mysql_password_v2,target=wp_mysql_password \
    wordpress_db

docker container exec -it $(docker ps --filter name=db -q) bash
mysql -uroot -p"$(< /run/secrets/wp_mysql_root_password)"
GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%';
exit
exit

docker container exec $(docker ps --filter name=db -q) \
    bash -c 'mysqladmin --user=wordpress --password="$(< /run/secrets/old_wp_mysql_password)" password "$(< /run/secrets/wp_mysql_password)"'

docker service update \
     --secret-rm wp_mysql_password \
     --secret-add source=wp_mysql_password_v2,target=wp_db_password,mode=0400 \
    wordpress_wordpress

docker service update \
     --secret-rm wp_mysql_password \
    wordpress_db

docker secret rm wp_mysql_password
```

5. Finalizando o ambiente
```sh
docker stack rm wordpress
docker volume rm wordpress_mydata wordpress_wpdata
docker secret rm wp_mysql_password_v2 wp_mysql_root_password
```