#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

ARG LARADOCK_PHP_VERSION
FROM php:${LARADOCK_PHP_VERSION}-alpine

LABEL maintainer="Vladlen Beilik <vladlen.beilik@gmail.com>"

ARG LARADOCK_PHP_VERSION

RUN apk --update add wget \
  curl \
  git \
  build-base \
  libmcrypt-dev \
  libxml2-dev \
  linux-headers \
  pcre-dev \
  zlib-dev \
  autoconf \
  cyrus-sasl-dev \
  libgsasl-dev \
  oniguruma-dev \
  libressl \
  libressl-dev \
  supervisor


RUN pecl channel-update pecl.php.net; \
  docker-php-ext-install mysqli mbstring pdo pdo_mysql tokenizer xml pcntl

# Add a non-root user:
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN addgroup -g ${PGID} laradock && \
  adduser -D -G laradock -u ${PUID} laradock

#Install BZ2:
ARG INSTALL_BZ2=false
RUN if [ ${INSTALL_BZ2} = true ]; then \
  apk --update add bzip2-dev; \
  docker-php-ext-install bz2; \
  fi

###########################################################################
# PHP GnuPG:
###########################################################################

ARG INSTALL_GNUPG=false

RUN set -eux; if [ ${INSTALL_GNUPG} = true ]; then \
  apk add --no-cache --no-progress --virtual BUILD_DEPS_PHP_GNUPG gpgme-dev; \
  apk add --no-cache --no-progress gpgme; \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  pecl install gnupg-1.5.0RC2; \
  else \
  pecl install gnupg; \
  fi; \
  docker-php-ext-enable gnupg; \
  fi

#Install LDAP
ARG INSTALL_LDAP=false;
RUN set -eux; if [ ${INSTALL_LDAP} = true ]; then \
  apk add --no-cache --no-progress openldap-dev; \
  docker-php-ext-install ldap; \
  php -m | grep -oiE '^ldap$'; \
  fi

#Install GD package:
ARG INSTALL_GD=false
RUN if [ ${INSTALL_GD} = true ]; then \
  apk add --update --no-cache freetype-dev libjpeg-turbo-dev jpeg-dev libpng-dev; \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ] && [ $(php -r "echo PHP_MINOR_VERSION;") = "4" ]; then \
  docker-php-ext-configure gd --with-freetype --with-jpeg --with-png; \
  else \
  docker-php-ext-configure gd --with-freetype-dir=/usr/lib/ --with-jpeg-dir=/usr/lib/ --with-png-dir=/usr/lib/; \
  fi; \
  docker-php-ext-install gd \
  ;fi

#Install ImageMagick:
ARG INSTALL_IMAGEMAGICK=false
ARG IMAGEMAGICK_VERSION=latest
ENV IMAGEMAGICK_VERSION ${IMAGEMAGICK_VERSION}
RUN set -eux; \
  if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
  apk add --update --no-cache imagemagick-dev imagemagick; \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  cd /tmp && \
  if [ ${IMAGEMAGICK_VERSION} = "latest" ]; then \
  git clone https://github.com/Imagick/imagick; \
  else \
  git clone --branch ${IMAGEMAGICK_VERSION} https://github.com/Imagick/imagick; \
  fi && \
  cd imagick && \
  phpize && \
  ./configure && \
  make && \
  make install && \
  rm -r /tmp/imagick; \
  else \
  pecl install imagick; \
  fi && \
  docker-php-ext-enable imagick; \
  php -m | grep -q 'imagick'; \
  fi

#Install GMP package:
ARG INSTALL_GMP=false
RUN if [ ${INSTALL_GMP} = true ]; then \
  apk add --update --no-cache gmp gmp-dev \
  && docker-php-ext-install gmp \
  ;fi

#Install BCMath package:
ARG INSTALL_BCMATH=false
RUN if [ ${INSTALL_BCMATH} = true ]; then \
  docker-php-ext-install bcmath \
  ;fi

#Install SOAP package:
ARG INSTALL_SOAP=false
RUN if [ ${INSTALL_SOAP} = true ]; then \
  docker-php-ext-install soap \
  ;fi

###########################################################################
# PHP OCI8:
###########################################################################

ARG INSTALL_OCI8=false

ENV LD_LIBRARY_PATH="/usr/local/instantclient"
ENV ORACLE_HOME="/usr/local/instantclient"

