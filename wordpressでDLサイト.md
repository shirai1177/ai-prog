# wordpressをインストール

nginx + wordpress のケース

## ファイルサイズ上限突破

/etc/php.ini に以下を設定
```
post_max_size = 30M
upload_max_filesize = 30M
max_execution_time = 300
```

/etc/nginx/nginx.conf に以下を設定
```
location ~ ^/wordpress/.*\.php {
  client_max_body_size 50m;        #<-------この行を追加
  fastcgi_pass 127.0.0.1:9000;
  fastcgi_index index.php;
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  include fastcgi_params;
 }
 ```

php-fpm と nginx を再起動


## プラグインのインストール

wordpressのプラグイン、download-monitorをインストールする。
ダウンロード－新規作成からファイルのアップロードを行なって、オブジェクトを作る
固定ページを使って、ダウンロードサイトを作る。
