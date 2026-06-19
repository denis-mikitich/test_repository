FROM --platform=linux/amd64 gradle:8.5-jdk21-alpine AS build-stage
WORKDIR /app
COPY build.gradle settings.gradle ./
COPY gradlew .
COPY gradle/ gradle/
RUN gradle build -x test --no-daemon || return 0

COPY src ./src
RUN gradle clean assemble -x test --no-daemon

FROM --platform=linux/amd64 eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build-stage /app/build/libs/ci-back-end-0.0.1-SNAPSHOT.jar app.jar
ENV POSTGRES_HOST=192.168.31.86 \
    POSTGRES_PORT=5432 \
    POSTGRES_USER=postgres \
    POSTGRES_PASSWORD=xkmjr3m3 \
    POSTGRES_DB=ci-db \
    REDIS_HOST=192.168.31.86 \
    REDIS_PORT=6379 \
    REDIS_PASSWORD=Uhsbp5jyDc \
    FLUENTD_ENABLED=false \
    SERVER_PORT=8081 \
    DEBUG=true

EXPOSE 8081

ENTRYPOINT ["java", "-Xmx512m", "-jar", "app.jar"]
#FROM eclipse-temurin:21-jre
#WORKDIR /app

#COPY build/libs/*.jar app.jar
#EXPOSE 8081
#ENTRYPOINT ["java", "-Xmx512m", "-jar", "app.jar"]