RUN if [ ${INSTALL_OCI8} = true ] && [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
  apk add make php7-pear php7-dev gcc musl-dev libnsl libaio poppler-utils libzip-dev zip unzip libaio-dev freetds-dev && \
  ## Download and unarchive Instant Client v11
  curl -o /tmp/basic.zip https://raw.githubusercontent.com/bumpx/oracle-instantclient/master/instantclient-basic-linux.x64-11.2.0.4.0.zip && \
  curl -o /tmp/sdk.zip https://raw.githubusercontent.com/bumpx/oracle-instantclient/master/instantclient-sdk-linux.x64-11.2.0.4.0.zip && \
  curl -o /tmp/sqlplus.zip https://raw.githubusercontent.com/bumpx/oracle-instantclient/master/instantclient-sqlplus-linux.x64-11.2.0.4.0.zip && \
  unzip -d /usr/local/ /tmp/basic.zip && \
  unzip -d /usr/local/ /tmp/sdk.zip && \
  unzip -d /usr/local/ /tmp/sqlplus.zip \
  ## Links are required for older SDKs
  && ln -s /usr/local/instantclient_11_2 ${ORACLE_HOME} && \
  ln -s ${ORACLE_HOME}/libclntsh.so.* ${ORACLE_HOME}/libclntsh.so && \
  ln -s ${ORACLE_HOME}/libocci.so.* ${ORACLE_HOME}/libocci.so && \
  ln -s ${ORACLE_HOME}/lib* /usr/lib && \
  ln -s ${ORACLE_HOME}/sqlplus /usr/bin/sqlplus &&\
  ln -s /usr/lib/libnsl.so.2.0.0  /usr/lib/libnsl.so.1 && \
  ## Build OCI8 with PECL
  echo "instantclient,${ORACLE_HOME}" | pecl install oci8 && \
  echo 'extension=oci8.so' > /etc/php7/conf.d/30-oci8.ini \
  #  Clean up
  apk del php7-pear php7-dev gcc musl-dev && \
  rm -rf /tmp/*.zip /tmp/pear/ && \
  docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/usr/local/instantclient \
  && docker-php-ext-configure pdo_dblib --with-libdir=/lib \
  && docker-php-ext-install pdo_oci \
  && docker-php-ext-enable oci8 \
  && docker-php-ext-install zip && \
  # Install the zip extension
  docker-php-ext-configure zip && \
  php -m | grep -q 'zip' \
  ;fi

# Install ZipArchive:
ARG INSTALL_ZIP_ARCHIVE=false
RUN set -eux; \
  if [ ${INSTALL_ZIP_ARCHIVE} = true ]; then \
  apk --update add libzip-dev && \
  if [ ${LARADOCK_PHP_VERSION} = "7.3" ] || [ ${LARADOCK_PHP_VERSION} = "7.4" ] || [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  docker-php-ext-configure zip; \
  else \
  docker-php-ext-configure zip --with-libzip; \
  fi && \
  # Install the zip extension
  docker-php-ext-install zip \
  ;fi

# Install MySQL Client:
ARG INSTALL_MYSQL_CLIENT=false
RUN if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
  apk --update add mysql-client \
  ;fi

# Install FFMPEG:
ARG INSTALL_FFMPEG=false
RUN if [ ${INSTALL_FFMPEG} = true ]; then \
  apk --update add ffmpeg \
  ;fi

# Install BBC Audio Waveform Image Generator:
ARG INSTALL_AUDIOWAVEFORM=false
RUN if [ ${INSTALL_AUDIOWAVEFORM} = true ]; then \
  apk add git make cmake gcc g++ libmad-dev libid3tag-dev libsndfile-dev gd-dev boost-dev libgd libpng-dev zlib-dev \
  && apk add autoconf automake libtool gettext \
  && wget https://github.com/xiph/flac/archive/1.3.3.tar.gz \
  && tar xzf 1.3.3.tar.gz \
  && cd flac-1.3.3 \
  && ./autogen.sh \
  && ./configure --enable-shared=no \
  && make \
  && make install \
  && cd .. \
  && git clone https://github.com/bbc/audiowaveform.git \
  && cd audiowaveform \
  && wget https://github.com/google/googletest/archive/release-1.10.0.tar.gz \
  && tar xzf release-1.10.0.tar.gz \
  && ln -s googletest-release-1.10.0/googletest googletest \
  && ln -s googletest-release-1.10.0/googlemock googlemock \
  && mkdir build \
  && cd build \
  && cmake .. \
  && make \
  && make install \
  ;fi

ARG INSTALL_GHOSTSCRIPT=false
RUN if [ $INSTALL_GHOSTSCRIPT = true ]; then \
  apk --update add ghostscript \
  ;fi

# Install Redis package:
ARG INSTALL_REDIS=false
RUN if [ ${INSTALL_REDIS} = true ]; then \
  # Install Redis Extension
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
  printf "\n" | pecl install -o -f redis-4.3.0; \
  else \
  printf "\n" | pecl install -o -f redis; \
  fi; \
  rm -rf /tmp/pear; \
  docker-php-ext-enable redis \
  ;fi

###########################################################################
# Imap EXTENSION
###########################################################################

ARG INSTALL_IMAP=false

RUN if [ ${INSTALL_IMAP} = true ]; then \
  apk add --update imap-dev && \
  docker-php-ext-configure imap --with-imap --with-imap-ssl && \
  docker-php-ext-install imap \
  ;fi

###########################################################################
# XMLRPC:
###########################################################################

ARG INSTALL_XMLRPC=false

RUN if [ ${INSTALL_XMLRPC} = true ]; then \
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "8" ]; then \
  pecl install xmlrpc-1.0.0RC2; \
  docker-php-ext-enable xmlrpc; \
  else \
  docker-php-ext-install xmlrpc; \
  fi; \
  php -m | grep -r 'xmlrpc'; \
  fi

###########################################################################
# PHP Memcached:
###########################################################################

ARG INSTALL_MEMCACHED=false

RUN if [ ${INSTALL_MEMCACHED} = true ]; then \
  apk --update add libmemcached-dev; \
  # Install the php memcached extension
  if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
  pecl install memcached-2.2.0; \
  else \
  pecl install memcached; \
  fi; \
  docker-php-ext-enable memcached; \
  php -m | grep -r 'memcached'; \
  fi

#
#--------------------------------------------------------------------------
# Optional Supervisord Configuration
#--------------------------------------------------------------------------
#
# Modify the ./supervisor.conf file to match your App's requirements.
# Make sure you rebuild your container with every change.
#

COPY supervisord.conf /etc/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisord.conf"]

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# If you need to modify this image, feel free to do it right here.
#
# -- Your awesome modifications go here -- #

#
#--------------------------------------------------------------------------
# Check PHP version
#--------------------------------------------------------------------------
#

RUN php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

# Clean up
RUN rm /var/cache/apk/* \
  && mkdir -p /var/www

WORKDIR /etc/supervisor/conf.d/
