FROM jetty:alpine

COPY target/gameoflife.war /var/lib/jetty/webapps/ROOT.war

EXPOSE 8088
CMD ["jetty.sh", "run"]
