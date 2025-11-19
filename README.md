## Flujo de autenticación y autorización

El proceso de autenticación y autorización en el sistema funciona de la siguiente manera:

1. El usuario inicia sesión enviando sus credenciales al microservicio de autenticación.
2. El microservicio valida las credenciales y, si son correctas, genera y devuelve un JWT que incluye los permisos (`authorities`) del usuario.
3. Cuando el usuario realiza una petición a otro endpoint, el API Gateway es el encargado de validar el JWT.
4. El API Gateway verifica si el JWT es válido y si el usuario tiene el permiso necesario para acceder al endpoint solicitado.

### Diagrama del flujo

```
┌─────────┐      Login       ┌─────────────────────┐
│ Usuario │ ──────────────▶ │ MSAuthentication     │
└─────────┘                  └─────────────────────┘
	▲                          │
	│────── Devuelve JWT ──────┘

┌─────────┐   Solicita acceso (con JWT)   ┌───────────────┐
│ Usuario │ ────────────────────────────▶ │ API Gateway   │
└─────────┘                               └───────────────┘
                                                │
                                                │ Valida JWT y authorities
                                                ▼
                                        ┌────────────────┐
                                        │ Servicio       │
                                        │ Destino        │
                                        └────────────────┘
```

# MSAuthentication

Microservicio de autenticación para el proyecto Arka, implementado en Java con Spring Boot. Proporciona APIs para la gestión de usuarios, autenticación, autorización y manejo de tokens JWT.

## Características principales
- Registro y autenticación de usuarios
- Validación y actualización de información de usuario
- Generación y validación de tokens JWT
- Integración con Spring Security
- Configuración centralizada con Spring Cloud Config
- Documentación de APIs con OpenAPI/Swagger

## Tecnologías utilizadas
Las principales tecnologías utilizadas en este microservicio son:

- **Java 21**
- **Spring Boot 3.5.6**
- **Spring Security**
- **Spring Data JPA**
- **PostgreSQL**
- **Spring Cloud Netflix Eureka** (descubrimiento de servicios)
- **Spring Cloud Config** (configuración centralizada)
- **Swagger/OpenAPI** (documentación de APIs)

## Tipos de usuarios
El sistema define los siguientes tipos de usuarios:


Cada tipo de usuario tiene diferentes niveles de acceso y permisos dentro del sistema.


## Permisos por tipo de usuario
Los permisos se asignan a cada usuario y se cargan en el campo `authorities` para la autenticación y autorización. A continuación se muestra qué permisos tiene cada tipo de usuario:

| Permiso         | SuperAdmin | Admin | Customer | Descripción                                      |
|-----------------|:---------:|:-----:|:--------:|--------------------------------------------------|
| Inventario:Crud |    ✔️     |  ✔️   |          | Crear y actualizar inventario                    |
| CreateAdmin     |    ✔️     |       |          | Crear administradores (solo SuperAdmin)          |
| Cart:Crud       |    ✔️     |  ✔️   |   ✔️     | Crear y actualizar mi carrito de compras |
| CartAdmin:Crud  |    ✔️     |  ✔️   |          | CRUD del lado del administrador                  |
| Orden:Crud      |    ✔️     |   ✔️  |   ✔️     | actualizar mi orden de compra y finalizar mi compra |
| UpdateUserInfo  |    ✔️     |  ✔️   |   ✔️     | Actualizar información del usuario               |
| OrdenAdmin:Crud |    ✔️     |  ✔️   |          | Actualizar estados de las ordenes de compra 

### Ejemplo de authorities por usuario

- **SuperAdmin**: `["Inventario:Crud", "CreateAdmin", "Cart:Crud", "CartAdmin:Crud", "Orden:Crud", "UpdateUserInfo", "OrdenAdmin:Crud"]`
- **Admin**: `["Inventario:Crud", "Cart:Crud", "CartAdmin:Crud", "UpdateUserInfo", "OrdenAdmin:Crud"]`
- **Customer**: `["Cart:Crud", "Orden:Crud", "UpdateUserInfo"]`

Estos permisos se cargan en el campo `authorities` del usuario y se utilizan para controlar el acceso a los diferentes endpoints del sistema.

## Ejemplo de JWT para SuperAdmin

```json
{
	"sub": "superadmin@arka.com",
	"authorities": "Cart:Crud,CartAdmin:Crud,CreateAdmin,Inventario:Crud,Orden:Crud,OrdenAdmin:Crud,UpdateUserInfo",
	"id": 1,
	"iat": 1763589716,
	"exp": 1763593316
}
```

Este JWT contiene los claims principales, incluyendo el correo del usuario, los permisos (authorities), el identificador, y las fechas de emisión y expiración.


## Ejemplo de configuración en API Gateway

```yaml
# CREAR ADMIN - Requiere permiso usuarios:crear_admin
- id: auth-create-admin
	uri: http://localhost:8098
	predicates:
		- Path=/api/v1/gateway/auth/createadmin
		- Method=POST
	filters:
		- AuthorizationFilter=CreateAdmin
		- RewritePath=/api/v1/gateway/auth/createadmin, /api/auth/createadmin
```

Este ejemplo muestra cómo el API Gateway valida el permiso `CreateAdmin` antes de permitir el acceso al endpoint para crear administradores.


## Ejecución
Para ejecutar el microservicio:
```sh
./gradlew bootRun
```

## Documentación de la API
La documentación interactiva está disponible en:

- [Swagger UI](http://localhost:8098/swagger-ui.html)




