<h1> Crud via API Rest - Java, Spring Boot, Flyway, PostgreSQL </h1>
<h3> Descrição : </h3>
API REST que faz operações CRUD (Create, Read, Update, Delete) sobre usuários armazenados em um banco de dados PostgreSQL.
<h3> Demonstração em video : </h3>

https://github.com/user-attachments/assets/98e8417e-b5ed-432e-8917-8ff469948cbe

![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.0-brightgreen) ![Java](https://img.shields.io/badge/Java-24-blue) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14.0-blue)

## ✨ Principais Funcionalidades

* **Arquitetura em camadas bem definida**

  * Separação clara entre **Controller**, **Service** e **Repository**, promovendo alta coesão e baixo acoplamento. <br />
   ![image](https://github.com/user-attachments/assets/541c518e-7c93-4f50-98cc-3396534ada74)


* **Uso de DTOs como registros (Java `record`)**

  * `CreateUserDTO` e `UpdateUserDTO` deixam explícita a estrutura de entrada, facilitando manutenção, validação e documentação automática.<br />
    ![image](https://github.com/user-attachments/assets/61727a51-2c41-40af-9632-6181229a6ac3)


* **Injeção de dependências via construtor**

  * Tornando o código mais desacoplado e facilitando a **testabilidade** (sem uso de `@Autowired` diretamente nos campos).<br />

* **API RESTful seguindo convenções HTTP**

  * **POST** `/v1/users` → cria novo usuário (**201 Created** + cabeçalho `Location`).
  * **GET** `/v1/users/{id}` → pesquisa por ID (**200 OK** ou **404 Not Found**).
  * **GET** `/v1/users` → lista todos os usuários (**200 OK**).
  * **PUT** `/v1/users/{id}` → atualiza parcialmente um usuário (**204 No Content**).
  * **DELETE** `/v1/users/{id}` → remove usuário (**204 No Content**).

* **Tratamento de entradas e conversão de tipos**

  * Converte `String` → `Integer` de forma segura e usa `Optional` para evitar `NullPointerException`.

* **Uso de Spring Data JPA**

  * `UserRepository` estende `JpaRepository`, aproveitando métodos prontos como `findById`, `findAll`, `save`, `existsById` e `deleteById`.

* **Atualização condicional de campos**

  * No método de **update**, só altera um campo se o valor no DTO for não-nulo, permitindo comportamento de *patch*.

* **Resposta padronizada e amigável**

  * Uso de `ResponseEntity` para controlar status HTTP e corpo de resposta de forma explícita.

* **Script de criação de tabela SQL**

  ```sql
  CREATE TABLE user_tb (
     id SERIAL PRIMARY KEY,
     first_name VARCHAR(255),
     last_name  VARCHAR(255),
     email      VARCHAR(255),
     password   VARCHAR(255)
  );
  ```

---

## 🛠️ Tecnologias Utilizadas

* **Java 24**
* **Spring Boot 3.5.0**
* **Spring Web (Spring MVC)**
* **Spring Data JPA**
* **PostgreSQL 14**
* **Maven**

---

## 📦 Como Executar

1. Clone o repositório:

   ```bash
   git clone https://github.com/seu-usuario/seu-projeto.git
   ```
2. Configure o banco de dados em `application.properties`:

   ```properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/seu_db
   spring.datasource.username=usuario
   spring.datasource.password=senha
   ```
3. Execute a aplicação:

   ```bash
   mvn spring-boot:run
   ```

Pronto! A API estará disponível em `http://localhost:8080/v1/users`.

## ☸️ Executando no Kubernetes

Os manifests em [`k8s/`](./k8s/) criam um namespace, a API com duas réplicas e um PostgreSQL com volume persistente. A API fica disponível internamente no cluster; use `kubectl port-forward` para acessá-la localmente.

1. Inicie um cluster local. Exemplo para Windows Pro com Hyper-V:

   ```powershell
   minikube start --driver=hyperv --hyperv-virtual-switch="Default Switch" --disk-size=10g --cpus=2 --memory=4096
   kubectl get nodes
   ```

   Aguarde o nó ficar `Ready`. O tamanho do disco deve caber no espaço disponível no computador.

2. Gere o JAR e construa a imagem diretamente no Minikube (não requer Docker Desktop):

   ```powershell
   .\mvnw.cmd package
   minikube image build --tag api-rest-java:1.0 .
   ```

   Para Kind, construa a imagem com Docker e carregue-a no cluster:

   ```powershell
   docker build -t api-rest-java:1.0 .
   kind load docker-image api-rest-java:1.0
   ```

   Em outros clusters, publique a imagem em um registry acessível pelos nós e atualize `image` em `k8s/api.yaml`.

3. Crie o namespace e o Secret do banco. Substitua os valores de exemplo, principalmente a senha:

   ```powershell
   kubectl apply -f k8s/namespace.yaml
   kubectl create secret generic postgres-credentials -n api-rest `
     --from-literal=database=db123 `
     --from-literal=username=user123 `
     --from-literal=password=password123
   ```

   O Secret não é armazenado nos manifests. Para produção, use uma senha forte e considere um gerenciador de Secrets; os valores em Secrets do Kubernetes não são criptografados por padrão no etcd.

4. Aplique os manifests e acompanhe os Pods:

   ```powershell
   kubectl apply -f k8s/
   kubectl get pods -n api-rest
   ```

5. Quando os Pods estiverem prontos, encaminhe a porta e acesse a API:

   ```powershell
   kubectl port-forward -n api-rest service/public-api 8080:8080
   ```

   A API estará em `http://localhost:8080/v1/users`. O PVC do PostgreSQL requer uma `StorageClass` padrão no cluster. Se alterar o nome do banco no Secret, atualize também `SPRING_DATASOURCE_URL` em `k8s/api.yaml`.
