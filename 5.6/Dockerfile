FROM centos/s2i-base-centos7

# This image provides an Apache+PHP environment for running PHP
# applications.

MAINTAINER Diego Hernandes <diego@hernandev.com>

EXPOSE 8000

ENV PHP_VERSION=7.0 \
    PATH=$PATH:/opt/rh/rh-php70/root/usr/bin

LABEL io.k8s.description="Nginx and PHP 7 (FPM)" \
      io.k8s.display-name="Nginx and PHP 7 (FPM)" \
      io.openshift.expose-services="8000:http" \
      io.openshift.tags="builder,nginx,php,php7,php70,php-fpm"

# Install Apache httpd and PHP
RUN curl 'https://setup.ius.io/' -o setup-ius.sh && \
    bash setup-ius.sh
RUN rm setup-ius.sh
RUN yum install -y centos-release-scl && \
    INSTALL_PKGS="rh-php56 rh-php56-php rh-php56-php-mysqlnd rh-php56-php-pgsql rh-php56-php-bcmath \
                  rh-php56-php-gd rh-php56-php-intl rh-php56-php-ldap rh-php56-php-mbstring rh-php56-php-pdo \
                  rh-php56-php-pecl-memcache rh-php56-php-process rh-php56-php-soap rh-php56-php-opcache rh-php56-php-xml \
                  rh-php56-php-pecl-xdebug rh-php56-php-gmp" && \
    yum install -y --setopt=tsflags=nodocs --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Each language image can have 'contrib' a directory with extra files needed to
# run and build the applications.
COPY ./contrib/ /opt/app-root

RUN cp /opt/app-root/etc/conf.d/php-fpm/pool.conf /etc/php-fpm.d/www.conf
RUN cp /opt/app-root/etc/conf.d/php-fpm/fpm.conf /etc/php-fpm.conf

# In order to drop the root user, we have to make some directories world
# writeable as OpenShift default security model is to run the container under
# random UID.
RUN mkdir /tmp/sessions && \
    mkdir -p /var/lib/nginx && \
    mkdir -p /var/log/nginx && \
    touch /var/log/nginx/error.log && \
    touch /var/log/nginx/access.log && \
    chown -R 1001:0 /var/log/nginx && \
    chown -R 1001:0 /var/lib/nginx && \
    chown -R 1001:0 /opt/app-root /tmp/sessions && \
    chmod -R a+rwx /tmp/sessions && \
    chmod -R a+rwx /var/log/nginx && \
    chmod -R a+rwx /var/lib/nginx && \
    chmod -R ug+rwx /var/log/nginx && \
    chmod -R ug+rwx /var/lib/nginx && \
    chmod -R ug+rwx /opt/app-root && \
    chmod +x /opt/app-root/services.sh

USER 1001

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/usage
