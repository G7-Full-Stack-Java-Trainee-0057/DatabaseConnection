# DatabaseConnection
Este repositorio contiene una guía paso a paso para configurar una conexión entre Spring Boot y PostgreSQL, utilizando un archivo `import.sql` para generar la estructura y datos iniciales de la base de datos.

## Índice
1. [Prerrequisitos](#prerrequisitos)
2. [Configuración de Spring Boot](#configuración-de-spring-boot)
3. [Conexion a base de datos sin Docker](docs/ConexionSinDocker.md)
4. [Conexion a base de datos con Docker](docs/ConexionConDocker.md)

## Prerrequisitos
- Java JDK 11 o superior
- Maven
- PostgreSQL / Docker (Dependera del caso)


## Configuración de Spring Boot
1. Añadir las dependencias necesarias en el `pom.xml`:
   ```xml
   <dependencies>
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
       <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
   </dependencies>
   ```

2. Configurar `application.properties`:
   ```properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/nombre_db
   spring.datasource.username=tu_usuario
   spring.datasource.password=tu_contraseña
   spring.datasource.driver-class-name=org.postgresql.Driver
   spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
   
   # Deshabilitar la generación automática del esquema
   spring.jpa.hibernate.ddl-auto=none
   
   # Habilitar la ejecución de import.sql
   spring.sql.init.mode=always
   
   # Mostrar las consultas SQL en la consola (opcional, útil para depuración)
   spring.jpa.show-sql=true
   ```