## Configuración de Spring Boot
```
proyecto-spring-postgres/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── ejemplo/
│   │   │           ├── ProyectoApplication.java
│   │   │           ├── model/
│   │   │           │   ├── Usuario.java
│   │   │           │   └── Rol.java
│   │   │           ├── repository/
│   │   │           │   ├── UsuarioRepository.java
│   │   │           │   └── RolRepository.java
│   │   │           ├── service/
│   │   │           │   ├── UsuarioService.java
│   │   │           │   └── RolService.java
│   │   │           └── controller/
│   │   │               ├── UsuarioController.java
│   │   │               └── RolController.java
│   │   └── resources/
│   │       ├── application.properties
│   └── test/
│       └── java/
│           └── com/
│               
├── docker-compose.yml
├── init.sql
├── pom.xml
└── README.md
```


1. Abre el archivo `src/main/resources/application.properties` y añade la siguiente configuración:

   ```properties
    spring.datasource.url=jdbc:postgresql://localhost:5432/nombre_db
    spring.datasource.username=usuario
    spring.datasource.password=contraseña
    spring.datasource.driver-class-name=org.postgresql.Driver
    spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect

    spring.jpa.hibernate.ddl-auto=none
    spring.sql.init.mode=always
    spring.jpa.show-sql=true
   ```

Asegúrate de reemplazar `nombre_db`, `tu_usuario` y `tu_contraseña` con tus datos reales.

Nota: `spring.jpa.hibernate.ddl-auto=none` es crucial aquí, ya que queremos que Hibernate no intente crear o modificar el esquema de la base de datos automáticamente.


## Creación de Tablas e Inserción de Datos

Crea un archivo `src/main/resources/import.sql`. Este archivo se ejecutará automáticamente cuando inicies la aplicación. Aquí puedes incluir tanto la creación de tablas como la inserción de datos iniciales:

```sql
-- Eliminar tablas si existen
DROP TABLE IF EXISTS usuarios;
DROP TABLE IF EXISTS roles;
DROP TABLE IF EXISTS usuario_roles;

-- Crear tabla de usuarios
CREATE TABLE usuarios (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Crear tabla de roles
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    nombre VARCHAR(50) UNIQUE NOT NULL
);

-- Crear tabla de relación entre usuarios y roles
CREATE TABLE usuario_roles (
    usuario_id INTEGER REFERENCES usuarios(id),
    rol_id INTEGER REFERENCES roles(id),
    PRIMARY KEY (usuario_id, rol_id)
);

-- Insertar datos de ejemplo
INSERT INTO usuarios (nombre, email, password) VALUES 
('Juan Pérez', 'juan@example.com', 'password123'),
('Ana García', 'ana@example.com', 'password456');

INSERT INTO roles (nombre) VALUES 
('ROLE_USER'),
('ROLE_ADMIN');

INSERT INTO usuario_roles (usuario_id, rol_id) VALUES 
(1, 1),  -- Juan es un usuario normal
(2, 1),  -- Ana es un usuario normal
(2, 2);  -- Ana también es admin
```

## Archivo docker-compose.yml
```
version: '3.8'

services:
  db:
    image: postgres:13-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=nombre_db
      - POSTGRES_USER=usuario
      - POSTGRES_PASSWORD=contraseña
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
```

1. El archivo `docker-compose.yml` está configurado en la raíz del proyecto para iniciar PostgreSQL.

2. Inicia la base de datos con el siguiente comando:
```
docker-compose up -d
```
3. Cuando hayas terminado de trabajar con el proyecto, puedes detener la base de datos con:
```
docker-compose down
```

## Solución de Problemas

- **Error de conexión a la base de datos**: Verifica que el contenedor de PostgreSQL esté en ejecución y que los datos de conexión en `application.properties` sean correctos.
- **Tablas no creadas o datos no insertados**: Verifica el contenido de `init.sql` y asegúrate de que `spring.sql.init.mode=always` esté configurado en `application.properties`.
- **Errores de "relation does not exist"**: Asegúrate de que los modelos JPA coincidan con la estructura de la base de datos definida en `init.sql`.

Si encuentras algún otro problema, revisa los logs de la aplicación para obtener más detalles sobre el error.

# Navegación dentro del contenedor PostgreSQL

Estos comandos te permitirán interactuar con el contenedor PostgreSQL y su base de datos.

## Acceder al contenedor

1. entrar al contenedor usando `docker`:
```
docker exec -it <nombre_o_id_del_contenedor> bash
```
   Reemplaza `<nombre_o_id_del_contenedor>` con el nombre o ID de tu contenedor PostgreSQL.

## Interactuar con PostgreSQL

Una vez dentro del contenedor:

1. Acceder a la línea de comandos de PostgreSQL:
```
psql -U usuario -d nombre_db
```
   Reemplaza `usuario` y `nombre_db` con los valores que configuraste en el `docker-compose.yml`.

2. Algunos comandos útiles de psql:
   - `\l`: Listar todas las bases de datos
   - `\c nombre_db`: Conectar a una base de datos específica
   - `\dt`: Listar todas las tablas en la base de datos actual
   - `\d nombre_tabla`: Describir la estructura de una tabla
   - `\q`: Salir de psql

## Ejemplos de consultas SQL

Una vez en psql, puedes ejecutar consultas SQL directamente(ejemplo):

1. Ver todos los usuarios:
```sql
SELECT * FROM usuarios;
```

2. Ver todos los roles:
```sql
SELECT * FROM roles;
```

3. Ver las relaciones usuario-rol:
```sql
SELECT u.nombre, r.nombre as rol 
FROM usuarios u 
JOIN usuario_roles ur ON u.id = ur.usuario_id 
JOIN roles r ON ur.rol_id = r.id;
```

## Salir del contenedor

Para salir del contenedor, simplemente escribe:
```
exit
```

Recuerda que estos comandos son útiles para depuración y verificación, pero para operaciones regulares, es mejor usar tu aplicación Spring Boot o scripts de migración para interactuar con la base de datos.
