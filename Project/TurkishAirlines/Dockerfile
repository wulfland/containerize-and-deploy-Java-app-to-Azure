FROM tomcat

COPY tomcat-users.xml /usr/local/tomcat/conf

ADD target/*.war /usr/local/tomcat/webapps/TurkishAirlines.war

EXPOSE 8080

CMD ["catalina.sh", "run"]