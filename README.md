# apachc-php-fpm

- [docker php](https://hub.docker.com/_/php)
- [docker httpd](https://hub.docker.com/_/httpd)

同じソースコードを別バージョンのPHPで動作確認したいという要望に応えるための docker 構成サンプル。  
サンプルでは php7.x 系と 8.x 系の php-fpm イメージを apache2.4 の `VirtualHost` で構成したもの。  
webサーバにapacheを採用している理由は、長年蓄積したリライト設定などとアプリケーションの振舞の依存関係が深く、nginx に置き換えるのがツライからというだけの話。  

## httpd

設定ファイルは `/usr/local/apache2/conf` 内に配置されている。  
テスト用のコンテナなので頻繁に設定を変更することも多い事から、volume してシンボリックリンクを作る形にしてある。  
これによって、再ビルドではなくコンテナの restart で設定が変更される。

### モジュール

FastCGIを利用するには、以下のモジュールを有効にしておく必要がある。

- `mod_proxy`
- `mod_proxy_fcgi`

`httpd.conf` でコメント化されているので、それを有効にしておこう。  
その他必要なものがあれば、適宜コメントを有効化すればOKだろう。

### DocumentRoot

`VirtualHost` を利用するにあたり、グローバルな `DocumentRoot` は無効にするべきらしい。  

- [名前ベースのバーチャルホスト](https://httpd.apache.org/docs/2.4/vhosts/name-based.html)

ということで `httpd.conf` のデフォルト設定はコメントしておくべし。

### VirtualHost

`VirtualHost` の設定ファイルは `extra/httpd-vhosts.conf` に記載してある。  
`httpd.conf` 内で `Include` してる部分がコメント化されているので、それを有効にしておく。  
※ 別のファイル名でも良いし、直接記載しても問題ないけどな。

以下がサブドメインを利用して、バージョン違いのPHPを動作させる設定のサンプル。

```bash
<VirtualHost *:80>
    ServerAdmin root@localhost
    ServerName php8.docker.local
    ErrorLog /dev/stdout
    CustomLog /dev/stdout common
    DirectoryIndex index.php index.html
    DocumentRoot "/var/www/php8"
    <Directory "/var/www/php8">
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://php8:9000"
    </FilesMatch>
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin root@localhost
    ServerName php7.docker.local
    ErrorLog /dev/stdout
    CustomLog /dev/stdout common
    DirectoryIndex index.php index.html
    DocumentRoot "/var/www/php7"
    <Directory "/var/www/php7">
        Options FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://php7:9000"
    </FilesMatch>
</VirtualHost>
```

`DocumentRoot` を同じにするとヨロシクなさそうだったので、分けることを推奨。  
volume で同じリソースを参照させれば実質共有出来るし、そうしてある。  
dcoker-compose を利用してるので、サービス名で疎通が出来る。

ここで指定している `DocumentRoot` は php-fpm イメージ内でも同じパスにする必要があるようだ。  
`SetHandler` を利用することで、`PATH_INFO` などもうまく良くとの噂だが、また試していない。

## php-fpm

設定は `/usr/local/etc` に配置されている。  
`php.ini` をカスタマイズする場合は `/usr/lical/php/php.ini` に配置すればOKだ。  


