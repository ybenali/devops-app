FROM tomcat:latest
USER root
#RUN chmod 777 -R /var/jenkins_home/workspace/devops-app_main/target/
COPY  target/demo-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps/
WORKDIR  /usr/local/tomcat/bin
EXPOSE 8088
CMD tail -f /dev/null
