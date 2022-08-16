# FROM docker-remote.artifactory.svc.elca.ch/sonarqube:8.9.3-community
# RUN apk --no-cache add curl
# COPY *.war /opt/sonarqube/extensions/plugins
FROM tomcat:7.0
ENV TZ Europe/Zurich
COPY *.war /usr/local/tomcat/webapps/