# 🛠️ Configuração do SonarQube + SonarScanner para Projetos Locais

Este repositório fornece um ambiente Docker para rodar **SonarQube** e um **SonarScanner**, permitindo a análise de qualidade de código em qualquer projeto local. 
**As configurações presentes são para um projeto JAVA**

## 📌 **Visão Geral**
O setup consiste em **dois `docker-compose` separados**:

1️⃣ **`docker-compose-sonar.yml`** → Sobe o SonarQube e um banco PostgreSQL.  
2️⃣ **`docker-compose-scanner.yml`** → Compila e escaneia um projeto específico, que deve conter:  
   - `docker-compose-scanner.yml`  
   - `Dockerfile-scanner`  
   - `sonar-project.properties`  

O SonarQube ficará rodando continuamente, e o scanner pode ser usado para testar qualquer projeto.

## 🚀 **Passo 1: Subindo o SonarQube**
Antes de executar qualquer análise, precisamos subir o servidor do **SonarQube**:

```sh
docker-compose -f docker-compose-sonar.yml up -d
```

Isso iniciará o SonarQube e um banco de dados PostgreSQL. Para verificar se está rodando, acesse:

🔗 **[http://localhost:9000](http://localhost:9000)**

Login padrão:  
- **Usuário:** `admin`  
- **Senha:** `admin`  

Se for a primeira vez acessando, o SonarQube solicitará a alteração da senha.

## 📂 **Passo 2: Configurando um Projeto para Análise**
Para testar um projeto localmente, adicione os seguintes arquivos na raiz do projeto:

### **1️⃣ `docker-compose-scanner.yml`** (Dentro do projeto a ser analisado)
```yaml
services:
  scanner:
    build:
      context: .
      dockerfile: Dockerfile-scanner
    container_name: sonar_scanner
```

### **2️⃣ `Dockerfile-scanner`** (Dentro do projeto a ser analisado)
```dockerfile
FROM maven:3.9.6-eclipse-temurin-21 AS builder

WORKDIR /app
COPY . /app

RUN mvn clean install -DskipTests

FROM sonarsource/sonar-scanner-cli:latest

WORKDIR /app
COPY --from=builder /app /app

CMD ["sonar-scanner", "-Dsonar.projectBaseDir=/app"]
```

### **3️⃣ `sonar-project.properties`** (Dentro do projeto a ser analisado)
```ini
sonar.projectKey=nome-do-projeto
sonar.projectName=Nome do Projeto
sonar.projectVersion=1.0
sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.java.binaries=target/classes
sonar.host.url=http://localhost:9000
sonar.login=SEU_TOKEN_DO_SONARQUBE
sonar.sourceEncoding=UTF-8
sonar.exclusions=**/node_modules/**, **/target/**, **/*.xml
sonar.test.exclusions=src/main/java/**/*
sonar.scm.provider=git
sonar.scm.disabled=true
# sonar.verbose=true
```

🔹 **Importante**: Gere um **Token de Autenticação** no SonarQube e substitua `SEU_TOKEN_DO_SONARQUBE` pelo token gerado.

## 🎯 **Passo 3: Rodando a Análise no Projeto**
Agora que o SonarQube está rodando, vá até o diretório do projeto que contém os arquivos de scanner e execute:

```sh
docker-compose -f docker-compose-scanner.yml up --build
```

Após a execução, acesse **[http://localhost:9000](http://localhost:9000)** para visualizar o relatório de qualidade do código.

## ✅ **Conclusão**
Agora você tem um ambiente funcional para rodar o **SonarQube localmente** e escanear qualquer projeto sem precisar instalar nada na máquina!

🚀 **Bons scans e código limpo!** 😃
