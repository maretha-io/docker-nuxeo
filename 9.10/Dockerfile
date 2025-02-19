# vim:set ft=dockerfile:
FROM       openjdk:8-jdk
MAINTAINER Nuxeo <packagers@nuxeo.com>

# Add needed convert tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    perl \
    locales \
    pwgen \
    imagemagick \
    ffmpeg2theora \
    poppler-utils \
    libwpd-tools \
    exiftool \
    ffmpeg \
    x264 \
 && rm -rf /var/lib/apt/lists/*


# Remove setuid/setgid binaries from images for security
RUN find / -perm 6000 -type f -exec chmod a-s {} \; || true

ENV NUXEO_USER nuxeo
ENV NUXEO_HOME /opt/nuxeo/server
ENV HOME /opt/nuxeo/server
ARG NUXEO_VERSION=9.10
ARG NUXEO_DIST_URL=http://community.nuxeo.com/static/releases/nuxeo-9.10/nuxeo-server-9.10-tomcat.zip
ARG NUXEO_MD5=327d23bbd5558565694027b11c0dd82a


# Create Nuxeo user
RUN useradd -m -d /home/$NUXEO_USER -u 1000 -s /bin/bash $NUXEO_USER

RUN chown -R $NUXEO_USER:$NUXEO_USER /opt
RUN whoami
RUN pwd


# Add distribution
RUN curl -fsSL "${NUXEO_DIST_URL}" -o /nuxeo-distribution-tomcat.zip 
RUN mkdir  /nuxeo-distribution
RUN mkdir -p /nuxeo-distribution $(dirname $NUXEO_HOME) 
RUN chmod -R 757 /nuxeo-distribution
RUN unzip -q -d /nuxeo-distribution /nuxeo-distribution-tomcat.zip \
    && DISTDIR=$(/bin/ls /nuxeo-distribution | head -n 1) \
    && mv /nuxeo-distribution/$DISTDIR $NUXEO_HOME \
    && sed -i -e "s/^org.nuxeo.distribution.package.*/org.nuxeo.distribution.package=docker/" $NUXEO_HOME/templates/common/config/distribution.properties \
    && chmod +x $NUXEO_HOME/bin/*ctl $NUXEO_HOME/bin/*.sh \
    # For arbitrary user in the admin group
    && chmod g+rwX $NUXEO_HOME/bin/*ctl $NUXEO_HOME/bin/*.sh \
    && $NUXEO_HOME/bin/nuxeoctl mp-init \
    && chown -R 1000:0 $NUXEO_HOME && chmod -R g+rwX $NUXEO_HOME

COPY docker-template $NUXEO_HOME/templates/docker
COPY nuxeo.conf /etc/nuxeo/nuxeo.conf.template
ENV NUXEO_CONF /etc/nuxeo/nuxeo.conf

# Protecting writeable directories to support arbitrary User IDs for OpenShift
# https://docs.openshift.com/container-platform/3.4/creating_images/guidelines.html
RUN chown -R 1000:0 /etc/nuxeo && chmod g+rwX /etc/nuxeo && rm -f $NUXEO_HOME/bin/nuxeo.conf \
    && mkdir -p /var/lib/nuxeo/data \
    && chown -R 1000:0 /var/lib/nuxeo/data && chmod -R g+rwX /var/lib/nuxeo/data \
    && mkdir -p /var/log/nuxeo \
    && chown -R 1000:0 /var/log/nuxeo && chmod -R g+rwX /var/log/nuxeo \
    && mkdir -p /var/run/nuxeo \
    && chown -R 1000:0 /var/run/nuxeo && chmod -R g+rwX /var/run/nuxeo \
    && mkdir -p /docker-entrypoint-initnuxeo.d \
    && chown -R 1000:0 /docker-entrypoint-initnuxeo.d && chmod -R g+rwX /docker-entrypoint-initnuxeo.d  \
    && chmod g=u /etc/passwd

ENV PATH $NUXEO_HOME/bin:$PATH

WORKDIR $NUXEO_HOME
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 8080
CMD ["nuxeoctl","console"]

USER 1000
