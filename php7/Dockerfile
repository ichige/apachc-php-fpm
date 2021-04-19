FROM php:7.4-fpm

ENV ORG_CONF_DIR=/usr/local/apache2/conf
ENV CUSTOM_CONF_DIR=/root/conf

# volume 内の設定ファイルをシンボリックリンクで構成する。
RUN ln -nfs ${CUSTOM_CONF_DIR}/httpd.conf ${ORG_CONF_DIR}/httpd/conf \
  && ln -nfs ${CUSTOM_CONF_DIR}/extra/httpd-vhosts.conf ${ORG_CONF_DIR}/extra/httpd-vhosts.conf ;