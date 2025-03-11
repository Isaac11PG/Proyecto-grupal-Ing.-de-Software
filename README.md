# Spring Security Project

Este proyecto implementa un sistema de autenticación y autorización utilizando Spring Security, con soporte para múltiples roles y permisos. Incluye endpoints para registro y login de usuarios, así como configuraciones para la gestión de seguridad y persistencia de datos.

## Características Principales

- **Autenticación y Autorización**: Sistema completo basado en JWT (JSON Web Tokens)
- **Gestión de Roles**: Implementación de RBAC (Role-Based Access Control)
- **Persistencia**: Configuración para MySQL
- **API RESTful**: Endpoints para registro, login y gestión de recursos protegidos

## Requisitos Previos

- Java 17 o superior
- Maven 3.6.0 o superior
- MySQL 8.0 o superior
- Postman (opcional, para pruebas de API)

## Estructura del Proyecto

```
src/
├── main/
│   ├── java/com/example/securitydemo/
│   │   ├── config/
│   │   │   ├── SecurityConfig.java
│   │   │   └── JwtConfig.java
│   │   ├── controller/
│   │   │   ├── AuthController.java
│   │   │   └── ResourceController.java
│   │   ├── model/
│   │   │   ├── User.java
│   │   │   └── Role.java
│   │   ├── repository/
│   │   │   ├── UserRepository.java
│   │   │   └── RoleRepository.java
│   │   ├── service/
│   │   │   ├── UserService.java
│   │   │   └── JwtService.java
│   │   └── SecurityDemoApplication.java
│   └── resources/
│       ├── application.properties
│       └── data.sql
└── test/
    └── java/com/example/securitydemo/
        └── SecurityDemoApplicationTests.java
```

## Instalación y Ejecución

### Paso 1: Clonar el Repositorio

```bash
git clone https://github.com/usuario/spring-security-demo.git
cd spring-security-demo
```

### Paso 2: Configurar la Base de Datos

Crea una base de datos MySQL:

```sql
CREATE DATABASE security_demo;
```

Asegúrate de que la configuración en `application.properties` coincida con tu entorno:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/security_demo
spring.datasource.username=tu_usuario
spring.datasource.password=tu_contraseña
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

### Paso 3: Compilar el Proyecto

```bash
mvn clean install
```

### Paso 4: Ejecutar la Aplicación

```bash
mvn spring-boot:run
```

La aplicación estará disponible en `http://localhost:8080`

## Endpoints API

### Autenticación

#### Registro de Usuario
```
POST /api/auth/register
Content-Type: application/json

{
  "username": "nuevo_usuario",
  "password": "contraseña_segura",
  "email": "usuario@ejemplo.com",
  "name": "Nombre Completo"
}
```

#### Login
```
POST /api/auth/login
Content-Type: application/json

{
  "username": "usuario",
  "password": "contraseña"
}
```

Respuesta:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "type": "Bearer",
  "username": "usuario",
  "roles": ["ROLE_USER"]
}
```

### Recursos Protegidos

#### Acceso a Recurso de Usuario
```
GET /api/resources/user
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Acceso a Recurso de Administrador
```
GET /api/resources/admin
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Implementación de Seguridad

### SecurityConfig

El proyecto utiliza Spring Security con JWT para la autenticación y autorización:

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationFilter jwtAuthFilter;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/resources/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/resources/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }
    
    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### Sistema de Roles y Permisos

El control de acceso basado en roles se implementa mediante:

1. **A nivel de configuración**: Usando `antMatchers` para proteger rutas específicas.
2. **A nivel de método**: Usando anotaciones como `@PreAuthorize` y `@Secured`.

Ejemplo de uso de anotaciones de seguridad:

```java
@RestController
@RequestMapping("/api/resources")
public class ResourceController {

    @GetMapping("/public")
    public String publicResource() {
        return "Este recurso es público";
    }

    @GetMapping("/user")
    @PreAuthorize("hasRole('USER') or hasRole('ADMIN')")
    public String userResource() {
        return "Este recurso requiere rol de USUARIO";
    }

    @GetMapping("/admin")
    @PreAuthorize("hasRole('ADMIN')")
    public String adminResource() {
        return "Este recurso requiere rol de ADMINISTRADOR";
    }
    
    @GetMapping("/manager")
    @Secured("ROLE_MANAGER")
    public String managerResource() {
        return "Este recurso requiere rol de MANAGER";
    }
}
```

## Historial de Commits (Ejemplo)

El proyecto sigue convenciones para mensajes de commit:

- `[feat]: implementación inicial de autenticación con JWT`
- `[fix]: corregido problema con validación de tokens expirados`
- `[refactor]: mejorada la estructura de permisos de usuarios`
- `[docs]: actualizada documentación de API en README`
- `[test]: añadidos tests para AuthController`

## Pruebas

Para probar los endpoints, puedes usar Postman o curl:

1. Registra un nuevo usuario
2. Obtén un token JWT mediante login
3. Usa el token en el header `Authorization` para acceder a recursos protegidos

Ejemplo con curl:

```bash
# Registro
curl -X POST http://localhost:8080/api/auth/register -H "Content-Type: application/json" -d '{"username":"test","password":"password","email":"test@example.com"}'

# Login
curl -X POST http://localhost:8080/api/auth/login -H "Content-Type: application/json" -d '{"username":"test","password":"password"}'

# Acceso a recurso protegido
curl -X GET http://localhost:8080/api/resources/user -H "Authorization: Bearer TU_TOKEN_JWT"
```

##CAPTURAS DE PANTALLA
![image](https://github.com/user-attachments/assets/b64305e1-3cac-4d6a-949f-331d160a1c2a)
![image](https://github.com/user-attachments/assets/73c84241-4572-4e1f-accc-bf624952ab92)
![image](https://github.com/user-attachments/assets/54e5cc09-ca08-442e-baf4-bf35126a807a)
![image](https://github.com/user-attachments/assets/21bc9943-2b00-4855-adb4-e4153a873327)
![image](https://github.com/user-attachments/assets/c80aef3c-5424-42d8-a18b-58d7757baf65)






