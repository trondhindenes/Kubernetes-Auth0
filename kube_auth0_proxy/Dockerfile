FROM ubuntu:latest
MAINTAINER Trond Hindenes <trond@hindenes.com>
RUN apt-get update
RUN apt-get install -y curl apache2 libapache2-mod-auth-openidc python
ENV DUMB_INIT_VERSION=1.2.0
RUN \
    curl -Ls https://github.com/Yelp/dumb-init/releases/download/v${DUMB_INIT_VERSION}/dumb-init_${DUMB_INIT_VERSION}_amd64.deb > dumb-init.deb &&\
    dpkg -i dumb-init.deb &&\
    rm dumb-init.deb
ENV LANG='en_US.UTF-8' LC_ALL='en_US.UTF-8'
#COPY auth_openidc.conf /etc/apache2/mods-available/auth_openidc.conf
RUN a2enmod auth_openidc
RUN a2enmod ssl
RUN a2enmod proxy
RUN a2enmod proxy_html
RUN a2enmod proxy_http
RUN a2enmod headers
RUN a2enmod xml2enc
RUN service apache2 stop
EXPOSE 80
WORKDIR /home
#The next 2 lines just creates a self-signed cert to run apache on
COPY sslgen.sh /home/sslgen.sh
RUN chmod +x sslgen.sh
RUN ./sslgen.sh;exit 0
COPY configure_and_run.py /home/configure_and_run.py
EXPOSE 443
#COPY 000-default.conf /etc/apache2/sites-enabled/000-default.conf
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["bash", "-c", "./configure_and_run.py && exec /usr/sbin/apachectl -e info -DFOREGROUND"]