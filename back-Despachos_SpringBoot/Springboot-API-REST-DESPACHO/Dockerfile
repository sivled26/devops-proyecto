# --- Etapa 1: Compilación ---
FROM maven:3.9.6-eclipse-temurin-17 AS build
WORKDIR /app

# Copiar el archivo de configuración de dependencias
COPY pom.xml .
# Descargar dependencias
RUN mvn dependency:go-offline

# Copiar el código fuente y compilar
COPY src ./src
RUN mvn clean package -DskipTests

# --- Etapa 2: Imagen de ejecución ---
FROM eclipse-temurin:17-jre-jammy
WORKDIR /app

# Copiar el JAR compilado
COPY --from=build /app/target/*.jar app.jar

# Despachos usa el puerto 8081 según sus properties
EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]