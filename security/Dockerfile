# Stage 1: Build
FROM openjdk:17-jdk-slim AS build

WORKDIR /app
COPY app.jar /app/app.jar

# Stage 2: Run as non-root
FROM openjdk:17-jdk-slim

# Create a non-root user and group
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app
COPY --from=build /app/app.jar /app/app.jar

# Change ownership to non-root user
RUN chown -R appuser:appgroup /app

# Switch to non-root
USER appuser

ENTRYPOINT ["java", "-jar", "app.jar"]

