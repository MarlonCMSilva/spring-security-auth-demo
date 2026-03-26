# Spring Security Auth Demo

Este projeto é uma demonstração de uma aplicação Spring Boot que implementa autenticação e autorização utilizando Spring Security 6, OAuth2 Authorization Server e OAuth2 Resource Server. Ele exemplifica um fluxo de autenticação com um Custom Password Grant Type e a emissão de tokens JWT para acesso a recursos protegidos.

## Funcionalidades

*   **Autenticação de Usuários:** Permite que usuários se autentiquem utilizando um `Custom Password Grant Type` para obter tokens de acesso.
*   **Autorização Baseada em Roles:** Proteção de endpoints da API com base em roles (`ROLE_ADMIN`, `ROLE_OPERATOR`).
*   **Servidor de Autorização OAuth2:** Configuração de um servidor de autorização para emissão de tokens.
*   **Servidor de Recursos OAuth2:** Proteção de recursos da API utilizando tokens JWT.
*   **Gerenciamento de Produtos:** Endpoints para listar, buscar e cadastrar produtos, com diferentes níveis de acesso.
*   **Banco de Dados em Memória:** Utiliza H2 Database para persistência de dados em ambiente de desenvolvimento/teste.

## Tecnologias Utilizadas

*   **Java 17**
*   **Spring Boot 3.1.0**
*   **Spring Security 6**
*   **Spring OAuth2 Authorization Server**
*   **Spring OAuth2 Resource Server**
*   **Spring Data JPA**
*   **H2 Database**
*   **Maven**

## Arquitetura de Segurança

O projeto implementa uma arquitetura de segurança baseada em OAuth2, com as seguintes características:

*   **OAuth2 Authorization Server:** Responsável por gerenciar clientes, emitir tokens de acesso e refresh tokens. Utiliza um `RegisteredClientRepository` em memória para um cliente pré-configurado.
*   **OAuth2 Resource Server:** Protege os endpoints da API, validando os tokens JWT recebidos nas requisições. A autorização é feita através de `JwtAuthenticationConverter` que extrai as roles do token.
*   **Custom Password Grant Type:** Foi implementado um `CustomPasswordAuthenticationConverter` e um `CustomPasswordAuthenticationProvider` para permitir a autenticação de usuários diretamente com username e password no endpoint `/oauth2/token`.
*   **JWT (JSON Web Tokens):** Os tokens de acesso são JWTs auto-contidos, assinados com chaves RSA, contendo informações sobre o usuário e suas permissões (`authorities`).

## Configuração e Execução

### Pré-requisitos

*   Java Development Kit (JDK) 17 ou superior
*   Maven 3.x

### Clonar o Repositório

```bash
git clone <URL_DO_SEU_REPOSITORIO>
cd spring-security-auth-demo
```

### Configuração do `application.properties`

O arquivo `src/main/resources/application.properties` contém as configurações essenciais:

```properties
spring.profiles.active=${APP_PROFILE:test}

spring.jpa.open-in-view=false

security.client-id=${CLIENT_ID:myclientid}
security.client-secret=${CLIENT_SECRET:myclientsecret}

security.jwt.duration=${JWT_DURATION:86400}

cors.origins=${CORS_ORIGINS:http://localhost:3000,http://localhost:5173}

# H2 Database (apenas para perfil 'test')
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.h2.console.settings.web-allow-others=true
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.sql.init.mode=always
```

**Variáveis de Ambiente:**

*   `APP_PROFILE`: Define o perfil ativo (padrão: `test`).
*   `CLIENT_ID`: ID do cliente OAuth2 (padrão: `myclientid`).
*   `CLIENT_SECRET`: Segredo do cliente OAuth2 (padrão: `myclientsecret`).
*   `JWT_DURATION`: Duração do JWT em segundos (padrão: `86400` - 24 horas).
*   `CORS_ORIGINS`: Origens permitidas para CORS (padrão: `http://localhost:3000,http://localhost:5173`).

### Executar a Aplicação

Você pode executar a aplicação usando Maven:

```bash
mvn spring-boot:run
```

Ou, se estiver usando uma IDE (IntelliJ IDEA, Eclipse, etc.), execute a classe principal `StartUpApplication.java`.

### Acessar o H2 Console

