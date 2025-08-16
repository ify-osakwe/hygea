#FROM maven:3.9.9-eclipse-temurin-21 AS builder
#WORKDIR /app
#COPY pom.xml .
#RUN mvn dependency:go-offline -B
#COPY src ./src
#RUN mvn clean package
#
#FROM openjdk:21-jdk AS runner
#WORKDIR /app
#COPY --from=builder ./app/target/analytics-service-0.0.1-SNAPSHOT.jar ./app.jar
#EXPOSE 4002
#ENTRYPOINT ["java", "-jar", "app.jar"]

FROM maven:3.9.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
# Preload deps; enable BuildKit caching for Maven repo
RUN --mount=type=cache,target=/root/.m2 mvn -B -q dependency:go-offline
COPY src ./src
RUN --mount=type=cache,target=/root/.m2 mvn -B -DskipTests clean package

FROM eclipse-temurin:21-jre-jammy AS run
WORKDIR /app
# Create a non-root user
RUN useradd -u 10001 -r spring && chown -R spring:spring /app
USER 10001

# Copy the built jar (no hard-coded name)
ARG JAR_FILE=target/*.jar
COPY --from=build /app/${JAR_FILE} /app/app.jar

EXPOSE 4002
# Let the JVM size itself sanely in containers & fail fast on OOM
ENV JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75 -XX:+ExitOnOutOfMemoryError"
ENTRYPOINT ["java","-jar","/app/app.jar"]