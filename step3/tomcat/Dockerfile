FROM tomcat:10.0.0-jdk11-corretto

WORKDIR /usr/local/tomcat/webapps/
RUN curl -O hhttps://tomcat.apache.org/tomcat-10.0-doc/appdev/sample/sample.war

CMD ["catalina.sh", "run"]