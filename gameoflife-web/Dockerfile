FROM jetty:alpine

COPY target/gameoflife.war /var/lib/jetty/webapps/ROOT.war

EXPOSE 8080
CMD ["java","-jar","/usr/local/jetty/start.jar"]
