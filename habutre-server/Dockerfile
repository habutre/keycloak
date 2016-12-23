FROM frolvlad/alpine-oraclejdk8:slim

ARG keycloak_user
ARG keycloak_pass

ENV KEYCLOAK_VERSION 2.4.0.Final
# Enables signals getting passed from startup script to JVM
# ensuring clean shutdown when container is stopped.
ENV LAUNCH_JBOSS_IN_BACKGROUND 1
ENV JBOSS_HOME /opt/jboss/keycloak
ENV KEYCLOAK_USER $keycloak_user
ENV KEYCLOAK_PASSWORD $keycloak_pass

USER root

RUN  apk update && apk upgrade -Uqfal && adduser -S jboss && mkdir -p /opt/jboss && chown -R jboss: /opt/jboss # update and upgrade quiet+forcing

USER jboss

RUN mkdir -p /opt/jboss/ && cd /opt/jboss/ && wget -cP . http://downloads.jboss.org/keycloak/$KEYCLOAK_VERSION/keycloak-$KEYCLOAK_VERSION.tar.gz && tar -zxf keycloak-$KEYCLOAK_VERSION.tar.gz && mv /opt/jboss/keycloak-$KEYCLOAK_VERSION /opt/jboss/keycloak && rm -f keycloak-$KEYCLOAK_VERSION.tar.gz && ls -lah /opt/jboss && ls -lah /opt/jboss/keycloak

ADD docker-entrypoint.sh /opt/jboss/
ADD setLogLevel.xsl /opt/jboss/keycloak/
ADD changeDatabase.xsl /opt/jboss/keycloak/
ADD saxon9he.jar /opt/jboss/saxon.jar

RUN java -jar /opt/jboss/saxon.jar -s:/opt/jboss/keycloak/standalone/configuration/standalone.xml -xsl:/opt/jboss/keycloak/setLogLevel.xsl -o:/opt/jboss/keycloak/standalone/configuration/standalone.xml && java -jar /opt/jboss/saxon.jar -s:/opt/jboss/keycloak/standalone/configuration/standalone.xml -xsl:/opt/jboss/keycloak/changeDatabase.xsl -o:/opt/jboss/keycloak/standalone/configuration/standalone.xml && java -jar /opt/jboss/saxon.jar -s:/opt/jboss/keycloak/standalone/configuration/standalone-ha.xml -xsl:/opt/jboss/keycloak/changeDatabase.xsl -o:/opt/jboss/keycloak/standalone/configuration/standalone-ha.xml && rm /opt/jboss/keycloak/changeDatabase.xsl && mkdir -p /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main && rm /opt/jboss/saxon.jar #&& cd /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main && curl -O http://central.maven.org/maven2/org/postgresql/postgresql/9.3-1102-jdbc3/postgresql-9.3-1102-jdbc3.jar #### all commands are inline in order to avoid multiple layers on docker image

ADD module.xml /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main/
ADD postgresql-9.4.1212.jar /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main/

USER root

RUN echo "Current path" && pwd && chown -R jboss: /opt/jboss && echo "ls -lah /opt/jboss/" && ls -lah /opt/jboss/ && echo "ls -lah /opt/jboss/keycloak/" && ls -lah /opt/jboss/keycloak/ && echo "ls -lah /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main/" && ls -lah /opt/jboss/keycloak/modules/system/layers/base/org/postgresql/jdbc/main/ && echo "ls -lah /opt/jboss/keycloak/standalone/configuration/" && ls -lah /opt/jboss/keycloak/standalone/configuration/

EXPOSE 8080

ENTRYPOINT [ "/bin/sh", "/opt/jboss/docker-entrypoint.sh" ]

CMD ["-b", "0.0.0.0"]

