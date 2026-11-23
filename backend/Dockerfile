# Stage 1: compile the JAR with Maven (no Java needed on the host)
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn -B dependency:go-offline          # dependencies cached in their own layer
COPY src ./src
RUN mvn -B package -DskipTests

# Stage 2: only the JAR on a small Java runtime image
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/target/demo-0.0.1.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
