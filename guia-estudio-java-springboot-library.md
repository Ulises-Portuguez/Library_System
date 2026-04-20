# Guía de Estudio: Java 21 + Spring Boot → Online Library System

> **Meta final:** Construir el backend completo del challenge "Online Library System" con Event-Driven Architecture (Option 3), cubriendo desde los fundamentos de Spring Boot hasta integración con RabbitMQ.

**Tiempo estimado total:** 8–9 semanas (2–3 horas diarias)

---

## Índice

1. [Fase 1: Fundamentos de Java 21 y Spring Boot](#fase-1-fundamentos-de-java-21-y-spring-boot)
2. [Fase 2: Arquitectura y Estructura del Proyecto](#fase-2-arquitectura-y-estructura-del-proyecto)
3. [Fase 3: JPA, Base de Datos y Modelado](#fase-3-jpa-base-de-datos-y-modelado)
4. [Fase 4: Flyway — Migraciones de Base de Datos](#fase-4-flyway--migraciones-de-base-de-datos)
5. [Fase 5: Seguridad — Spring Security + JWT + BCrypt](#fase-5-seguridad--spring-security--jwt--bcrypt)
6. [Fase 6: API REST Completa y Validación](#fase-6-api-rest-completa-y-validación)
7. [Fase 7: @Transactional y Consistencia de Datos](#fase-7-transactional-y-consistencia-de-datos)
8. [Fase 8: Logging y Trazabilidad Contextual](#fase-8-logging-y-trazabilidad-contextual)
9. [Fase 9: Event-Driven Architecture con RabbitMQ](#fase-9-event-driven-architecture-con-rabbitmq)
10. [Fase 10: Testing](#fase-10-testing)
11. [Fase 11: Dockerización y Entrega](#fase-11-dockerización-y-entrega)
12. [Recursos Generales](#recursos-generales)

---

## Fase 1: Fundamentos de Java 21 y Spring Boot

**Duración estimada:** 1 semana

### ¿Por qué esta fase?

El challenge exige Java 21 explícitamente y pide aprovechar sus features modernas. Sin entender cómo funciona Spring Boot por dentro, el resto de las fases serán confusas.

### Objetivos puntuales

- [x] **1.1** — Instalar JDK 21 (Temurin o GraalVM), Maven o Gradle, e IntelliJ IDEA (o VS Code con Extension Pack for Java). Verificar con `java --version` que la versión es 21+.
- [x] **1.2** — Estudiar **Records** de Java 21: crear al menos 3 Records (ej. `BookDTO`, `UserDTO`, `ErrorResponse`) y entender que son inmutables, tienen `equals`, `hashCode` y `toString` auto-generados, y sirven como DTOs ideales.
- [ ] **1.3** — Estudiar **Pattern Matching con `instanceof`**: escribir un método que reciba un `Object` y use `if (obj instanceof String s)` para extraer el tipo sin cast manual. Luego practicar con **Switch Expressions** mejoradas usando `case String s -> ...`.
- [ ] **1.4** — Estudiar **Sealed Classes / Interfaces**: crear una jerarquía `sealed interface LibraryEvent permits BookReserved, ReviewSubmitted`. Entender que el compilador garantiza que todos los subtipos están cubiertos en un switch.
- [ ] **1.5** — Estudiar **Virtual Threads (Project Loom)**: entender la diferencia entre threads de plataforma (OS-level, costosos) y virtual threads (gestionados por la JVM, miles simultáneos sin problema). Configurar Spring Boot para usarlos con `spring.threads.virtual.enabled=true`.
- [ ] **1.6** — Generar un proyecto desde [start.spring.io](https://start.spring.io) con las dependencias: Spring Web, Spring Boot DevTools, Lombok (opcional). Elegir Maven o Gradle y Java 21.
- [ ] **1.7** — Entender el **contenedor IoC** de Spring: cómo Spring crea y gestiona objetos (beans), qué es la inyección de dependencias, y por qué la inyección por constructor es la recomendada sobre `@Autowired` en campos.
- [ ] **1.8** — Conocer los **estereotipos** principales y cuándo usar cada uno:
  - `@Component` → bean genérico
  - `@Service` → lógica de negocio
  - `@Repository` → acceso a datos (agrega traducción de excepciones de BD)
  - `@Controller` / `@RestController` → endpoints HTTP
- [ ] **1.9** — Entender el ciclo de vida de un bean: instanciación → inyección de dependencias → `@PostConstruct` → uso → `@PreDestroy`. Conocer los scopes: `singleton` (default, una instancia compartida) vs `prototype` (nueva instancia cada vez que se solicita).
- [ ] **1.10** — Aprender a configurar la aplicación: diferencia entre `application.properties` y `application.yml`, cómo usar **profiles** (`application-dev.yml`, `application-prod.yml`), y cómo activar un profile con `spring.profiles.active`.
- [ ] **1.11** — Crear un endpoint `GET /api/hello` que devuelva un Record como JSON. Verificar con curl o Postman que funciona. Entender qué hace Jackson (serialización/deserialización JSON) por detrás.

### Entregable de fase

Una aplicación Spring Boot que arranca correctamente, tiene un endpoint funcional que devuelve JSON usando un Record de Java 21, y usa virtual threads habilitados.

---

## Fase 2: Arquitectura y Estructura del Proyecto

**Duración estimada:** 2–3 días

### ¿Por qué esta fase?

Definir la estructura antes de escribir lógica evita refactorizaciones costosas después. El challenge evalúa "clear, maintainable code", y eso empieza por la organización.

### Objetivos puntuales

- [ ] **2.1** — Estudiar la **arquitectura en capas (Layered Architecture)**: Controller → Service → Repository. Entender que cada capa tiene una responsabilidad única: el Controller maneja HTTP, el Service contiene lógica de negocio, el Repository accede a datos. Ventaja: simple y familiar. Desventaja: puede generar "servicios gordos" y acoplamiento vertical.
- [ ] **2.2** — Estudiar la **arquitectura hexagonal (Ports & Adapters)**: el dominio está en el centro sin dependencias externas. Los "puertos" son interfaces que el dominio define (ej. `BookRepository` como interfaz en el dominio), y los "adaptadores" son implementaciones concretas (ej. `JpaBookRepository`). Ventaja: dominio testeable sin Spring ni BD. Desventaja: más archivos y abstracción inicial.
- [ ] **2.3** — Decidir e implementar una **estructura por feature** (paquete por dominio, no por capa):

```
src/main/java/com/library/
├── book/
│   ├── controller/
│   │   └── BookController.java
│   ├── service/
│   │   └── BookService.java
│   ├── repository/
│   │   └── BookRepository.java
│   ├── model/
│   │   └── Book.java
│   └── dto/
│       ├── BookResponse.java        (Record)
│       ├── BookSearchRequest.java   (Record)
│       └── CreateBookRequest.java   (Record)
├── reservation/
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── model/
│   └── dto/
├── review/
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── model/
│   └── dto/
├── user/
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── model/
│   └── dto/
├── auth/
│   ├── controller/
│   ├── service/
│   └── dto/
├── event/                          ← Para la Option 3
│   ├── model/
│   ├── publisher/
│   ├── consumer/
│   └── store/
└── shared/
    ├── config/                     ← Configuraciones globales
    ├── exception/                  ← Manejo global de errores
    ├── security/                   ← Filtros JWT, SecurityConfig
    └── logging/                    ← Filtros MDC, utilidades de log
```

- [ ] **2.4** — Entender la diferencia entre los tres modelos de datos y cuándo usar cada uno:
  - **Entidad (`@Entity`):** Modelo de persistencia, mapeado a la tabla. Tiene anotaciones JPA. Nunca se expone directamente al exterior.
  - **DTO (Record):** Objeto de transferencia, lo que entra y sale del API. Inmutable, sin lógica, con anotaciones de validación.
  - **Mapper:** Clase que convierte entre entidad y DTO. Puede ser manual o usar MapStruct.
- [ ] **2.5** — Crear un mapper manual para `Book` → `BookResponse` y `CreateBookRequest` → `Book`. Entender por qué esta separación protege tu modelo interno de cambios en el API y viceversa.
- [ ] **2.6** — Crear la estructura de paquetes completa (pueden estar vacíos por ahora) y verificar que la aplicación sigue arrancando correctamente.

### Entregable de fase

La estructura de paquetes completa del proyecto creada, con al menos el módulo `book` teniendo un Controller, Service, Repository (vacío por ahora), entidad y DTOs como Records.

---

## Fase 3: JPA, Base de Datos y Modelado

**Duración estimada:** 1 semana

### ¿Por qué esta fase?

La base de datos es el corazón de la aplicación. Un mal diseño de esquema genera problemas que son muy costosos de corregir después.

### Objetivos puntuales

#### JPA Fundamentals

- [ ] **3.1** — Agregar las dependencias `spring-boot-starter-data-jpa` y el driver de PostgreSQL (`postgresql`). Configurar la conexión en `application.yml`:
  ```yaml
  spring:
    datasource:
      url: jdbc:postgresql://localhost:5432/library
      username: library_user
      password: library_pass
    jpa:
      hibernate:
        ddl-auto: validate  # Solo valida, no modifica (usaremos Flyway)
      show-sql: true        # Solo en dev
      properties:
        hibernate:
          format_sql: true
  ```
- [ ] **3.2** — Estudiar las anotaciones de entidad fundamentales:
  - `@Entity` — Marca la clase como entidad JPA
  - `@Table(name = "books")` — Nombre de la tabla
  - `@Id` + `@GeneratedValue(strategy = GenerationType.IDENTITY)` — Clave primaria auto-incremental
  - `@Column(name = "title", nullable = false, length = 255)` — Configuración de columna
  - `@Enumerated(EnumType.STRING)` — Para enums (ej. estado de reservación)
  - `@CreationTimestamp`, `@UpdateTimestamp` — Timestamps automáticos
- [ ] **3.3** — Estudiar las relaciones JPA y mapear las del challenge:
  - `@ManyToOne` — Una reservación pertenece a un usuario y a un libro
  - `@OneToMany(mappedBy = "user")` — Un usuario tiene muchas reservaciones
  - `@JoinColumn(name = "user_id")` — Define la columna FK
  - Siempre usar `FetchType.LAZY` excepto en `@ManyToOne` donde evaluar caso a caso
  - Entender el problema N+1 y cómo resolverlo con `@EntityGraph` o `JOIN FETCH`

#### Modelado del esquema

- [ ] **3.4** — Crear la entidad `User`:
  - Campos: `id` (Long), `username` (único), `email` (único), `passwordHash`, `role` (enum: USER, ADMIN), `createdAt`
  - Constraints: `@Column(unique = true)` en username y email
- [ ] **3.5** — Crear la entidad `Book`:
  - Campos: `id`, `title`, `author`, `genre`, `isbn` (único), `description`, `availableCopies`, `totalCopies`, `publishedDate`, `createdAt`
  - Relación: `@OneToMany(mappedBy = "book") List<Review> reviews`
- [ ] **3.6** — Crear la entidad `Reservation`:
  - Campos: `id`, `user` (FK), `book` (FK), `status` (enum: ACTIVE, COMPLETED, CANCELLED), `reservedAt`, `expiresAt`, `returnedAt`
  - Relaciones: `@ManyToOne(fetch = LAZY) @JoinColumn(name = "user_id") User user`
- [ ] **3.7** — Crear la entidad `Review`:
  - Campos: `id`, `user` (FK), `book` (FK), `rating` (1–5), `comment`, `createdAt`
  - Constraint único: un usuario solo puede hacer un review por libro → `@Table(uniqueConstraints = @UniqueConstraint(columns = {"user_id", "book_id"}))`

#### Normalización

- [ ] **3.8** — Estudiar las formas normales y verificar que tu esquema las cumple:
  - **1NF (Primera Forma Normal):** Cada columna contiene valores atómicos, no listas ni objetos anidados. Ejemplo incorrecto: `genres = "fiction,drama"`. Correcto: una tabla `book_genres` o un enum con un solo género principal.
  - **2NF (Segunda Forma Normal):** Cada columna no-clave depende de la clave primaria completa, no de una parte. Relevante cuando tienes claves compuestas.
  - **3NF (Tercera Forma Normal):** No hay dependencias transitivas. Ejemplo incorrecto: guardar `authorName` y `authorBiography` en la tabla `books` → si necesitas datos del autor, normaliza a una tabla `authors`. Para el challenge, mantener `author` como String en `books` es aceptable si no necesitas más datos del autor (desnormalización pragmática).
- [ ] **3.9** — Documentar las decisiones de normalización/desnormalización en el README. Ejemplo: "Se mantuvo `author` como campo de texto en `books` en lugar de una tabla separada porque el alcance del challenge no requiere gestión de autores como entidad independiente."

#### Indexación

- [ ] **3.10** — Estudiar qué es un índice de base de datos: es una estructura de datos (generalmente B-Tree) que permite encontrar filas sin escanear toda la tabla. Acelera lecturas pero hace las escrituras ligeramente más lentas.
- [ ] **3.11** — Identificar las columnas que necesitan índice según los patrones de consulta del challenge:
  - `books.title` — Búsqueda por título (`GET /books?title=...`)
  - `books.author` — Búsqueda por autor
  - `books.genre` — Filtro por género
  - `books.isbn` — Ya tiene índice por ser `unique`
  - `reservations.user_id` + `reservations.status` — Índice compuesto para "reservaciones activas de un usuario"
  - `reviews.book_id` — Listar reviews de un libro (`GET /reviews?bookId=...`)
- [ ] **3.12** — Implementar índices en JPA:
  ```java
  @Table(name = "books", indexes = {
      @Index(name = "idx_book_title", columnList = "title"),
      @Index(name = "idx_book_author", columnList = "author"),
      @Index(name = "idx_book_genre", columnList = "genre")
  })
  ```
- [ ] **3.13** — Entender cuándo un índice B-Tree **no** ayuda: búsquedas con `LIKE '%keyword%'` (wildcard al inicio) no usan el índice. Para búsqueda de texto parcial en títulos, estudiar alternativas:
  - **PostgreSQL full-text search** con `tsvector` y `tsquery`
  - **Índice trigram** con la extensión `pg_trgm` (soporta `LIKE '%keyword%'`)
  - Para el challenge, un `LIKE 'keyword%'` (sin wildcard al inicio) es aceptable y sí usa el índice

#### Spring Data JPA Repositories

- [ ] **3.14** — Crear los repositorios extendiendo `JpaRepository<T, ID>`:
  ```java
  public interface BookRepository extends JpaRepository<Book, Long> {
      Page<Book> findByTitleContainingIgnoreCase(String title, Pageable pageable);
      List<Book> findByGenre(String genre);
      Optional<Book> findByIsbn(String isbn);
  }
  ```
- [ ] **3.15** — Estudiar **Specifications** para búsquedas dinámicas. Cuando `GET /books` puede filtrar por título, autor, género (todos opcionales), los query methods no escalan. Implementar `Specification<Book>` para construir queries dinámicamente:
  ```java
  public class BookSpecifications {
      public static Specification<Book> hasTitle(String title) {
          return (root, query, cb) ->
              title == null ? null : cb.like(cb.lower(root.get("title")), "%" + title.toLowerCase() + "%");
      }
      // ... similar para author, genre
  }
  ```
- [ ] **3.16** — Probar que los repositorios funcionan. Puedes usar `ddl-auto: create` temporalmente para verificar que las tablas se crean correctamente. En la siguiente fase lo reemplazaremos con Flyway.

### Entregable de fase

Las 4 entidades JPA creadas con relaciones, constraints e índices. Los repositorios con query methods y al menos una Specification. Diagrama ER documentado (puede ser texto o diagrama).

---

## Fase 4: Flyway — Migraciones de Base de Datos

**Duración estimada:** 2 días

### ¿Por qué esta fase?

`hibernate.ddl-auto=update` es peligroso en producción: nunca borra columnas, no renombra correctamente, y no deja historial. Flyway versiona tu esquema como versionas tu código.

### Objetivos puntuales

- [ ] **4.1** — Agregar la dependencia `flyway-core` y `flyway-database-postgresql`. Configurar en `application.yml`:
  ```yaml
  spring:
    flyway:
      enabled: true
      locations: classpath:db/migration
      baseline-on-migrate: true
    jpa:
      hibernate:
        ddl-auto: validate  # IMPORTANTE: no 'update', no 'create'
  ```
- [ ] **4.2** — Entender la convención de nombres de Flyway:
  - `V1__descripcion.sql` — Migración versionada (se ejecuta una sola vez, en orden)
  - `R__descripcion.sql` — Migración repetible (se re-ejecuta si el contenido cambia, útil para vistas o funciones)
  - El doble guion bajo (`__`) después del número es obligatorio
  - Los números deben ser secuenciales y nunca se modifican después de aplicar
- [ ] **4.3** — Crear la migración `V1__create_users_table.sql`:
  ```sql
  CREATE TABLE users (
      id BIGSERIAL PRIMARY KEY,
      username VARCHAR(50) NOT NULL UNIQUE,
      email VARCHAR(100) NOT NULL UNIQUE,
      password_hash VARCHAR(255) NOT NULL,
      role VARCHAR(20) NOT NULL DEFAULT 'USER',
      created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
  );
  ```
- [ ] **4.4** — Crear la migración `V2__create_books_table.sql` incluyendo los índices:
  ```sql
  CREATE TABLE books (
      id BIGSERIAL PRIMARY KEY,
      title VARCHAR(255) NOT NULL,
      author VARCHAR(150) NOT NULL,
      genre VARCHAR(50),
      isbn VARCHAR(13) UNIQUE,
      description TEXT,
      available_copies INT NOT NULL DEFAULT 0,
      total_copies INT NOT NULL DEFAULT 0,
      published_date DATE,
      created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
  );

  CREATE INDEX idx_book_title ON books (title);
  CREATE INDEX idx_book_author ON books (author);
  CREATE INDEX idx_book_genre ON books (genre);
  ```
- [ ] **4.5** — Crear `V3__create_reservations_table.sql` con foreign keys y el índice compuesto:
  ```sql
  CREATE TABLE reservations (
      id BIGSERIAL PRIMARY KEY,
      user_id BIGINT NOT NULL REFERENCES users(id),
      book_id BIGINT NOT NULL REFERENCES books(id),
      status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
      reserved_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
      expires_at TIMESTAMP,
      returned_at TIMESTAMP
  );

  CREATE INDEX idx_reservation_user_status ON reservations (user_id, status);
  CREATE INDEX idx_reservation_book ON reservations (book_id);
  ```
- [ ] **4.6** — Crear `V4__create_reviews_table.sql` con el constraint único:
  ```sql
  CREATE TABLE reviews (
      id BIGSERIAL PRIMARY KEY,
      user_id BIGINT NOT NULL REFERENCES users(id),
      book_id BIGINT NOT NULL REFERENCES books(id),
      rating INT NOT NULL CHECK (rating BETWEEN 1 AND 5),
      comment TEXT,
      created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
      UNIQUE (user_id, book_id)
  );

  CREATE INDEX idx_review_book ON reviews (book_id);
  ```
- [ ] **4.7** — Crear `V5__seed_sample_data.sql` con datos de prueba: al menos 5 usuarios, 15 libros de diferentes géneros, algunas reservaciones y reviews.
- [ ] **4.8** — Arrancar la aplicación y verificar que Flyway ejecuta las migraciones. Revisar la tabla `flyway_schema_history` que Flyway crea automáticamente para rastrear qué migraciones se han aplicado.
- [ ] **4.9** — Practicar un escenario de evolución: crear `V6__add_column_book_language.sql` que agregue una columna `language` a `books`. Verificar que se aplica sin afectar datos existentes.
- [ ] **4.10** — Entender las reglas de oro de Flyway:
  - Nunca modificar una migración ya aplicada
  - Nunca borrar una migración ya aplicada
  - Si cometiste un error, crea una nueva migración que lo corrija
  - En producción, probar migraciones en un ambiente de staging primero

### Entregable de fase

5+ archivos de migración en `src/main/resources/db/migration/`. La aplicación arranca, Flyway ejecuta las migraciones, y `hibernate.ddl-auto=validate` confirma que entidades y esquema coinciden.

---

## Fase 5: Seguridad — Spring Security + JWT + BCrypt

**Duración estimada:** 1 semana

### ¿Por qué esta fase?

El challenge requiere usuarios y reservaciones asociadas a usuarios. Sin autenticación, cualquiera podría hacer reservaciones o reviews a nombre de otro.

### Objetivos puntuales

#### BCrypt — Encriptación de contraseñas

- [ ] **5.1** — Entender por qué nunca se almacenan contraseñas en texto plano ni con hashes simples (MD5, SHA-256): son vulnerables a ataques de fuerza bruta y rainbow tables. BCrypt es deliberadamente lento (configurable con el parámetro "strength") e incluye un salt aleatorio automático, lo que hace cada hash único incluso para contraseñas iguales.
- [ ] **5.2** — Registrar un bean `PasswordEncoder`:
  ```java
  @Bean
  public PasswordEncoder passwordEncoder() {
      return new BCryptPasswordEncoder(12); // 12 rounds, buen balance seguridad/velocidad
  }
  ```
- [ ] **5.3** — Implementar el flujo de registro: recibir contraseña en texto plano → `passwordEncoder.encode(rawPassword)` → guardar el hash en BD. Verificar que el hash guardado empieza con `$2a$` (firma de BCrypt).

#### JWT — JSON Web Tokens

- [ ] **5.4** — Agregar la dependencia `io.jsonwebtoken:jjwt-api`, `jjwt-impl`, y `jjwt-jackson`. Estudiar la estructura de un JWT:
  - **Header:** `{"alg": "HS256", "typ": "JWT"}` — algoritmo de firma
  - **Payload:** `{"sub": "userId", "username": "john", "role": "USER", "iat": 1234567890, "exp": 1234654290}` — claims (datos)
  - **Signature:** HMAC-SHA256 del header + payload + secret key
- [ ] **5.5** — Crear un `JwtService` con los siguientes métodos:
  - `generateToken(User user)` → Crea un JWT con userId como subject, username y role como claims, expiración de 24 horas, firmado con una clave secreta (mínimo 256 bits, almacenada en `application.yml` y nunca en código).
  - `extractUsername(String token)` → Parsea el JWT y extrae el username.
  - `isTokenValid(String token, UserDetails userDetails)` → Verifica firma, expiración, y que el usuario coincida.
- [ ] **5.6** — Entender que el JWT es **stateless**: el servidor no almacena sesiones. Cada request lleva su propia "credencial" (el token). Esto simplifica la escalabilidad horizontal (múltiples instancias del servidor).

#### Spring Security — Configuración

- [ ] **5.7** — Agregar la dependencia `spring-boot-starter-security`. Entender que al agregarla, Spring Security protege **todos** los endpoints automáticamente (devuelve 401 a todo). El trabajo es configurar qué se protege y qué no.
- [ ] **5.8** — Crear `SecurityConfig` con `SecurityFilterChain`:
  ```java
  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
      return http
          .csrf(csrf -> csrf.disable())  // Innecesario en APIs REST stateless
          .sessionManagement(session ->
              session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
          .authorizeHttpRequests(auth -> auth
              .requestMatchers("/auth/**").permitAll()          // Login/registro público
              .requestMatchers(HttpMethod.GET, "/books/**").permitAll()  // Búsqueda pública
              .requestMatchers("/actuator/health").permitAll()  // Health check público
              .anyRequest().authenticated()                     // Todo lo demás requiere JWT
          )
          .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
          .build();
  }
  ```
- [ ] **5.9** — Crear `JwtAuthenticationFilter` (extiende `OncePerRequestFilter`):
  - Extraer el header `Authorization`
  - Verificar que empieza con `Bearer `
  - Extraer el token (quitar el prefijo "Bearer ")
  - Validar el token con `JwtService`
  - Si es válido, crear un `UsernamePasswordAuthenticationToken` y establecer en `SecurityContextHolder`
  - Si no es válido o no hay token, la cadena de filtros continúa sin autenticación (Spring Security devolverá 401/403 según la configuración)
- [ ] **5.10** — Implementar `UserDetailsService` que carga el usuario desde la BD por username. Spring Security lo usa internamente para la autenticación.

#### Endpoints de autenticación

- [ ] **5.11** — Crear `AuthController` con:
  - `POST /auth/register` — Recibe `RegisterRequest(username, email, password)`, valida que username y email no existan, hashea la contraseña, guarda el usuario, devuelve un JWT.
  - `POST /auth/login` — Recibe `LoginRequest(username, password)`, busca el usuario, verifica la contraseña con `passwordEncoder.matches()`, devuelve un JWT si es correcto o 401 si no.
- [ ] **5.12** — Probar el flujo completo:
  1. Registrar un usuario con `POST /auth/register`
  2. Copiar el JWT devuelto
  3. Hacer una request autenticada con `Authorization: Bearer <token>` a un endpoint protegido
  4. Verificar que sin token devuelve 401
  5. Verificar que con token expirado o inválido devuelve 401
- [ ] **5.13** — Implementar acceso al usuario autenticado en controllers usando `@AuthenticationPrincipal UserDetails userDetails` o creando una utilidad `SecurityUtils.getCurrentUserId()`.

### Entregable de fase

Registro y login funcionales. JWT generado y validado correctamente. Endpoints protegidos que rechazan requests sin token. Contraseñas hasheadas con BCrypt en la BD.

---

## Fase 6: API REST Completa y Validación

**Duración estimada:** 1 semana

### ¿Por qué esta fase?

Esta fase implementa todos los endpoints que el challenge pide explícitamente. Al completarla, ya tienes la Option 1 funcional.

### Objetivos puntuales

#### Validación

- [ ] **6.1** — Agregar la dependencia `spring-boot-starter-validation`. Estudiar las anotaciones principales:
  - `@NotNull`, `@NotBlank` (para Strings no vacíos)
  - `@Size(min, max)` (longitud de strings o colecciones)
  - `@Min`, `@Max` (valores numéricos)
  - `@Email` (formato de email)
  - `@Pattern(regexp)` (para ISBN u otros formatos)
  - `@Valid` (en el controller para activar la validación)
- [ ] **6.2** — Crear DTOs con validación para cada operación:
  ```java
  public record CreateBookRequest(
      @NotBlank @Size(max = 255) String title,
      @NotBlank @Size(max = 150) String author,
      @Size(max = 50) String genre,
      @Pattern(regexp = "\\d{13}", message = "ISBN debe tener 13 dígitos") String isbn,
      String description,
      @Min(0) int totalCopies
  ) {}

  public record CreateReservationRequest(
      @NotNull Long bookId
  ) {}

  public record CreateReviewRequest(
      @NotNull Long bookId,
      @Min(1) @Max(5) int rating,
      @Size(max = 2000) String comment
  ) {}
  ```

#### Manejo global de excepciones

- [ ] **6.3** — Crear un `@RestControllerAdvice` llamado `GlobalExceptionHandler`:
  ```java
  @RestControllerAdvice
  public class GlobalExceptionHandler {

      @ExceptionHandler(EntityNotFoundException.class)
      public ResponseEntity<ErrorResponse> handleNotFound(EntityNotFoundException ex) { ... }

      @ExceptionHandler(MethodArgumentNotValidException.class)
      public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) { ... }

      @ExceptionHandler(DataIntegrityViolationException.class)
      public ResponseEntity<ErrorResponse> handleConstraintViolation(DataIntegrityViolationException ex) { ... }

      @ExceptionHandler(AccessDeniedException.class)
      public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex) { ... }
  }
  ```
- [ ] **6.4** — Crear un Record `ErrorResponse` estándar:
  ```java
  public record ErrorResponse(
      Instant timestamp,
      int status,
      String error,
      String message,
      String path,
      Map<String, String> fieldErrors  // Para errores de validación
  ) {}
  ```
- [ ] **6.5** — Crear excepciones custom: `BookNotFoundException`, `NoAvailableCopiesException`, `DuplicateReviewException`. Todas extienden `RuntimeException` y llevan información contextual.

#### Paginación y búsqueda

- [ ] **6.6** — Implementar `GET /books` con paginación y filtros opcionales:
  - Parámetros: `title`, `author`, `genre`, `page`, `size`, `sort`
  - Usar `Specification<Book>` para combinar filtros dinámicamente
  - Devolver `Page<BookResponse>` que incluye: contenido, número de página, total de páginas, total de elementos
- [ ] **6.7** — Entender cómo Spring traduce `Pageable`: un request a `/books?page=0&size=10&sort=title,asc` se convierte automáticamente en un objeto `Pageable` que JPA usa para el `LIMIT`/`OFFSET` y `ORDER BY`.

#### Endpoints del challenge

- [ ] **6.8** — Implementar `GET /books` (público):
  - Filtrar por título, autor, género (todos opcionales)
  - Paginación con defaults: page=0, size=20
  - Devolver `Page<BookResponse>`
- [ ] **6.9** — Implementar `POST /reservations` (autenticado):
  - Extraer el userId del JWT (no del body)
  - Verificar que el libro existe y tiene copias disponibles
  - Verificar que el usuario no tenga ya una reservación activa del mismo libro
  - Decrementar `availableCopies` y crear la reservación
  - Devolver `ReservationResponse` con estado 201
- [ ] **6.10** — Implementar `POST /reviews` (autenticado):
  - Extraer el userId del JWT
  - Verificar que el libro existe
  - Verificar que el usuario no haya hecho review de ese libro (constraint único)
  - Crear el review
  - Devolver `ReviewResponse` con estado 201
- [ ] **6.11** — Implementar `GET /reviews?bookId=...` (público):
  - Listar reviews de un libro con paginación
  - Incluir datos básicos del usuario (username, no el hash de password)
  - Ordenar por fecha descendente (más recientes primero)
- [ ] **6.12** — Implementar endpoints adicionales útiles:
  - `GET /books/{id}` — Detalle de un libro con rating promedio
  - `GET /reservations/my` — Reservaciones del usuario autenticado
  - `DELETE /reservations/{id}` — Cancelar una reservación propia
- [ ] **6.13** — Probar todos los endpoints con Postman o curl. Crear una colección de Postman con todos los requests documentados, incluyendo el flujo de autenticación.

### Entregable de fase

Todos los endpoints del challenge funcionando con validación, paginación, manejo de errores consistente, y autenticación. Al completar esta fase, la Option 1 del challenge está completa.

---

## Fase 7: @Transactional y Consistencia de Datos

**Duración estimada:** 2–3 días

### ¿Por qué esta fase?

Operaciones como reservar un libro involucran múltiples escrituras. Sin transacciones, un fallo parcial deja la BD en estado inconsistente.

### Objetivos puntuales

- [ ] **7.1** — Estudiar qué es una transacción y sus propiedades ACID:
  - **Atomicidad:** Todo se ejecuta o nada se ejecuta
  - **Consistencia:** La BD pasa de un estado válido a otro estado válido
  - **Isolation:** Transacciones concurrentes no interfieren entre sí
  - **Durabilidad:** Una vez commitada, la transacción persiste aunque el servidor caiga
- [ ] **7.2** — Entender cómo Spring gestiona `@Transactional`:
  - Spring crea un **proxy** alrededor de la clase/método anotado
  - Al entrar al método, abre una transacción
  - Si el método termina sin excepción, hace commit
  - Si lanza una `RuntimeException` (unchecked), hace rollback automáticamente
  - Si lanza una `checked exception`, **no** hace rollback (a menos que configures `@Transactional(rollbackFor = Exception.class)`)
- [ ] **7.3** — Estudiar los niveles de **propagación**:
  - `REQUIRED` (default): Si hay una transacción activa, se une a ella; si no, crea una nueva
  - `REQUIRES_NEW`: Siempre crea una nueva transacción, suspendiendo la actual si existe (útil para audit logs que deben guardarse incluso si la transacción principal falla)
  - `MANDATORY`: Exige que ya exista una transacción; lanza excepción si no
  - `SUPPORTS`: Se une a la transacción existente o ejecuta sin transacción si no hay
  - `NOT_SUPPORTED`: Ejecuta sin transacción, suspendiendo la actual
- [ ] **7.4** — Aplicar `@Transactional` al método de reservación:
  ```java
  @Transactional
  public ReservationResponse createReservation(Long userId, Long bookId) {
      Book book = bookRepository.findById(bookId)
          .orElseThrow(() -> new BookNotFoundException(bookId));

      if (book.getAvailableCopies() <= 0) {
          throw new NoAvailableCopiesException(bookId);
      }

      // Verificar reservación duplicada
      reservationRepository.findByUserIdAndBookIdAndStatus(userId, bookId, Status.ACTIVE)
          .ifPresent(r -> { throw new DuplicateReservationException(userId, bookId); });

      book.setAvailableCopies(book.getAvailableCopies() - 1);
      bookRepository.save(book);

      Reservation reservation = new Reservation(userId, bookId, Status.ACTIVE, ...);
      return mapper.toResponse(reservationRepository.save(reservation));
  }
  ```
  Si `reservationRepository.save()` falla, el decremento de `availableCopies` se deshace automáticamente.
- [ ] **7.5** — Entender la trampa de **invocación interna**: `@Transactional` no funciona si llamas al método desde dentro de la misma clase, porque el proxy no intercepta llamadas internas. Ejemplo incorrecto:
  ```java
  public void methodA() {
      methodB(); // ¡La transacción de methodB NO se aplica!
  }

  @Transactional
  public void methodB() { ... }
  ```
  Solución: mover `methodB` a otra clase inyectada, o inyectar `self` (el propio bean).
- [ ] **7.6** — Usar `@Transactional(readOnly = true)` en todos los métodos de solo lectura (búsquedas, listados). Esto permite a Hibernate y al driver optimizar: no mantiene dirty checking, puede usar réplicas de lectura, y es una señal de intención clara.
- [ ] **7.7** — Estudiar **niveles de aislamiento** (isolation levels) y cuándo configurarlos:
  - `READ_COMMITTED` (default en PostgreSQL): No lees datos no commitados de otras transacciones
  - `REPEATABLE_READ`: Garantiza que si lees un dato dos veces en la misma transacción, obtienes el mismo resultado
  - `SERIALIZABLE`: Máximo aislamiento, las transacciones se ejecutan como si fueran secuenciales
  - Para el challenge, `READ_COMMITTED` es suficiente. Considera `SERIALIZABLE` si detectas race conditions en las reservaciones
- [ ] **7.8** — Manejar **concurrencia optimista** con `@Version`:
  ```java
  @Entity
  public class Book {
      @Version
      private Long version;
      // ...
  }
  ```
  Si dos requests intentan decrementar `availableCopies` simultáneamente, JPA detecta el conflicto y lanza `OptimisticLockException`. Puedes capturarla y reintentar o devolver 409 Conflict.
- [ ] **7.9** — Probar la transaccionalidad: hacer un request de reservación que falle a mitad de camino (simular una excepción) y verificar que la BD queda intacta.

### Entregable de fase

Todos los servicios con `@Transactional` aplicado correctamente. Pruebas que demuestran rollback en caso de fallo. Locking optimista en la entidad `Book`.

---

## Fase 8: Logging y Trazabilidad Contextual

**Duración estimada:** 3–4 días

### ¿Por qué esta fase?

El challenge evalúa observabilidad. En producción, sin buenos logs no puedes diagnosticar problemas. Con trazabilidad contextual, puedes seguir un request de principio a fin.

### Objetivos puntuales

#### Logging básico

- [ ] **8.1** — Entender la stack de logging de Spring Boot: SLF4J es la fachada (interfaz), Logback es la implementación (incluida por defecto). Nunca uses `System.out.println()` para logs.
- [ ] **8.2** — Usar el logger correctamente en cada clase:
  ```java
  // Opción 1: Manual
  private static final Logger log = LoggerFactory.getLogger(BookService.class);

  // Opción 2: Con Lombok
  @Slf4j
  public class BookService { ... }
  ```
- [ ] **8.3** — Conocer los niveles de log y cuándo usar cada uno:
  - `ERROR` — Algo falló y requiere atención (excepción inesperada, servicio externo caído)
  - `WARN` — Algo inesperado pero manejable (intento de reservar libro sin copias, token expirado)
  - `INFO` — Eventos de negocio significativos (reservación creada, usuario registrado)
  - `DEBUG` — Información detallada para desarrollo (parámetros de búsqueda, queries ejecutadas)
  - `TRACE` — Máximo detalle (raramente usado, para debugging profundo)
- [ ] **8.4** — Usar **placeholders** en lugar de concatenación:
  ```java
  // Correcto: el placeholder {} se reemplaza solo si el nivel está habilitado
  log.info("Reservation created: userId={}, bookId={}, reservationId={}", userId, bookId, reservationId);

  // Incorrecto: la concatenación se ejecuta siempre, incluso si INFO está deshabilitado
  log.info("Reservation created: userId=" + userId + ", bookId=" + bookId);
  ```

#### MDC — Trazabilidad contextual

- [ ] **8.5** — Estudiar qué es MDC (Mapped Diagnostic Context): es un mapa thread-local que agrega contexto a cada línea de log automáticamente. Permite rastrear todas las operaciones de un mismo request sin pasar el correlationId manualmente a cada método.
- [ ] **8.6** — Crear un filtro `MdcFilter` que se ejecuta en cada request:
  ```java
  @Component
  @Order(Ordered.HIGHEST_PRECEDENCE)
  public class MdcFilter extends OncePerRequestFilter {
      @Override
      protected void doFilterInternal(HttpServletRequest request,
                                       HttpServletResponse response,
                                       FilterChain chain) throws ServletException, IOException {
          try {
              String correlationId = Optional.ofNullable(request.getHeader("X-Correlation-ID"))
                  .orElse(UUID.randomUUID().toString().substring(0, 8));
              MDC.put("correlationId", correlationId);
              MDC.put("method", request.getMethod());
              MDC.put("uri", request.getRequestURI());
              // Agregar userId después de la autenticación si está disponible
              response.setHeader("X-Correlation-ID", correlationId);
              chain.doFilter(request, response);
          } finally {
              MDC.clear(); // CRÍTICO: limpiar para evitar memory leaks con thread pools
          }
      }
  }
  ```
- [ ] **8.7** — Configurar el patrón de log en `logback-spring.xml` para incluir el contexto:
  ```xml
  <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{correlationId}] [%X{userId:-anonymous}]
           %-5level %logger{36} - %msg%n</pattern>
  ```
  Resultado: `2025-03-15 14:30:22.123 [virtual-thread-1] [a1b2c3d4] [user42] INFO  BookService - Search executed: title=java, results=15`
- [ ] **8.8** — Agregar el userId al MDC después de la autenticación JWT exitosa, dentro del `JwtAuthenticationFilter`.

#### Logging de operaciones clave

- [ ] **8.9** — Agregar logs significativos (no excesivos) en cada servicio:
  - `BookService`: INFO al ejecutar búsqueda (con parámetros y cantidad de resultados), DEBUG con los IDs de resultados
  - `ReservationService`: INFO al crear/cancelar reservación, WARN cuando no hay copias disponibles
  - `ReviewService`: INFO al crear review, WARN cuando se detecta review duplicado
  - `AuthService`: INFO al registrar/logear usuario (sin loggear contraseñas nunca), WARN en intentos de login fallidos
- [ ] **8.10** — Configurar niveles de log por ambiente:
  ```yaml
  # application-dev.yml
  logging:
    level:
      com.library: DEBUG
      org.hibernate.SQL: DEBUG

  # application-prod.yml
  logging:
    level:
      com.library: INFO
      org.hibernate.SQL: WARN
  ```

#### Spring Boot Actuator

- [ ] **8.11** — Agregar la dependencia `spring-boot-starter-actuator`. Configurar endpoints:
  ```yaml
  management:
    endpoints:
      web:
        exposure:
          include: health, info, metrics, prometheus
    endpoint:
      health:
        show-details: when-authorized
  ```
- [ ] **8.12** — Implementar un **health check custom** para verificar la conexión a BD y RabbitMQ:
  ```java
  @Component
  public class DatabaseHealthIndicator implements HealthIndicator {
      @Override
      public Health health() {
          // Verificar conexión a BD
          return Health.up().withDetail("database", "PostgreSQL").build();
      }
  }
  ```
- [ ] **8.13** — Crear **métricas custom** con Micrometer para las operaciones del challenge:
  ```java
  @Service
  public class ReservationService {
      private final Counter reservationCounter;
      private final Timer searchTimer;

      public ReservationService(MeterRegistry registry) {
          this.reservationCounter = Counter.builder("library.reservations.created")
              .description("Total reservations created")
              .register(registry);
          this.searchTimer = Timer.builder("library.books.search.time")
              .description("Book search duration")
              .register(registry);
      }
  }
  ```
- [ ] **8.14** — Verificar que `/actuator/health` devuelve `UP`, y que `/actuator/metrics` muestra las métricas custom.

### Entregable de fase

Logs estructurados con correlationId en cada request. Actuator con health check y métricas custom. Capacidad de rastrear un request completo de principio a fin en los logs.

---

## Fase 9: Event-Driven Architecture con RabbitMQ

**Duración estimada:** 1.5 semanas

### ¿Por qué esta fase?

Esta fase implementa la Option 3 del challenge. Transforma operaciones síncronas en eventos asíncronos, permitiendo escalabilidad, auditoría y desacoplamiento.

### Objetivos puntuales

#### RabbitMQ — Conceptos fundamentales

- [ ] **9.1** — Instalar RabbitMQ con Docker:
  ```bash
  docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
  ```
  Acceder al panel de administración en `http://localhost:15672` (guest/guest). Explorar la interfaz: exchanges, queues, bindings, connections.
- [ ] **9.2** — Estudiar el modelo de messaging de RabbitMQ:
  - **Producer** → publica un mensaje a un **Exchange**
  - **Exchange** → enruta el mensaje a una o más **Queues** según reglas de **Binding**
  - **Consumer** → lee de una Queue y procesa el mensaje
  - Los mensajes no van directo a la queue; siempre pasan por un exchange
- [ ] **9.3** — Estudiar los tipos de Exchange:
  - **Direct:** Enruta por routing key exacto. Ejemplo: routing key `reservation.created` va solo a la queue bindeada con ese key exacto.
  - **Topic:** Enruta por patrón de routing key con wildcards. Ejemplo: `reservation.*` matchea `reservation.created` y `reservation.cancelled`. `#` matchea cualquier cosa.
  - **Fanout:** Broadcast, ignora el routing key. Envía a todas las queues bindeadas. Útil para notificaciones donde todos los consumers necesitan el mensaje.
  - **Headers:** Enruta por headers del mensaje (raramente usado).
  - Para el challenge usa **Topic Exchange**: te da flexibilidad para filtrar eventos por tipo.
- [ ] **9.4** — Estudiar conceptos de confiabilidad:
  - **Acknowledgments (acks):** El consumer confirma que procesó el mensaje. Si no lo confirma, RabbitMQ lo re-encola.
  - **Persistent messages:** Marcar mensajes como duraderos para que sobrevivan un restart de RabbitMQ.
  - **Dead Letter Queue (DLQ):** Queue a donde van los mensajes que fallan repetidamente. Esencial para debugging.

#### Spring AMQP — Integración

- [ ] **9.5** — Agregar la dependencia `spring-boot-starter-amqp`. Configurar la conexión:
  ```yaml
  spring:
    rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
  ```
- [ ] **9.6** — Configurar exchanges, queues y bindings como beans:
  ```java
  @Configuration
  public class RabbitMQConfig {

      public static final String LIBRARY_EXCHANGE = "library.events";
      public static final String NOTIFICATION_QUEUE = "library.notifications";
      public static final String ANALYTICS_QUEUE = "library.analytics";
      public static final String AUDIT_QUEUE = "library.audit";

      @Bean
      public TopicExchange libraryExchange() {
          return new TopicExchange(LIBRARY_EXCHANGE, true, false);
      }

      @Bean
      public Queue notificationQueue() {
          return QueueBuilder.durable(NOTIFICATION_QUEUE)
              .withArgument("x-dead-letter-exchange", "library.dlx")
              .build();
      }

      @Bean
      public Binding notificationBinding(Queue notificationQueue, TopicExchange exchange) {
          return BindingBuilder.bind(notificationQueue).to(exchange).with("reservation.*");
      }

      // Repeat for analytics (*.created) and audit (#, captures everything)
  }
  ```
- [ ] **9.7** — Configurar la serialización JSON para los mensajes:
  ```java
  @Bean
  public Jackson2JsonMessageConverter messageConverter() {
      return new Jackson2JsonMessageConverter();
  }

  @Bean
  public RabbitTemplate rabbitTemplate(ConnectionFactory factory,
                                        Jackson2JsonMessageConverter converter) {
      RabbitTemplate template = new RabbitTemplate(factory);
      template.setMessageConverter(converter);
      return template;
  }
  ```

#### Diseño de eventos

- [ ] **9.8** — Definir los eventos como Records usando sealed interfaces (Java 21):
  ```java
  public sealed interface LibraryEvent permits
      BookReservedEvent, ReservationCancelledEvent, ReviewSubmittedEvent {

      String eventType();
      Instant timestamp();
  }

  public record BookReservedEvent(
      Long reservationId, Long userId, Long bookId,
      String bookTitle, Instant timestamp
  ) implements LibraryEvent {
      public String eventType() { return "BOOK_RESERVED"; }
  }

  public record ReservationCancelledEvent(
      Long reservationId, Long userId, Long bookId,
      Instant timestamp
  ) implements LibraryEvent {
      public String eventType() { return "RESERVATION_CANCELLED"; }
  }

  public record ReviewSubmittedEvent(
      Long reviewId, Long userId, Long bookId,
      int rating, Instant timestamp
  ) implements LibraryEvent {
      public String eventType() { return "REVIEW_SUBMITTED"; }
  }
  ```
- [ ] **9.9** — Crear un `EventPublisher` que encapsula la publicación:
  ```java
  @Service
  @Slf4j
  public class EventPublisher {
      private final RabbitTemplate rabbitTemplate;

      public void publish(LibraryEvent event) {
          String routingKey = switch (event) {
              case BookReservedEvent e -> "reservation.created";
              case ReservationCancelledEvent e -> "reservation.cancelled";
              case ReviewSubmittedEvent e -> "review.created";
          };
          log.info("Publishing event: type={}, routingKey={}", event.eventType(), routingKey);
          rabbitTemplate.convertAndSend(RabbitMQConfig.LIBRARY_EXCHANGE, routingKey, event);
      }
  }
  ```

#### Integración con los servicios existentes

- [ ] **9.10** — Modificar `ReservationService` para publicar eventos después de cada operación exitosa:
  ```java
  @Transactional
  public ReservationResponse createReservation(Long userId, Long bookId) {
      // ... lógica existente de reservación ...
      Reservation saved = reservationRepository.save(reservation);

      // Publicar evento DESPUÉS del commit
      eventPublisher.publish(new BookReservedEvent(
          saved.getId(), userId, bookId, book.getTitle(), Instant.now()
      ));

      return mapper.toResponse(saved);
  }
  ```
  **Nota importante:** Idealmente el evento se publica DESPUÉS del commit de la transacción para evitar publicar eventos de operaciones que luego hacen rollback. Estudiar `@TransactionalEventListener(phase = AFTER_COMMIT)` como alternativa más robusta.
- [ ] **9.11** — Hacer lo mismo en `ReviewService` y en la cancelación de reservaciones.

#### Consumers

- [ ] **9.12** — Crear el consumer de **notificaciones**:
  ```java
  @Component
  @Slf4j
  public class NotificationConsumer {

      @RabbitListener(queues = RabbitMQConfig.NOTIFICATION_QUEUE)
      public void handleReservationEvent(LibraryEvent event) {
          switch (event) {
              case BookReservedEvent e ->
                  log.info("NOTIFICATION: User {} reserved '{}'. Confirmation sent.",
                      e.userId(), e.bookTitle());
              case ReservationCancelledEvent e ->
                  log.info("NOTIFICATION: Reservation {} cancelled for user {}.",
                      e.reservationId(), e.userId());
              default -> log.warn("Unhandled event type: {}", event.eventType());
          };
          // En producción aquí enviarías un email o push notification
      }
  }
  ```
- [ ] **9.13** — Crear el consumer de **analytics** que actualiza contadores:
  ```java
  @Component
  @Slf4j
  public class AnalyticsConsumer {

      private final MeterRegistry meterRegistry;

      @RabbitListener(queues = RabbitMQConfig.ANALYTICS_QUEUE)
      public void handleEvent(LibraryEvent event) {
          meterRegistry.counter("library.events.processed", "type", event.eventType()).increment();
          log.info("ANALYTICS: Processed event type={}", event.eventType());
      }
  }
  ```

#### Event Store y Audit Log

- [ ] **9.14** — Crear una tabla `domain_events` (migración Flyway `V7__create_domain_events_table.sql`):
  ```sql
  CREATE TABLE domain_events (
      id BIGSERIAL PRIMARY KEY,
      event_type VARCHAR(50) NOT NULL,
      aggregate_type VARCHAR(50) NOT NULL,
      aggregate_id BIGINT NOT NULL,
      payload JSONB NOT NULL,
      user_id BIGINT,
      created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
  );

  CREATE INDEX idx_events_type ON domain_events (event_type);
  CREATE INDEX idx_events_aggregate ON domain_events (aggregate_type, aggregate_id);
  CREATE INDEX idx_events_created ON domain_events (created_at);
  ```
- [ ] **9.15** — Crear el consumer de **audit** que persiste cada evento:
  ```java
  @Component
  public class AuditConsumer {

      @RabbitListener(queues = RabbitMQConfig.AUDIT_QUEUE)
      @Transactional(propagation = Propagation.REQUIRES_NEW)  // Transacción independiente
      public void handleEvent(LibraryEvent event) {
          DomainEvent domainEvent = DomainEvent.from(event);
          domainEventRepository.save(domainEvent);
      }
  }
  ```
- [ ] **9.16** — Implementar endpoint de consulta de eventos para auditoría:
  - `GET /events?type=BOOK_RESERVED&from=2025-01-01&to=2025-03-15` — Filtrar eventos por tipo y rango de fecha
  - `GET /events?aggregateId=42&aggregateType=RESERVATION` — Ver todos los eventos de una reservación específica
- [ ] **9.17** — Implementar **replay de eventos**: un endpoint o comando que lee los eventos del store y reconstruye el estado. Ejemplo: contar cuántas reservaciones tuvo cada libro sumando los eventos `BOOK_RESERVED` y restando `RESERVATION_CANCELLED`.

#### Dead Letter Queue y resiliencia

- [ ] **9.18** — Configurar una DLQ:
  ```java
  @Bean
  public TopicExchange deadLetterExchange() {
      return new TopicExchange("library.dlx");
  }

  @Bean
  public Queue deadLetterQueue() {
      return QueueBuilder.durable("library.dlq").build();
  }
  ```
- [ ] **9.19** — Configurar retry: Spring AMQP puede reintentar un mensaje N veces antes de enviarlo a la DLQ:
  ```yaml
  spring:
    rabbitmq:
      listener:
        simple:
          retry:
            enabled: true
            initial-interval: 1000
            max-attempts: 3
            multiplier: 2.0
  ```
- [ ] **9.20** — Crear un endpoint `GET /events/dead-letter` para inspeccionar mensajes fallidos (para debugging).

#### Observabilidad de eventos

- [ ] **9.21** — Agregar métricas de eventos con Micrometer:
  - Contador de eventos publicados por tipo
  - Contador de eventos consumidos por tipo y consumer
  - Timer de latencia de procesamiento (timestamp del evento vs timestamp de consumo)
  - Contador de eventos fallidos / reintentados
- [ ] **9.22** — Agregar logs con correlationId en los consumers (propagar el correlationId como header del mensaje para trazabilidad end-to-end).

### Entregable de fase

Eventos publicados en cada operación de negocio. Tres consumers funcionando (notificaciones, analytics, audit). Event store con endpoint de consulta y replay. DLQ configurada. Métricas de eventos visibles en Actuator.

---

## Fase 10: Testing

**Duración estimada:** 1 semana

### ¿Por qué esta fase?

El challenge evalúa calidad de código. Sin tests, refactorizar es arriesgado. Los tests también sirven como documentación del comportamiento esperado.

### Objetivos puntuales

#### Tests unitarios

- [ ] **10.1** — Configurar JUnit 5 (incluido por defecto) y Mockito. Estudiar las anotaciones clave:
  - `@ExtendWith(MockitoExtension.class)` — Habilita Mockito en la clase
  - `@Mock` — Crea un mock del repositorio/dependencia
  - `@InjectMocks` — Inyecta los mocks en el servicio bajo test
  - `when(...).thenReturn(...)` — Configura el comportamiento del mock
  - `verify(mock).method(...)` — Verifica que se llamó al mock
  - `assertThrows(...)` — Verifica que se lanza una excepción
- [ ] **10.2** — Escribir tests para `BookService`:
  - Test: buscar libros con filtros devuelve resultados paginados
  - Test: buscar con filtros vacíos devuelve todos los libros
- [ ] **10.3** — Escribir tests para `ReservationService`:
  - Test: reservar libro con copias disponibles → éxito, decrementa copias
  - Test: reservar libro sin copias → lanza `NoAvailableCopiesException`
  - Test: reservar libro que no existe → lanza `BookNotFoundException`
  - Test: reservar libro ya reservado por el mismo usuario → lanza `DuplicateReservationException`
  - Test: cancelar reservación propia → éxito, incrementa copias
  - Test: cancelar reservación de otro usuario → lanza excepción
- [ ] **10.4** — Escribir tests para `ReviewService`:
  - Test: crear review válido → éxito
  - Test: crear review duplicado → lanza `DuplicateReviewException`
  - Test: crear review con rating fuera de rango → falla validación
- [ ] **10.5** — Escribir tests para `JwtService`:
  - Test: generar token y extraer username → coinciden
  - Test: token expirado → `isTokenValid` devuelve false
  - Test: token con firma inválida → lanza excepción

#### Tests de integración

- [ ] **10.6** — Agregar Testcontainers: dependencias `testcontainers`, `postgresql`, `rabbitmq`. Testcontainers levanta contenedores Docker reales para los tests:
  ```java
  @SpringBootTest
  @Testcontainers
  class ReservationIntegrationTest {

      @Container
      static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

      @Container
      static RabbitMQContainer rabbit = new RabbitMQContainer("rabbitmq:3-management");

      @DynamicPropertySource
      static void configureProperties(DynamicPropertyRegistry registry) {
          registry.add("spring.datasource.url", postgres::getJdbcUrl);
          registry.add("spring.rabbitmq.host", rabbit::getHost);
          registry.add("spring.rabbitmq.port", rabbit::getAmqpPort);
      }
  }
  ```
- [ ] **10.7** — Escribir tests de integración con `MockMvc` para cada endpoint:
  - Test flujo completo: registrar usuario → login → reservar libro → verificar copia decrementada
  - Test: crear review con JWT válido → 201
  - Test: crear review sin JWT → 401
  - Test: buscar libros sin autenticación → 200 (público)
- [ ] **10.8** — Escribir tests de integración para el flujo de eventos:
  - Test: al crear reservación, un mensaje llega a la queue de audit
  - Test: el consumer persiste el evento en la tabla `domain_events`

#### Tests de repositorio

- [ ] **10.9** — Usar `@DataJpaTest` para testear queries custom:
  ```java
  @DataJpaTest
  @AutoConfigureTestDatabase(replace = Replace.NONE)
  @Testcontainers
  class BookRepositoryTest {
      @Test
      void shouldFindBooksByTitleContaining() { ... }

      @Test
      void shouldFilterByGenre() { ... }
  }
  ```

#### Cobertura

- [ ] **10.10** — Configurar JaCoCo para medir cobertura de tests. Apuntar a al menos 70% de cobertura en las clases de servicio. No buscar 100%, es contraproducente.

### Entregable de fase

Suite de tests unitarios para todos los servicios. Tests de integración con Testcontainers para flujos críticos. Reporte de cobertura con JaCoCo.

---

## Fase 11: Dockerización y Entrega

**Duración estimada:** 3–4 días

### ¿Por qué esta fase?

El challenge pide "host your solution in a public Git repository" con instrucciones de setup. Docker garantiza que cualquier evaluador pueda ejecutar todo con un solo comando.

### Objetivos puntuales

#### Dockerfile

- [ ] **11.1** — Crear un `Dockerfile` multi-stage:
  ```dockerfile
  # Etapa 1: Compilación
  FROM eclipse-temurin:21-jdk AS builder
  WORKDIR /app
  COPY pom.xml .
  COPY src ./src
  RUN ./mvnw clean package -DskipTests

  # Etapa 2: Ejecución
  FROM eclipse-temurin:21-jre-alpine
  WORKDIR /app
  COPY --from=builder /app/target/*.jar app.jar
  EXPOSE 8080
  ENTRYPOINT ["java", "-jar", "app.jar"]
  ```
  **¿Por qué multi-stage?** La imagen final no incluye Maven ni el JDK completo, solo el JRE. Resultado: imagen mucho más pequeña (~200MB vs ~800MB).
- [ ] **11.2** — Crear `.dockerignore` para excluir archivos innecesarios (`.git`, `target/`, `.idea/`).

#### Docker Compose

- [ ] **11.3** — Crear `docker-compose.yml` con los 3 servicios:
  ```yaml
  version: '3.8'
  services:
    postgres:
      image: postgres:16-alpine
      environment:
        POSTGRES_DB: library
        POSTGRES_USER: library_user
        POSTGRES_PASSWORD: library_pass
      ports:
        - "5432:5432"
      volumes:
        - postgres_data:/var/lib/postgresql/data

    rabbitmq:
      image: rabbitmq:3-management-alpine
      ports:
        - "5672:5672"
        - "15672:15672"

    app:
      build: .
      ports:
        - "8080:8080"
      environment:
        SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/library
        SPRING_RABBITMQ_HOST: rabbitmq
        SPRING_PROFILES_ACTIVE: prod
      depends_on:
        - postgres
        - rabbitmq

  volumes:
    postgres_data:
  ```
- [ ] **11.4** — Agregar health checks en compose para asegurar que PostgreSQL y RabbitMQ estén listos antes de arrancar la app:
  ```yaml
  postgres:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U library_user"]
      interval: 5s
      retries: 5
  app:
    depends_on:
      postgres:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
  ```
- [ ] **11.5** — Verificar que `docker-compose up` levanta todo correctamente y la app es accesible en `http://localhost:8080`.

#### Documentación (README)

- [ ] **11.6** — Escribir un README completo con las siguientes secciones:
  - **Descripción:** Qué es el proyecto y qué option del challenge implementa
  - **Arquitectura:** Diagrama de la arquitectura (puede ser ASCII art o imagen). Explicar la estructura por features, el flujo de eventos, y cómo interactúan los componentes
  - **Tecnologías:** Java 21, Spring Boot 3.x, PostgreSQL, RabbitMQ, Flyway, JWT, Docker
  - **Decisiones de diseño:** Por qué estructura por features, por qué Topic Exchange, por qué locking optimista, decisiones de normalización/desnormalización, etc.
  - **Cómo ejecutar:** Instrucciones paso a paso con Docker Compose
  - **Endpoints:** Tabla con todos los endpoints, métodos, auth requerida, y ejemplo de request/response
  - **Ejemplos de uso:** Flujo completo con curl:
    1. Registrar usuario
    2. Login y obtener token
    3. Buscar libros
    4. Reservar un libro
    5. Dejar un review
    6. Consultar eventos de auditoría
- [ ] **11.7** — Crear una **colección de Postman** exportada como JSON en el repositorio, o un script `demo.sh` con requests curl que demuestren todas las funcionalidades.

#### Repositorio Git

- [ ] **11.8** — Crear el repositorio en GitHub/GitLab. Asegurar que el `.gitignore` excluye archivos de IDE, `target/`, archivos `.env` con secretos.
- [ ] **11.9** — Escribir commits limpios y descriptivos. Considerar organizar por fases o features (ej. `feat: add book search with specifications`, `feat: add JWT authentication`, `feat: add RabbitMQ event publishing`).
- [ ] **11.10** — Opcional: Agregar un GitHub Action de CI que ejecute los tests automáticamente en cada push.

### Entregable de fase

Proyecto completo en un repositorio público. `docker-compose up` levanta todo. README con documentación completa. Script de demo o colección Postman.

---

## Recursos Generales

### Documentación oficial
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Spring AMQP Reference](https://docs.spring.io/spring-amqp/docs/current/reference/html/)
- [Flyway Documentation](https://flywaydb.org/documentation/)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials)
- [Java 21 JEP Index](https://openjdk.org/projects/jdk/21/)

### Cursos y tutoriales recomendados
- Spring Boot: [Baeldung](https://www.baeldung.com/) — Referencia casi obligatoria para cualquier tema de Spring
- JPA: "Spring Data JPA Tutorial" en Baeldung
- Security: "Spring Security JWT Tutorial" en Baeldung
- RabbitMQ: Tutoriales oficiales de RabbitMQ (tienen versión Java/Spring)
- Testing: "Guide to Testcontainers" en Baeldung

### Herramientas
- [start.spring.io](https://start.spring.io/) — Generador de proyectos Spring Boot
- [Postman](https://www.postman.com/) — Testing de APIs
- [DBeaver](https://dbeaver.io/) — Cliente de base de datos
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) — Contenedores

---

## Checklist Final del Challenge

- [ ] `GET /books` con búsqueda por título, autor, género y paginación
- [ ] `POST /reservations` con validación y transaccionalidad
- [ ] `POST /reviews` y `GET /reviews?bookId=...`
- [ ] Base de datos relacional con Users, Books, Reservations, Reviews
- [ ] Java 21 features: Records, Pattern Matching, Sealed Classes, Virtual Threads
- [ ] Logging con correlationId y MDC
- [ ] Métricas con Micrometer + health check con Actuator
- [ ] Event-driven: eventos para reservaciones, reviews, cancelaciones
- [ ] Event store con append-only log y capacidad de replay
- [ ] Procesamiento asíncrono con RabbitMQ (notificaciones, analytics, audit)
- [ ] Monitoreo de latencia y throughput de eventos
- [ ] Repositorio público con README, instrucciones de setup, y demo
- [ ] Docker Compose para levantar todo con un comando
