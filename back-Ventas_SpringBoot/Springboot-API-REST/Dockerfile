# --- Etapa 1: Compilación ---
FROM maven:3.9.6-eclipse-temurin-17 AS build
WORKDIR /app

# Copiar el archivo de configuración de dependencias
COPY pom.xml .
# Descargar dependencias para guardarlas en caché de Docker
RUN mvn dependency:go-offline

# Copiar el código fuente y compilar el archivo JAR (saltando pruebas)
COPY src ./src
RUN mvn clean package -DskipTests

# --- Etapa 2: Imagen de ejecución ---
FROM eclipse-temurin:17-jre-jammy
WORKDIR /app

# Copiar el JAR compilado desde la etapa anterior
COPY --from=build /app/target/*.jar app.jar

# Ventas corre en el puerto por defecto de Spring Boot (8080)
EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]