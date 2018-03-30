FROM ubuntu:trusty
LABEL maintainer="Alexey Pustovalov <alexey.pustovalov@zabbix.com>"

ARG BUILD_DATE
ARG VCS_REF

ARG APT_FLAGS_COMMON="-qq -y"
ARG APT_FLAGS_PERSISTANT="${APT_FLAGS_COMMON} --no-install-recommends"
ARG APT_FLAGS_DEV="${APT_FLAGS_COMMON} --no-install-recommends"
ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en LC_ALL=en_US.UTF-8 DEBIAN_FRONTEND=noninteractive TERM=xterm \
    MIBDIRS=/var/lib/mibs/iana:/var/lib/mibs/ietf:/usr/share/snmp/mibs:/var/lib/zabbix/mibs MIBS=+ALL \
    ZBX_TYPE=server ZBX_DB_TYPE=mysql ZBX_OPT_TYPE=nginx \
    MYSQL_ALLOW_EMPTY_PASSWORD=true ZBX_ADD_SERVER=true ZBX_ADD_WEB=true DB_SERVER_HOST=localhost MYSQL_USER=zabbix ZBX_ADD_JAVA_GATEWAY=true ZBX_JAVAGATEWAY_ENABLE=true ZBX_JAVAGATEWAY=localhost

LABEL org.label-schema.name="zabbix-${ZBX_TYPE}-${ZBX_DB_TYPE}-ubuntu" \
      org.label-schema.vendor="Zabbix LLC" \
      org.label-schema.url="https://zabbix.com/" \
      org.label-schema.description="Zabbix appliance with MySQL database support and ${ZBX_OPT_TYPE} web-server" \
      org.label-schema.vcs-ref="${VCS_REF}" \
      org.label-schema.build-date="${BUILD_DATE}" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.license="GPL 2.0"

STOPSIGNAL SIGTERM

