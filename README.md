# Curso Microsserviços Java com Spring Boot e Spring Cloud

**Atenção:** Este curso foi atualizado para utilizar **Java 17** e **Spring Boot 3.3.2**

Instrutor: **Nelio Alves**

- [Udemy](https://www.udemy.com/user/nelio-alves)
- [YouTube](https://youtube.com/devsuperior)
- [Instagram](https://instagram.com/devsuperior.ig)

---

## Checklist para baixar e executar o projeto

1. **JDK 17**, variáveis `PATH` e `JAVA_HOME` configuradas.
2. Configurar a IDE para usar o Java 17.
3. Importar projetos na IDE.
4. Configurar credenciais do Config Server.
5. Modelo do curso: [GitHub - ms-course-configs](https://github.com/acenelio/ms-course-configs)
6. Preparar o Postman (collections e environments).
7. Subir os projetos na seguinte ordem:
   - Config Server
   - Eureka Server
   - Outros microsserviços

---

## Banco de Dados PostgreSQL na AWS

Este projeto utiliza um banco de dados **PostgreSQL** hospedado no **Amazon RDS**. Certifique-se de configurar sua instância RDS e ajustar as configurações de conexão conforme descrito abaixo.

### Configurando o Banco de Dados na AWS

1. **Criar uma instância PostgreSQL no Amazon RDS**:
   - Defina as credenciais (usuário e senha).
   - Escolha o nome do banco de dados, por exemplo, `hr_worker_db`.
   - Habilite o acesso público (somente para fins de desenvolvimento).
   - Configure o **Security Group** para permitir conexões do seu endereço IP.

2. **Anote as informações de conexão**:
   - **Endpoint**: fornecido pela AWS após a criação da instância.
   - **Porta**: geralmente `5432`.
   - **Usuário** e **Senha**: definidos na criação da instância.

3. **Configurar os microsserviços para usar o PostgreSQL**:
   - Atualize o arquivo `application.properties` ou `bootstrap.properties` de cada microsserviço que utiliza banco de dados para incluir as configurações de conexão.

   Exemplo de configuração no `application.properties`:

   ```properties
   # Configurações do banco de dados
   spring.datasource.url=jdbc:postgresql://SEU_ENDPOINT:5432/hr_worker_db
   spring.datasource.username=SEU_USUARIO
   spring.datasource.password=SUA_SENHA
   spring.datasource.driver-class-name=org.postgresql.Driver

   # Configurações do Hibernate
   spring.jpa.hibernate.ddl-auto=update
   spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
   ```

---

## Fase 1: Comunicação simples, Feign, Ribbon

### 1.1 Criar projeto `hr-worker`

### 1.2 Implementar projeto `hr-worker`

**Script SQL** para popular o banco de dados:

```sql
INSERT INTO tb_worker (name, daily_income) VALUES ('Bob', 200.0);
INSERT INTO tb_worker (name, daily_income) VALUES ('Maria', 300.0);
INSERT INTO tb_worker (name, daily_income) VALUES ('Alex', 250.0);
```

**application.properties**:

```properties
spring.application.name=hr-worker
server.port=8001

# Configurações do banco de dados PostgreSQL
spring.datasource.url=jdbc:postgresql://SEU_ENDPOINT:5432/hr_worker_db
spring.datasource.username=SEU_USUARIO
spring.datasource.password=SUA_SENHA
spring.datasource.driver-class-name=org.postgresql.Driver

# Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Mostrar SQL no console (opcional)
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### 1.3 Criar projeto `hr-payroll`

**application.properties**:

```properties
spring.application.name=hr-payroll
server.port=8101

# Configurações de conexão (se necessário)
```

### 1.4 Implementar projeto `hr-payroll` (mock)

### 1.5 RestTemplate

### 1.6 Feign

### 1.7 Ribbon load balancing

**Configuração de execução**:

Adicione o argumento para especificar uma porta diferente ao iniciar múltiplas instâncias:

```
-Dserver.port=8002
```

---

## Fase 2: Eureka, Hystrix, Zuul

### 2.1 Criar projeto `hr-eureka-server`

### 2.2 Configurar `hr-eureka-server`

**Porta padrão**: 8761

Acesse o dashboard no navegador: [http://localhost:8761](http://localhost:8761)

### 2.3 Configurar clientes Eureka

- Remova o Ribbon de `hr-payroll`:
  - Atualize as dependências Maven.
  - Adicione as anotações necessárias no programa principal.
  - Ajuste as configurações no `application.properties`.

**Atenção**: Aguarde alguns instantes após subir os microsserviços para que eles se registrem no Eureka.

### 2.4 Random port para `hr-worker`

No `application.properties`:

```properties
server.port=${PORT:0}

eureka.instance.instance-id=${spring.application.name}:${spring.application.instance_id:${random.value}}
```

**Atenção**: Exclua as configurações múltiplas de execução de `hr-worker`.

### 2.5 Tolerância a falhas com Hystrix

### 2.6 Timeout de Hystrix e Ribbon

Teste sem a anotação do Hystrix antes de configurar o timeout.

Configurações no `application.properties`:

```properties
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=60000
ribbon.ConnectTimeout=10000
ribbon.ReadTimeout=20000
```

### 2.7 Criar projeto `hr-zuul-server`

### 2.8 Configurar `hr-zuul-server`

**Porta padrão**: 8765

### 2.9 Random port para `hr-payroll`

### 2.10 Zuul timeout

Mesmo que o timeout do Hystrix e Ribbon esteja configurado em um microsserviço, se o Zuul não tiver seu timeout configurado, ele poderá apresentar problemas de timeout. Configure o timeout no Zuul para evitar isso.

---

## Fase 3: Configuração centralizada

### 3.1 Criar projeto `hr-config-server`

### 3.2 Configurar projeto `hr-config-server`

Quando um microsserviço é iniciado, antes de se registrar no Eureka, ele busca as configurações no repositório central.

**Arquivos de configuração**:

- `hr-worker.properties`

  ```properties
  test.config=My config value default profile
  ```

- `hr-worker-test.properties`

  ```properties
  test.config=My config value test profile
  ```

**Testes**:

- [http://localhost:8888/hr-worker/default](http://localhost:8888/hr-worker/default)
- [http://localhost:8888/hr-worker/test](http://localhost:8888/hr-worker/test)

### 3.3 `hr-worker` como cliente do servidor de configuração, profiles ativos

No arquivo `bootstrap.properties` configuramos somente o que for relacionado ao servidor de configuração e o profile do projeto.

**Atenção**: As configurações do `bootstrap.properties` têm prioridade sobre as do `application.properties`.

### 3.4 Actuator para atualizar configurações em runtime

Adicione a anotação `@RefreshScope` em todas as classes que acessam configurações para permitir a atualização em tempo de execução.

### 3.5 Repositório Git privativo

**Atenção**: Reinicie a IDE após adicionar as variáveis de ambiente.

---

## Fase 4: Autenticação e Autorização

### 4.1 Criar projeto `hr-user`

### 4.2 Configurar projeto `hr-user`

### 4.3 Entidades `User`, `Role` e associação N-N

### 4.4 Carga inicial do banco de dados

**Script SQL**:

```sql
INSERT INTO tb_user (name, email, password) VALUES ('Nina Brown', 'nina@gmail.com', '$2a$10$...');
INSERT INTO tb_user (name, email, password) VALUES ('Leia Red', 'leia@gmail.com', '$2a$10$...');

INSERT INTO tb_role (role_name) VALUES ('ROLE_OPERATOR');
INSERT INTO tb_role (role_name) VALUES ('ROLE_ADMIN');

INSERT INTO tb_user_role (user_id, role_id) VALUES (1, 1);
INSERT INTO tb_user_role (user_id, role_id) VALUES (2, 1);
INSERT INTO tb_user_role (user_id, role_id) VALUES (2, 2);
```

### 4.5 `UserRepository`, `UserResource`, configuração do Zuul

### 4.6 Criar projeto `hr-oauth`

### 4.7 Configurar projeto `hr-oauth`

### 4.8 `UserFeignClient`

### 4.9 Login e geração do Token JWT

- Sobrescreva os métodos `configure(AuthenticationManagerBuilder)` e `authenticationManager()`.
- **Autorização Básica**: `"Basic " + Base64.encode(client-id + ":" + client-secret)`

### 4.10 Autorização de recursos pelo gateway Zuul

### 4.11 Configurando o Postman

**Variáveis de ambiente**:

- `api-gateway`: `http://localhost:8765`
- `config-host`: `http://localhost:8888`
- `client-name`: `CLIENT-NAME`
- `client-secret`: `CLIENT-SECRET`
- `username`: `leia@gmail.com`
- `password`: `123456`
- `token`: (será atribuído pelo script)

**Script para atribuir o token à variável de ambiente do Postman**:

```javascript
if (responseCode.code >= 200 && responseCode.code < 300) {
    var json = JSON.parse(responseBody);
    postman.setEnvironmentVariable('token', json.access_token);
}
```

### 4.12 Configuração de segurança para o servidor de configuração

### 4.13 Configurando CORS

**Teste no navegador**:

```javascript
fetch("http://localhost:8765/hr-worker/workers", {
  "headers": {
    "accept": "*/*",
    "accept-language": "en-US,en;q=0.9,pt-BR;q=0.8,pt;q=0.7",
    "sec-fetch-dest": "empty",
    "sec-fetch-mode": "cors",
    "sec-fetch-site": "cross-site"
  },
  "referrer": "http://localhost:3000",
  "referrerPolicy": "no-referrer-when-downgrade",
  "body": null,
  "method": "GET",
  "mode": "cors",
  "credentials": "omit"
});
```

---

## Observações Finais

- Certifique-se de ajustar todas as configurações de acordo com as versões atualizadas do Java e Spring Boot.
- Atente-se para as configurações do banco de dados PostgreSQL hospedado na AWS.
- Utilize variáveis de ambiente ou arquivos de configuração seguros para armazenar informações sensíveis, como credenciais do banco de dados.

---
