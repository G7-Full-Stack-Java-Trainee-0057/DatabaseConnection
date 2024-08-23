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
│   │       └── import.sql
│   └── test/
│       └── java/
│           └── com/
│               
├── pom.xml
└── README.md
```


1. Abre el archivo `src/main/resources/application.properties` y añade la siguiente configuración:

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


## Solución de Problemas

- **Tablas no creadas o datos no insertados**: 
  - Asegúrate de que el archivo `import.sql` esté en la ubicación correcta (`src/main/resources/`).
  - Verifica que no haya errores de sintaxis en las sentencias SQL.
  - Comprueba que `spring.sql.init.mode=always` esté configurado en `application.properties`.

- **Errores de "relation does not exist"**: 
  - Este error puede ocurrir si tu modelo JPA no coincide con la estructura de la tabla en la base de datos. Asegúrate de que los nombres de las columnas en tu clase de entidad coincidan exactamente con los de la tabla en `import.sql`.

- **Datos duplicados al reiniciar la aplicación**: 
  - Si no quieres que los datos se inserten cada vez que la aplicación se inicia, considera mover las sentencias INSERT a un archivo separado que ejecutes manualmente solo cuando sea necesario.

Si encuentras algún otro problema, revisa los logs de la aplicación para obtener más detalles sobre el error.