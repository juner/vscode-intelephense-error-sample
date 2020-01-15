FROM centos:7

# 日本語表示を正しくする (lessとRuby向け)
ENV LESSCHARSET="utf-8"
ENV LANG="en_US.UTF-8"

# cakephp
ENV APP_ENCODING="UTF-8"
ENV APP_DEFAULT_LOCALE="ja_JP"
ENV APP_DEFAULT_TIMEZONE="Asia/Tokyo"

RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 && \
    yum -y update
# apache の最新をインストール
# https://iwatani.tv/linux-server/centos7-apache/
RUN yum install -y epel-release && \
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7 && \
    yum -y install https://centos7.iuscommunity.org/ius-release.rpm && \
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-IUS-7 && \
    sed -i -e "s/enabled *= *1/enabled=0/g" /etc/yum.repos.d/ius.repo && \
    sed -i -e "s/enabled *= *1/enabled=0/g" /etc/yum.repos.d/epel.repo
RUN yum --enablerepo=epel -y install brotli nghttp2 && \
    yum -y install epel-release && \
    yum -y install mailcap system-logos && \
    yum -y install openssl && \
    yum -y --disablerepo=base,extras,updates --enablerepo=ius install httpd && \
    yum -y --disablerepo=base,extras,updates --enablerepo=ius install mod_ssl && \
    yum clean all
# prefork -> event
RUN sed -i -e "s|LoadModule mpm_prefork_module modules/mod_mpm_prefork.so|#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so|" /etc/httpd/conf.modules.d/00-mpm.conf && \
    sed -i -e "s|#LoadModule mpm_event_module modules/mod_mpm_event.so|LoadModule mpm_event_module modules/mod_mpm_event.so|" /etc/httpd/conf.modules.d/00-mpm.conf

# view apach version info
RUN apachectl -V

# make SSL server certificate
RUN mkdir -p /etc/ssl/private && \
   openssl req -new -newkey rsa:2048 -nodes -out /etc/ssl/private/server.csr -keyout /etc/ssl/private/server.key -subj "/C=/ST=/L=/O=/OU=/CN=*.lvh.me" && \
   openssl x509 -days 3650 -req -signkey /etc/ssl/private/server.key -in /etc/ssl/private/server.csr -out /etc/ssl/private/server.crt

# 外部リポジトリ（Remi）を追加
RUN yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm && \
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-remi && \
    # apache その他 phpパッケージを導入
    yum -y install --enablerepo=remi,remi-php74 php php-cli php-common php-devel php-fpm php-gd php-mbstring php-pdo php-pear php-pgsql php-pecl-apcu php-pecl-uuid php-xml php-xmlrpc php-intl php-xdebug && \
    yum -y install zip unzip

# Xdebug Settings
RUN sed -i -e "s|;xdebug.auto_trace = false|xdebug.auto_trace = true|" /etc/php.d/15-xdebug.ini && \
    sed -i -e "s|;xdebug.auto_trace = false|xdebug.auto_trace = true|" /etc/php-zts.d/15-xdebug.ini && \
    sed -i -e "s|;xdebug.remote_enable = false|xdebug.remote_enable = true|" /etc/php.d/15-xdebug.ini && \
    sed -i -e "s|;xdebug.remote_enable = false|xdebug.remote_enable = true|" /etc/php-zts.d/15-xdebug.ini && \
    sed -i -e "s|;xdebug.remote_autostart = false|xdebug.remote_autostart = true|" /etc/php.d/15-xdebug.ini && \
    sed -i -e "s|;xdebug.remote_autostart = false|xdebug.remote_autostart = true|" /etc/php-zts.d/15-xdebug.ini

# git のインストール
RUN yum install -y --enablerepo=ius git2u

# composerのインストール
RUN curl -sS https://getcomposer.org/installer | php && \
    mv composer.phar /usr/local/bin/composer
RUN composer global require jonjomckay/php-pecl-uuid-stubs
WORKDIR /var/www/html

CMD ["/usr/sbin/httpd","-D","FOREGROUND"]
