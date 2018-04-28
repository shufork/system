FROM openjdk:8-jre-alpine
ADD admin-*.jar app.jar
EXPOSE 18002
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]