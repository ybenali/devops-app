FROM tomcat:latest
USER root
#RUN chmod 777 -R /var/jenkins_home/workspace/devops-app_main/target/
COPY /var/jenkins_home/workspace/devops-app_main/target/demo-0.0.1-SNAPSHOT.war /usr/local/tomcat/webapps/
WORKDIR  /usr/local/tomcat/bin
CMD ./startup
