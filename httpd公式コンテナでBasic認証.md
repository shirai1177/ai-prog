
Basic認証をかけたいフォルダに.htaccessを作成する

```
# cat > .htaccess
AuthType Basic
AuthName "Basic Authentication"
AuthUserFile /usr/local/apache2/conf/.htpasswd
require valid-user
```

.htpasswdファイルをコンテナ内で作成する

```
# htpasswd -c /usr/local/apache2/conf/.htpasswd guest
```

httpd.conf を変更し、.htaccessを有効にする

```
# cd /usr/local/apache2/conf
# sed 's/AllowOverride None/AllowOverride All/g' httpd.conf > aaa
# mv httpd.conf httpd.conf.org
# mv aaa httpd.conf
```

コンテナを再起動する

```
# docker-compose restart
```
