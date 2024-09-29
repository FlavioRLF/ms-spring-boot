# Projeto HR Worker

Este é um projeto Spring Boot que utiliza um banco de dados PostgreSQL hospedado no Amazon RDS. O objetivo é demonstrar como configurar um projeto Java Spring para se conectar a um banco de dados PostgreSQL na AWS, incluindo todas as configurações necessárias para executar o projeto corretamente.

## Índice

- [Pré-requisitos](#pré-requisitos)
- [Configuração do Projeto](#configuração-do-projeto)
    - [Clonar o Repositório](#clonar-o-repositório)
    - [Configurar o Banco de Dados PostgreSQL no AWS RDS](#configurar-o-banco-de-dados-postgresql-no-aws-rds)
    - [Configurar as Dependências do Projeto](#configurar-as-dependências-do-projeto)
    - [Configurar o `application.properties`](#configurar-o-applicationproperties)
    - [Estrutura de Pacotes](#estrutura-de-pacotes)
    - [Popular o Banco de Dados](#popular-o-banco-de-dados)
- [Executar o Projeto](#executar-o-projeto)
- [Testar os Endpoints](#testar-os-endpoints)
- [Conectar ao Banco de Dados via IntelliJ IDEA](#conectar-ao-banco-de-dados-via-intellij-idea)
- [Troubleshooting (Resolução de Problemas)](#troubleshooting-resolução-de-problemas)
- [Referências](#referências)

---

## Pré-requisitos

Antes de começar, certifique-se de ter instalado em sua máquina:

- **Java JDK 17** ou superior.
- **Maven** para gerenciamento de dependências.
- **IntelliJ IDEA** ou outra IDE de sua preferência.
- **Postman** ou outra ferramenta para testar APIs REST.
- **Conta na AWS** com acesso para criar uma instância RDS PostgreSQL.

---

## Configuração do Projeto

### Clonar o Repositório

Clone o repositório do projeto para sua máquina local:

```bash
git clone https://github.com/seu-usuario/ms-spring-boot.git
```

### Configurar o Banco de Dados PostgreSQL no AWS RDS

#### 1. Criar uma Instância PostgreSQL no RDS

- **Acesse o Console da AWS** e navegue até o serviço **RDS**.
- **Crie uma nova instância** de banco de dados PostgreSQL:
    - Escolha o tipo de instância (versão gratuita, se aplicável).
    - Defina as credenciais do administrador (nome de usuário e senha).
    - Defina o **Database Name** (Nome do Banco de Dados), por exemplo, `hr_worker_db`.
    - Habilite a opção **Public accessibility** para permitir conexões externas (apenas para desenvolvimento).
    - Configure o **Security Group** para permitir conexões da sua máquina:
        - Adicione uma regra de entrada (Inbound Rule) permitindo o tráfego na porta `5432` do seu endereço IP.

#### 2. Anote as Informações de Conexão

- **Endpoint**: O endereço do banco de dados (exemplo: `hr-worker-db.xxxxxxxxxxxx.us-east-1.rds.amazonaws.com`).
- **Porta**: Geralmente `5432`.
- **Nome do Banco de Dados**: O nome que você definiu (exemplo: `hr_worker_db`).
- **Usuário** e **Senha**: As credenciais do administrador que você configurou.

### Configurar as Dependências do Projeto

No arquivo `pom.xml`, certifique-se de incluir a dependência do driver JDBC do PostgreSQL:

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.6.0</version>
</dependency>
```

### Configurar o `application.properties`

No arquivo `src/main/resources/application.properties`, configure as propriedades de conexão com o banco de dados:

```properties
# Configurações do banco de dados
spring.datasource.url=jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.datasource.driver-class-name=org.postgresql.Driver

# Configurações do Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Mostrar SQL no console (opcional)
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

**Nota:** Estamos usando variáveis de ambiente (`${...}`) para armazenar informações sensíveis. Defina essas variáveis em seu ambiente de desenvolvimento ou na configuração de execução da sua IDE.

#### Definir Variáveis de Ambiente no IntelliJ IDEA

1. **Abra as Configurações de Execução**:
    - Vá em **Run** -> **Edit Configurations**.

2. **Selecione sua Configuração de Execução**:
    - Escolha a configuração usada para executar sua aplicação.

3. **Defina as Variáveis de Ambiente**:
    - Encontre o campo **Environment variables**.
    - Adicione as variáveis necessárias:
      ```
      DB_HOST=seu-endpoint-rds.amazonaws.com;DB_PORT=5432;DB_NAME=hr_worker_db;DB_USERNAME=seu_usuario;DB_PASSWORD=sua_senha
      ```

### Estrutura de Pacotes

Certifique-se de que a estrutura de pacotes está organizada corretamente para o Spring Boot escanear todos os componentes:

```
com.dev.hrworker
├── HrWorkerApplication.java
├── entities
│   └── Worker.java
├── repository
│   └── WorkerRepository.java
└── resources
    └── WorkerResource.java
```

### Popular o Banco de Dados

Crie um arquivo `data.sql` em `src/main/resources` para popular o banco de dados ao iniciar a aplicação:

```sql
INSERT INTO tb_worker (name, daily_income) VALUES ('Bob', 200.0);
INSERT INTO tb_worker (name, daily_income) VALUES ('Maria', 300.0);
INSERT INTO tb_worker (name, daily_income) VALUES ('Alex', 250.0);
```

---

## Executar o Projeto

1. **Compile o Projeto**:

   ```bash
   mvn clean install
   ```

2. **Execute a Aplicação**:

    - Via linha de comando:

      ```bash
      mvn spring-boot:run
      ```

    - Ou execute diretamente pela sua IDE (IntelliJ IDEA).

---

## Testar os Endpoints

Use o **Postman** ou outra ferramenta similar para testar os endpoints da API.

- **Listar Todos os Trabalhadores**:

    - **Método**: GET
    - **URL**: `http://localhost:8080/workers`

- **Exemplo de Resposta**:

  ```json
  [
    {
      "id": 1,
      "name": "Bob",
      "dailyIncome": 200.0
    },
    {
      "id": 2,
      "name": "Maria",
      "dailyIncome": 300.0
    },
    {
      "id": 3,
      "name": "Alex",
      "dailyIncome": 250.0
    }
  ]
  ```

---

## Conectar ao Banco de Dados via IntelliJ IDEA

Para gerenciar o banco de dados diretamente pelo IntelliJ IDEA:

1. **Abrir a Janela de Banco de Dados**:

    - Vá em **View** -> **Tool Windows** -> **Database**.

2. **Adicionar uma Nova Fonte de Dados**:

    - Clique em **+** -> **Data Source** -> **PostgreSQL**.

3. **Configurar a Conexão**:

    - **Host**: `seu-endpoint-rds.amazonaws.com`
    - **Port**: `5432`
    - **Database**: `hr_worker_db`
    - **User**: `seu_usuario`
    - **Password**: `sua_senha`

4. **Testar a Conexão**:

    - Clique em **Test Connection** para verificar se está tudo correto.

5. **Salvar e Conectar**:

    - Após o teste bem-sucedido, clique em **OK** para salvar a configuração.

Agora você pode explorar as tabelas, executar consultas SQL e gerenciar o banco de dados diretamente pelo IntelliJ IDEA.

---

## Troubleshooting (Resolução de Problemas)

- **Erro ao Conectar ao Banco de Dados**:

    - Verifique se as variáveis de ambiente estão definidas corretamente.
    - Certifique-se de que o Security Group do RDS permite conexões do seu IP.
    - Verifique se o driver do PostgreSQL está incluído nas dependências.

- **Erro 404 Not Found ao Acessar o Endpoint**:

    - Certifique-se de que o controlador está no pacote correto e está sendo escaneado pelo Spring Boot.
    - Verifique a estrutura de pacotes e ajuste conforme necessário.

- **Tabelas Não Criadas Automaticamente**:

    - Verifique se `spring.jpa.hibernate.ddl-auto` está definido como `update` ou `create`.
    - Certifique-se de que as entidades estão corretamente anotadas com `@Entity` e mapeadas para as tabelas.

- **Problemas ao Executar o `data.sql`**:

    - Verifique se o arquivo está em `src/main/resources`.
    - Certifique-se de que os nomes das colunas correspondem exatamente aos do banco de dados.
    - Verifique se o banco de dados está sendo inicializado antes da execução do script.

---

## Referências

- [Documentação do Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [Driver JDBC do PostgreSQL](https://jdbc.postgresql.org/)
- [AWS RDS para PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.html)
- [Conectar-se a um Banco de Dados com IntelliJ IDEA](https://www.jetbrains.com/help/idea/connecting-to-a-database.html)
- [Spring Data JPA - Referência](https://spring.io/projects/spring-data-jpa)

---

**Nota:** Este projeto é destinado a fins educacionais e de desenvolvimento. Para ambientes de produção, recomenda-se seguir as melhores práticas de segurança, incluindo o gerenciamento adequado de credenciais e a configuração de ambientes seguros.