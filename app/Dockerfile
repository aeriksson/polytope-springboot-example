# syntax=docker/dockerfile:1.10.0

# Produce a minimal JRE build
FROM eclipse-temurin:23-jdk-alpine AS jre-build

RUN $JAVA_HOME/bin/jlink \
         --add-modules java.base,java.desktop,java.logging,java.management,java.naming,java.security.jgss,java.instrument \
         --strip-debug \
         --no-man-pages \
         --no-header-files \
         --compress=zip-6 \
         --output /javaruntime

# Build the application
FROM gradle:8-jdk23 AS build

WORKDIR /app

# See https://github.com/gradle/gradle/issues/11517.
ENV GRADLE_OPTS="-XX:MaxMetaspaceSize=384m -XX:+HeapDumpOnOutOfMemoryError -Xms256m -Xmx512m"

COPY build.gradle settings.gradle .
RUN --mount=type=cache,target=/root/.gradle \
    --mount=type=cache,target=/root/.m2 \
    gradle --no-daemon build -Dorg.gradle.project.buildDir=/build || true

COPY . .
RUN --mount=type=cache,target=/root/.gradle \
    --mount=type=cache,target=/root/.m2 \
    gradle --no-daemon build -Dorg.gradle.project.buildDir=/build \
    && rm /build/libs/*-plain.jar

# Build the final image
FROM alpine:3.20 AS final

WORKDIR /app
ENV JAVA_HOME=/opt/java/openjdk
ENV PATH="${JAVA_HOME}/bin:${PATH}"
COPY --from=jre-build /javaruntime $JAVA_HOME
# COPY --from=build /build /build
COPY --from=build /build/libs/*.jar /app/app.jar

CMD ["java", "-jar", "app.jar"]
