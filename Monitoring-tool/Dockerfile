FROM openjdk:17-jdk-slim

RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app
COPY app.jar /app/app.jar
RUN chown -R appuser:appgroup /app

USER appuser
ENTRYPOINT ["java", "-jar", "app.jar"]

