FROM openjdk:8-jre-alpine
ADD config-*.jar app.jar
EXPOSE 18001
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]