Com a aplicação em execução, o console do H2 estará disponível em:

[http://localhost:8080/h2-console](http://localhost:8080/h2-console)

Use as seguintes credenciais:

*   **JDBC URL:** `jdbc:h2:mem:testdb`
*   **User Name:** `sa`
*   **Password:** (deixe em branco)

O banco de dados será populado com usuários de teste (`alex` com `ROLE_OPERATOR` e `maria` com `ROLE_ADMIN`) através do `import.sql`.

## Endpoints da API

### 1. Autenticação (Obtenção de Token)

Para obter um token de acesso, faça uma requisição `POST` para o endpoint `/oauth2/token` com o `Custom Password Grant Type`.

**URL:** `POST http://localhost:8080/oauth2/token`

**Headers:**

*   `Content-Type: application/x-www-form-urlencoded`
*   `Authorization: Basic <base64_encoded_client_id:client_secret>`
    *   Exemplo: `Authorization: Basic bXljbGllbnRpZDpteWNsaWVudHNlY3JldA==` (para `myclientid:myclientsecret`)

**Body (x-www-form-urlencoded):**

*   `grant_type: password`
*   `username: <username>` (ex: `alex` ou `maria`)
*   `password: <password>` (ex: `123456`)

**Exemplo de Resposta (Sucesso):**

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiI...",
    "token_type": "Bearer",
    "expires_in": 86399,
    "scope": "read write"
}
```

### 2. Endpoints de Produto (`/products`)

Estes endpoints são protegidos e exigem um token de acesso válido no cabeçalho `Authorization: Bearer <access_token>`.

*   **`GET /products`**
    *   **Descrição:** Lista todos os produtos.
    *   **Acesso:** Público (não requer autenticação).
    *   **Exemplo de Resposta:**
        ```json
        [
            { "id": 1, "name": "TV", "price": 1200.00 },
            { "id": 2, "name": "Notebook", "price": 1800.00 }
        ]
        ```

*   **`GET /products/{id}`**
    *   **Descrição:** Retorna um produto específico pelo ID.
    *   **Acesso:** `ROLE_ADMIN` ou `ROLE_OPERATOR`.
    *   **Exemplo de Resposta:**
        ```json
        { "id": 1, "name": "TV", "price": 1200.00 }
        ```

*   **`POST /products`**
    *   **Descrição:** Cadastra um novo produto.
    *   **Acesso:** `ROLE_ADMIN`.
    *   **Body (JSON):**
        ```json
        {
            "name": "Novo Produto",
            "price": 500.00
        }
        ```
    *   **Exemplo de Resposta (201 Created):**
        ```json
        { "id": 3, "name": "Novo Produto", "price": 500.00 }
        ```

## Estrutura do Projeto

```
spring-security-auth-demo
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── marlonsilva
│   │   │           └── springSecurityAuthDemo
│   │   │               ├── StartUpApplication.java
│   │   │               ├── config
│   │   │               │   ├── AuthorizationServerConfig.java
│   │   │               │   └── ResourceServerConfig.java
│   │   │               │   └── customgrant
│   │   │               │       ├── CustomPasswordAuthenticationConverter.java
│   │   │               │       ├── CustomPasswordAuthenticationProvider.java
│   │   │               │       ├── CustomPasswordAuthenticationToken.java
│   │   │               │       └── CustomUserAuthorities.java
│   │   │               ├── controllers
│   │   │               │   └── ProductController.java
│   │   │               ├── dto
│   │   │               │   └── ProductDTO.java
│   │   │               ├── entities
│   │   │               │   ├── Product.java
│   │   │               │   ├── Role.java
│   │   │               │   └── User.java
│   │   │               ├── repositories
│   │   │               │   ├── ProductRepository.java
│   │   │               │   └── UserRepository.java
│   │   │               ├── services
│   │   │               │   ├── ProductService.java
│   │   │               │   └── UserService.java
│   │   │               └── projections
│   │   │                   └── UserDetailsProjection.java
│   │   └── resources
│   │       ├── application.properties
│   │       └── import.sql
│   └── test
│       └── java
│           └── com
│               └── marlonsilva
│                   └── springSecurityAuthDemo
│                       └── SpringSecurityAuthDemoApplicationTests.java
├── pom.xml
└── ... (outros arquivos de configuração)
```