RUN locale-gen $LC_ALL && \
    echo "#!/bin/sh\nexit 0" > /usr/sbin/policy-rc.d && \
    addgroup --system --quiet zabbix && \
    adduser --quiet \
            --system --disabled-login \
            --ingroup zabbix \
            --home /var/lib/zabbix/ \
            --no-create-home \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /var/lib/zabbix && \
    mkdir -p /usr/lib/zabbix/alertscripts && \
    mkdir -p /var/lib/zabbix/enc && \
    mkdir -p /usr/lib/zabbix/externalscripts && \
    mkdir -p /var/lib/zabbix/mibs && \
    mkdir -p /var/lib/zabbix/modules && \
    mkdir -p /var/lib/zabbix/snmptraps && \
    mkdir -p /var/lib/zabbix/ssh_keys && \
    mkdir -p /var/lib/zabbix/ssl && \
    mkdir -p /var/lib/zabbix/ssl/certs && \
    mkdir -p /var/lib/zabbix/ssl/keys && \
    mkdir -p /var/lib/zabbix/ssl/ssl_ca && \
    chown --quiet -R zabbix:root /var/lib/zabbix && \
    mkdir -p /usr/share/doc/zabbix-${ZBX_TYPE}-${ZBX_DB_TYPE}/ && \
    apt-get ${APT_FLAGS_COMMON} update && \
    apt-get ${APT_FLAGS_COMMON} install \
            wget 1>/dev/null && \
    DISTRIB_CODENAME=$(/bin/bash -c 'source /etc/lsb-release && echo $DISTRIB_CODENAME') && \
    wget -q https://nginx.org/keys/nginx_signing.key -O- | apt-key add - && \
    echo "deb http://nginx.org/packages/ubuntu/ $DISTRIB_CODENAME nginx" >> /etc/apt/sources.list.d/nginx.list && \
    apt-get ${APT_FLAGS_COMMON} update && \
    apt-get ${APT_FLAGS_PERSISTANT} install \
            mysql-server \
            mysql-client \
            nginx \
            php5-fpm \
            php5-gd \
            php5-json \
            php5-ldap \
            php5-mysql \
            supervisor \
            ttf-dejavu-core \
            fping \
            libcurl3 \
            libiksemel3 \
            libmysqlclient18 \
            libopenipmi0 \
            libsnmp30 \
            libssh2-1 \
            libssl1.0.0 \
            libxml2 \
            mysql-client \
            snmp-mibs-downloader \
            unixodbc \
            openjdk-7-jre-headless 1>/dev/null && \
    apt-get ${APT_FLAGS_COMMON} purge \
            wget 1>/dev/null && \
    apt-get ${APT_FLAGS_COMMON} autoremove && \
    apt-get ${APT_FLAGS_COMMON} clean && \
    mkdir -p /var/lib/php5 && \
    chown --quiet -R www-data:www-data /var/lib/php5 && \
    rm -rf /var/cache/nginx/* && \
    rm -rf /var/lib/apt/lists/*

ARG MAJOR_VERSION=master
ARG ZBX_VERSION=${MAJOR_VERSION}
ARG ZBX_SOURCES=svn://svn.zabbix.com/tags/${ZBX_VERSION}/
ENV ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES}

LABEL org.label-schema.usage="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.label-schema.version="${ZBX_VERSION}" \
      org.label-schema.vcs-url="${ZBX_SOURCES}" \
      org.label-schema.docker.cmd="docker run --name zabbix -p 80:80 -p 10051:10051 -d zabbix:ubuntu-${ZBX_VERSION}"

COPY ["conf/tmp/font-config", "/tmp/font-config"]

RUN apt-get ${APT_FLAGS_COMMON} update && \
    apt-get ${APT_FLAGS_DEV} install \
            gettext \
            patch \
            subversion \
            autoconf \
            automake \
            gcc \
            libc6-dev \
            libcurl4-openssl-dev \
            libiksemel-dev \
            libldap2-dev \
            libmysqlclient-dev \
            libopenipmi-dev \
            libsnmp-dev \
            libssh2-1-dev \
            libxml2-dev \
            make \
            pkg-config \
            subversion \
            unixodbc-dev \
            autoconf \
            automake \
            libc6-dev \
            make \
            openjdk-7-jdk \
            pkg-config \
            subversion \
            gcc 1>/dev/null && \
    cd /tmp/ && \
    svn --quiet export ${ZBX_SOURCES} zabbix-${ZBX_VERSION} 1>/dev/null && \
    cd /tmp/zabbix-${ZBX_VERSION} && \
    zabbix_revision=`svn info ${ZBX_SOURCES} |grep "Last Changed Rev"|awk '{print $4;}'` && \
    sed -i "s/{ZABBIX_REVISION}/$zabbix_revision/g" include/version.h && \
    sed -i "s/{ZABBIX_REVISION}/$zabbix_revision/g" include/version.h && \
    sed -i "s/{ZABBIX_REVISION}/$zabbix_revision/g" src/zabbix_java/src/com/zabbix/gateway/GeneralInformation.java && \
    ./bootstrap.sh 1>/dev/null && \
    export CFLAGS="-fPIC -pie -Wl,-z,relro -Wl,-z,now" && \
    ./configure \
            --datadir=/usr/lib \
            --libdir=/usr/lib/zabbix \
            --prefix=/usr \
            --sysconfdir=/etc/zabbix \
            --enable-agent \
            --enable-${ZBX_TYPE} \
            --with-${ZBX_DB_TYPE} \
            --with-jabber \
            --with-ldap \
            --with-libcurl \
            --with-libxml2 \
            --enable-java \
            --with-net-snmp \
            --with-openipmi \
            --with-openssl \
            --with-ssh2 \
            --with-unixodbc \
            --enable-ipv6 \
            --silent && \
    make -j"$(nproc)" -s dbschema 1>/dev/null && \
    make -j"$(nproc)" -s 1>/dev/null && \
    cp src/zabbix_${ZBX_TYPE}/zabbix_${ZBX_TYPE} /usr/sbin/zabbix_${ZBX_TYPE} && \
    cp src/zabbix_get/zabbix_get /usr/bin/zabbix_get && \
    cp src/zabbix_sender/zabbix_sender /usr/bin/zabbix_sender && \
    cp conf/zabbix_${ZBX_TYPE}.conf /etc/zabbix/zabbix_${ZBX_TYPE}.conf && \
    chown --quiet -R zabbix:root /etc/zabbix && \
    cat database/${ZBX_DB_TYPE}/schema.sql > database/${ZBX_DB_TYPE}/create.sql && \
    cat database/${ZBX_DB_TYPE}/images.sql >> database/${ZBX_DB_TYPE}/create.sql && \
    cat database/${ZBX_DB_TYPE}/data.sql >> database/${ZBX_DB_TYPE}/create.sql && \
    gzip database/${ZBX_DB_TYPE}/create.sql && \
    cp database/${ZBX_DB_TYPE}/create.sql.gz /usr/share/doc/zabbix-${ZBX_TYPE}-${ZBX_DB_TYPE}/ && \
    mkdir -p /usr/sbin/zabbix_java/ && \
    cp -r src/zabbix_java/bin /usr/sbin/zabbix_java/ && \
    cp -r src/zabbix_java/lib /usr/sbin/zabbix_java/ && \
    rm -rf /usr/sbin/zabbix_java/lib/*.xml && \
    cd /tmp/ && \
    rm -rf /tmp/zabbix-${ZBX_VERSION}/ && \
    cd /usr/share/ && \
    svn --quiet export ${ZBX_SOURCES}/frontends/php/ zabbix 1>/dev/null && \
    cd /usr/share/zabbix/ && \
    patch -p3 < /tmp/font-config && \
    rm /tmp/font-config && \
    rm -f conf/zabbix.conf.php && \
    rm -rf tests && \
    rm /usr/share/zabbix/fonts/DejaVuSans.ttf && \
    ./locale/make_mo.sh 2>/dev/null && \
    update-alternatives --install /usr/share/zabbix/fonts/graphfont.ttf \
            zabbix-frontend-font /usr/share/fonts/truetype/ttf-dejavu/DejaVuSans.ttf 10 && \
    mkdir -p /var/lib/locales/supported.d/ && \
    rm -f /var/lib/locales/supported.d/local && \
    cat /usr/share/zabbix/include/locales.inc.php | grep display | grep true | awk '{$1=$1};1' | \
                cut -d"'" -f 2 | sort | \
                xargs -I '{}' bash -c 'echo "{}.UTF-8 UTF-8" >> /var/lib/locales/supported.d/local' && \
    dpkg-reconfigure locales 1>/dev/null && \
    find /usr/share/zabbix/locale -name '*.po' | xargs rm -f && \
    find /usr/share/zabbix/locale -name '*.sh' | xargs rm -f && \
    apt-get ${APT_FLAGS_COMMON} purge \
            gettext \
            patch \
            subversion \
            autoconf \
            automake \
            gcc \
            libc6-dev \
            libcurl4-openssl-dev \
            libiksemel-dev \
            libldap2-dev \
            libmysqlclient-dev \
            libopenipmi-dev \
            libsnmp-dev \
            libssh2-1-dev \
            libxml2-dev \
            make \
            pkg-config \
            subversion \
            unixodbc-dev \
            autoconf \
            automake \
            libc6-dev \
            make \
            openjdk-7-jdk \
            pkg-config \
            subversion \
            gcc 1>/dev/null && \
    apt-get ${APT_FLAGS_COMMON} autoremove && \
    apt-get ${APT_FLAGS_COMMON} clean && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /root/.subversion

EXPOSE 80/TCP 443/TCP 10051/TCP

WORKDIR /var/lib/zabbix

VOLUME ["/etc/ssl/nginx"]
VOLUME ["/usr/lib/zabbix/alertscripts", "/usr/lib/zabbix/externalscripts", "/var/lib/zabbix/enc", "/var/lib/zabbix/mibs", "/var/lib/zabbix/modules"]
VOLUME ["/var/lib/zabbix/snmptraps", "/var/lib/zabbix/ssh_keys", "/var/lib/zabbix/ssl/certs", "/var/lib/zabbix/ssl/keys", "/var/lib/zabbix/ssl/ssl_ca"]

COPY ["conf/etc/supervisor/", "/etc/supervisor/"]
COPY ["conf/etc/zabbix/nginx.conf", "/etc/zabbix/"]
COPY ["conf/etc/zabbix/nginx_ssl.conf", "/etc/zabbix/"]
COPY ["conf/etc/zabbix/web/zabbix.conf.php", "/etc/zabbix/web/"]
COPY ["conf/etc/nginx/nginx.conf", "/etc/nginx/"]
COPY ["conf/etc/php5/fpm/conf.d/99-zabbix.ini", "/etc/php5/fpm/conf.d/"]
COPY ["conf/etc/zabbix/zabbix_java_gateway_logback.xml", "/etc/zabbix/"]
COPY ["conf/usr/sbin/zabbix_java_gateway", "/usr/sbin/"]
COPY ["docker-entrypoint.sh", "/usr/bin/"]

ENV ZBX_TYPE=appliance

ENTRYPOINT ["docker-entrypoint.sh"]