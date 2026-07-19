FROM gradle:8.5-jdk21-alpine AS build-stage
WORKDIR /app
COPY build.gradle settings.gradle gradle.properties ./
COPY gradle/ gradle/
COPY gradlew .
RUN gradle dependencies --no-daemon || true
COPY src ./src
RUN gradle assemble -x test --no-daemon

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build-stage /app/build/libs/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-Xmx512m", "-jar", "app.jar"]