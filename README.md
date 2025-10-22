# 🏥 Healthcare Microservices Architecture

> **Author:** Prahlad-7  
> A comprehensive guide for setting up and configuring a distributed healthcare management system

---

## 📋 Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Services](#services)
  - [Patient Service](#patient-service)
  - [Billing Service](#billing-service)
  - [Notification Service](#notification-service)
  - [Auth Service](#auth-service)
- [Infrastructure Components](#infrastructure-components)
- [Communication Patterns](#communication-patterns)
- [Getting Started](#getting-started)

---

## 🎯 Overview

This repository contains a microservices-based healthcare management system built with Spring Boot, featuring:

- **gRPC** for inter-service communication
- **Apache Kafka** for event-driven architecture
- **PostgreSQL** for persistent data storage
- **JWT-based authentication** for secure access control
- **Protocol Buffers** for efficient data serialization

---

## 🏛️ System Architecture

### High-Level Architecture Diagram

```mermaid
graph TB
    subgraph "Client Layer"
        Client[Web/Mobile Client]
    end
    
    subgraph "API Gateway Layer"
        Gateway[API Gateway]
    end
    
    subgraph "Service Layer"
        Auth[Auth Service<br/>Port: 8080]
        Patient[Patient Service<br/>Port: 8081]
        Billing[Billing Service<br/>Port: 8082]
        Notification[Notification Service<br/>Port: 8083]
    end
    
    subgraph "Message Broker"
        Kafka[Apache Kafka<br/>Port: 9092/9094]
    end
    
    subgraph "Data Layer"
        AuthDB[(Auth DB<br/>PostgreSQL)]
        PatientDB[(Patient DB<br/>PostgreSQL)]
    end
    
    Client -->|HTTP/REST| Gateway
    Gateway -->|HTTP| Auth
    Gateway -->|HTTP| Patient
    Gateway -->|HTTP| Billing
    
    Patient -->|gRPC| Billing
    Patient -->|Publish Events| Kafka
    Kafka -->|Consume Events| Notification
    
    Auth -.->|Read/Write| AuthDB
    Patient -.->|Read/Write| PatientDB
    
    style Auth fill:#e1f5ff
    style Patient fill:#e1f5ff
    style Billing fill:#e1f5ff
    style Notification fill:#e1f5ff
    style Kafka fill:#fff4e1
    style AuthDB fill:#f0f0f0
    style PatientDB fill:#f0f0f0
```

### Service Dependency Map

```mermaid
graph LR
    subgraph "Dependencies"
        PS[Patient Service]
        BS[Billing Service]
        NS[Notification Service]
        AS[Auth Service]
        K[Kafka Broker]
        
        PS -->|gRPC Call| BS
        PS -->|Produces Events| K
        K -->|Consumes Events| NS
        AS -.->|JWT Validation| PS
        AS -.->|JWT Validation| BS
    end
    
    style PS fill:#4CAF50,color:#fff
    style BS fill:#2196F3,color:#fff
    style NS fill:#FF9800,color:#fff
    style AS fill:#9C27B0,color:#fff
    style K fill:#FFC107
```

---

## 🔄 Communication Patterns

### Request-Response Pattern (gRPC)

```mermaid
sequenceDiagram
    participant C as Client
    participant PS as Patient Service
    participant BS as Billing Service
    
    C->>PS: Create Patient Appointment
    activate PS
    PS->>PS: Validate Request
    PS->>BS: Get Billing Info (gRPC)
    activate BS
    BS-->>PS: Billing Details
    deactivate BS
    PS->>PS: Save Appointment
    PS-->>C: Appointment Created
    deactivate PS
```

### Event-Driven Pattern (Kafka)

```mermaid
sequenceDiagram
    participant PS as Patient Service
    participant K as Kafka
    participant NS as Notification Service
    participant U as User
    
    PS->>K: Publish PatientCreated Event
    activate K
    K->>NS: Consume PatientCreated Event
    activate NS
    NS->>NS: Process Event
    NS->>U: Send Welcome Email
    NS->>U: Send SMS Notification
    deactivate NS
    deactivate K
```

### Authentication Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant AS as Auth Service
    participant DB as Auth Database
    participant PS as Patient Service
    
    C->>AS: POST /login (credentials)
    activate AS
    AS->>DB: Validate User
    DB-->>AS: User Details
    AS->>AS: Generate JWT Token
    AS-->>C: JWT Token
    deactivate AS
    
    C->>PS: GET /patients (with JWT)
    activate PS
    PS->>PS: Validate JWT
    PS->>PS: Extract User Info
    PS-->>C: Patient Data
    deactivate PS
```

---

## 📊 Data Flow Diagram

```mermaid
graph TD
    subgraph "Patient Management Flow"
        A[Client Request] -->|1. Authentication| B[Auth Service]
        B -->|2. JWT Token| C[Patient Service]
        C -->|3. gRPC Request| D[Billing Service]
        D -->|4. Billing Info| C
        C -->|5. Publish Event| E[Kafka]
        E -->|6. Consume| F[Notification Service]
        F -->|7. Send Alert| G[Email/SMS]
        C -->|8. Store Data| H[(Patient DB)]
    end
    
    style A fill:#e3f2fd
    style B fill:#f3e5f5
    style C fill:#e8f5e9
    style D fill:#e1f5fe
    style E fill:#fff9c4
    style F fill:#ffe0b2
    style G fill:#ffebee
    style H fill:#f5f5f5
```

---

## 🚀 Services

### Patient Service

The Patient Service manages patient records and coordinates with other services through gRPC and Kafka.

#### Service Architecture

```mermaid
graph TB
    subgraph "Patient Service Internal Architecture"
        Controller[REST Controller]
        Service[Business Logic]
        Repository[JPA Repository]
        GrpcClient[gRPC Client]
        KafkaProducer[Kafka Producer]
        
        Controller -->|Calls| Service
        Service -->|CRUD| Repository
        Service -->|Remote Call| GrpcClient
        Service -->|Publish| KafkaProducer
        
        Repository -.->|Persist| DB[(PostgreSQL)]
        GrpcClient -.->|Connect| BillingService[Billing Service]
        KafkaProducer -.->|Send| Kafka[Kafka Broker]
    end
    
    style Controller fill:#4CAF50,color:#fff
    style Service fill:#8BC34A,color:#fff
    style Repository fill:#CDDC39
    style GrpcClient fill:#2196F3,color:#fff
    style KafkaProducer fill:#FFC107
```

#### 📦 Environment Variables

```bash
# Database Configuration
SPRING_DATASOURCE_URL=jdbc:postgresql://patient-service-db:5432/db
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_DATASOURCE_PASSWORD=password
SPRING_JPA_HIBERNATE_DDL_AUTO=update
SPRING_SQL_INIT_MODE=always

# gRPC Configuration
BILLING_SERVICE_ADDRESS=billing-service
BILLING_SERVICE_GRPC_PORT=9005

# Kafka Configuration
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092

# Debug Configuration
JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005
```

#### 🔧 Maven Dependencies

Add these dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- gRPC Core -->
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-netty-shaded</artifactId>
        <version>1.69.0</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-protobuf</artifactId>
        <version>1.69.0</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-stub</artifactId>
        <version>1.69.0</version>
    </dependency>
    
    <!-- Java 9+ Compatibility -->
    <dependency>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>annotations-api</artifactId>
        <version>6.0.53</version>
        <scope>provided</scope>
    </dependency>
    
    <!-- Spring Boot gRPC Starter -->
    <dependency>
        <groupId>net.devh</groupId>
        <artifactId>grpc-spring-boot-starter</artifactId>
        <version>3.1.0.RELEASE</version>
    </dependency>
    
    <!-- Protocol Buffers -->
    <dependency>
        <groupId>com.google.protobuf</groupId>
        <artifactId>protobuf-java</artifactId>
        <version>4.29.1</version>
    </dependency>
</dependencies>
```

#### 🏗️ Build Configuration

```xml
<build>
    <extensions>
        <!-- OS-specific protoc compatibility -->
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.7.0</version>
        </extension>
    </extensions>
    
    <plugins>
        <!-- Spring Boot Plugin -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>

        <!-- Protocol Buffers Compiler -->
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>com.google.protobuf:protoc:3.25.5:exe:${os.detected.classifier}</protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.68.1:exe:${os.detected.classifier}</pluginArtifact>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#### ⚙️ Kafka Producer Configuration

Add to `application.properties`:

```properties
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.ByteArrayDeserializer
```

---

### Billing Service

Handles billing operations and exposes gRPC endpoints for other services.

#### Service Architecture

```mermaid
graph TB
    subgraph "Billing Service Internal Architecture"
        GrpcServer[gRPC Server]
        BillingLogic[Billing Logic]
        Calculator[Price Calculator]
        
        GrpcServer -->|Receives Requests| BillingLogic
        BillingLogic -->|Calculate| Calculator
        Calculator -->|Returns| BillingLogic
        BillingLogic -->|Response| GrpcServer
    end
    
    style GrpcServer fill:#2196F3,color:#fff
    style BillingLogic fill:#03A9F4,color:#fff
    style Calculator fill:#00BCD4,color:#fff
```

#### 📦 Dependencies & Build

Uses the same gRPC dependencies and build configuration as the Patient Service (see above).

---

### Notification Service

Processes events from Kafka and sends notifications to users.

#### Service Architecture

```mermaid
graph TB
    subgraph "Notification Service Internal Architecture"
        KafkaListener[Kafka Consumer]
        EventProcessor[Event Processor]
        EmailService[Email Service]
        SMSService[SMS Service]
        
        KafkaListener -->|Receives Events| EventProcessor
        EventProcessor -->|Send Email| EmailService
        EventProcessor -->|Send SMS| SMSService
        
        EmailService -.->|SMTP| EmailProvider[Email Provider]
        SMSService -.->|API| SMSProvider[SMS Gateway]
    end
    
    style KafkaListener fill:#FF9800,color:#fff
    style EventProcessor fill:#FFB74D,color:#fff
    style EmailService fill:#FDD835
    style SMSService fill:#FDD835
```

#### 📦 Environment Variables

```bash
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
```

#### 🔧 Maven Dependencies

```xml
<dependencies>
    <!-- Kafka Support -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
        <version>3.3.0</version>
    </dependency>

    <!-- Protocol Buffers -->
    <dependency>
        <groupId>com.google.protobuf</groupId>
        <artifactId>protobuf-java</artifactId>
        <version>4.29.1</version>
    </dependency>
</dependencies>
```

#### 🏗️ Build Configuration

```xml
<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.7.0</version>
        </extension>
    </extensions>
    
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>

        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>com.google.protobuf:protoc:3.25.5:exe:${os.detected.classifier}</protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.68.1:exe:${os.detected.classifier}</pluginArtifact>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

---

### Auth Service

Provides JWT-based authentication and authorization for the system.

#### Service Architecture

```mermaid
graph TB
    subgraph "Auth Service Internal Architecture"
        AuthController[Auth Controller]
        AuthService[Auth Service]
        JWTUtil[JWT Utility]
        UserRepo[User Repository]
        SecurityConfig[Security Config]
        
        AuthController -->|Login/Register| AuthService
        AuthService -->|Validate| UserRepo
        AuthService -->|Generate Token| JWTUtil
        SecurityConfig -->|Configure| AuthController
        
        UserRepo -.->|Query| AuthDB[(Auth Database)]
    end
    
    style AuthController fill:#9C27B0,color:#fff
    style AuthService fill:#AB47BC,color:#fff
    style JWTUtil fill:#BA68C8,color:#fff
    style UserRepo fill:#CE93D8,color:#fff
    style SecurityConfig fill:#E1BEE7
```

#### 📦 Environment Variables

```bash
# Database Configuration
SPRING_DATASOURCE_URL=jdbc:postgresql://auth-service-db:5432/db
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_DATASOURCE_PASSWORD=password
SPRING_JPA_HIBERNATE_DDL_AUTO=update
SPRING_SQL_INIT_MODE=always
```

#### 🔧 Maven Dependencies

```xml
<dependencies>
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- JWT Support -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.12.6</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.12.6</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.12.6</version>
        <scope>runtime</scope>
    </dependency>

    <!-- Database Drivers -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>

    <!-- API Documentation -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.6.0</version>
    </dependency>

    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

#### 💾 Database Initialization

Create `src/main/resources/data.sql`:

```sql
-- Create users table
CREATE TABLE IF NOT EXISTS "users" (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL
);

-- Insert default admin user (password: test123)
INSERT INTO "users" (id, email, password, role)
SELECT 
    '223e4567-e89b-12d3-a456-426614174006',
    'testuser@test.com',
    '$2b$12$7hoRZfJrRKD2nIm2vHLs7OBETy.LWenXXMLKf99W8M4PUwO6KB7fu',
    'ADMIN'
WHERE NOT EXISTS (
    SELECT 1 FROM "users"
    WHERE id = '223e4567-e89b-12d3-a456-426614174006'
       OR email = 'testuser@test.com'
);
```

> **Default Credentials:**  
> Email: `testuser@test.com`  
> Password: `test123`

---

## 🏗️ Infrastructure Components

### Kafka Architecture

```mermaid
graph TB
    subgraph "Kafka Cluster"
        Broker[Kafka Broker<br/>Node: 0<br/>Roles: controller,broker]
        
        subgraph "Listeners"
            Internal[PLAINTEXT<br/>Port: 9092<br/>Internal Network]
            External[EXTERNAL<br/>Port: 9094<br/>localhost]
            Controller[CONTROLLER<br/>Port: 9093<br/>Management]
        end
        
        Broker --> Internal
        Broker --> External
        Broker --> Controller
    end
    
    subgraph "Producers"
        PS[Patient Service]
    end
    
    subgraph "Consumers"
        NS[Notification Service]
    end
    
    PS -->|Publish| Internal
    Internal -->|Consume| NS
    
    style Broker fill:#FFC107
    style Internal fill:#4CAF50,color:#fff
    style External fill:#2196F3,color:#fff
    style Controller fill:#9C27B0,color:#fff
```

#### 🐳 Docker Environment Variables

For IntelliJ IDEA container configuration:

```bash
KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092,EXTERNAL://localhost:9094
KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka:9093
KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,EXTERNAL:PLAINTEXT,PLAINTEXT:PLAINTEXT
KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
KAFKA_CFG_NODE_ID=0
KAFKA_CFG_PROCESS_ROLES=controller,broker
```

#### 📡 Port Mapping

| Listener | Port | Purpose | Access |
|----------|------|---------|--------|
| PLAINTEXT | 9092 | Internal service communication | Docker network |
| EXTERNAL | 9094 | External testing/debugging | localhost |
| CONTROLLER | 9093 | Cluster management | Internal only |

---

### Database Architecture

```mermaid
graph TB
    subgraph "Database Layer"
        subgraph "Auth Database"
            AuthDB[(PostgreSQL<br/>Port: 5432<br/>DB: db)]
            AuthSchema[Schema: users]
            AuthDB --> AuthSchema
        end
        
        subgraph "Patient Database"
            PatientDB[(PostgreSQL<br/>Port: 5432<br/>DB: db)]
            PatientSchema[Schema: patients,<br/>appointments]
            PatientDB --> PatientSchema
        end
    end
    
    AS[Auth Service] -.->|Read/Write| AuthDB
    PS[Patient Service] -.->|Read/Write| PatientDB
    
    style AuthDB fill:#f0f0f0
    style PatientDB fill:#f0f0f0
```

#### Auth Service Database

```bash
POSTGRES_DB=db
POSTGRES_USER=admin_user
POSTGRES_PASSWORD=password
```

**Container hostname:** `auth-service-db`  
**Port:** `5432`

---

## 🔄 Complete Request Flow Example

### Creating a Patient with Billing

```mermaid
sequenceDiagram
    autonumber
    participant Client
    participant Auth as Auth Service
    participant Patient as Patient Service
    participant Billing as Billing Service
    participant Kafka
    participant Notification as Notification Service
    participant DB as Patient DB
    
    Client->>Auth: POST /login
    Auth-->>Client: JWT Token
    
    Client->>Patient: POST /patients (JWT)
    activate Patient
    Patient->>Patient: Validate JWT
    
    Patient->>Billing: GetBillingPlan(patientType) [gRPC]
    activate Billing
    Billing-->>Patient: BillingPlan Details
    deactivate Billing
    
    Patient->>DB: Save Patient Record
    DB-->>Patient: Patient Saved
    
    Patient->>Kafka: Publish PatientCreated Event
    Patient-->>Client: 201 Created
    deactivate Patient
    
    Kafka->>Notification: PatientCreated Event
    activate Notification
    Notification->>Notification: Process Event
    Notification->>Client: Send Welcome Email
    Notification->>Client: Send SMS
    deactivate Notification
```

---

## 🚀 Getting Started

### Prerequisites

- ☕ Java 17+
- 📦 Maven 3.8+
- 🐳 Docker & Docker Compose
- 💻 IntelliJ IDEA (recommended)

### Deployment Architecture

```mermaid
graph TB
    subgraph "Development Environment"
        subgraph "Docker Containers"
            K[Kafka Container]
            ADB[(Auth DB Container)]
            PDB[(Patient DB Container)]
        end
        
        subgraph "Spring Boot Applications"
            AS[Auth Service<br/>Port: 8080]
            PS[Patient Service<br/>Port: 8081]
            BS[Billing Service<br/>Port: 8082]
            NS[Notification Service<br/>Port: 8083]
        end
        
        PS --> K
        PS --> BS
        K --> NS
        AS --> ADB
        PS --> PDB
    end
    
    style K fill:#FFC107
    style AS fill:#9C27B0,color:#fff
    style PS fill:#4CAF50,color:#fff
    style BS fill:#2196F3,color:#fff
    style NS fill:#FF9800,color:#fff
```

### Quick Start

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd <project-directory>
   ```

2. **Start infrastructure services**
   ```bash
   docker-compose up -d
   ```

3. **Build all services**
   ```bash
   mvn clean install
   ```

4. **Run services**
   - Start each service individually using your IDE
   - Or use Maven: `mvn spring-boot:run`

5. **Verify setup**
   - Auth Service: `http://localhost:8080`
   - Patient Service: `http://localhost:8081`
   - Billing Service: `http://localhost:8082` (gRPC)
   - Notification Service: `http://localhost:8083`
   - API Documentation: `http://localhost:8080/swagger-ui.html`

### Service Startup Order

```mermaid
graph LR
    Step1[1. Start Kafka] --> Step2[2. Start Databases]
    Step2 --> Step3[3. Start Auth Service]
    Step3 --> Step4[4. Start Billing Service]
    Step4 --> Step5[5. Start Patient Service]
    Step5 --> Step6[6. Start Notification Service]
    
    style Step1 fill:#FFC107
    style Step2 fill:#f0f0f0
    style Step3 fill:#9C27B0,color:#fff
    style Step4 fill:#2196F3,color:#fff
    style Step5 fill:#4CAF50,color:#fff
    style Step6 fill:#FF9800,color:#fff
```

---

## 📊 Monitoring & Observability

### Health Check Endpoints

| Service | Endpoint | Port |
|---------|----------|------|
| Auth Service | `/actuator/health` | 8080 |
| Patient Service | `/actuator/health` | 8081 |
| Billing Service | `/actuator/health` | 8082 |
| Notification Service | `/actuator/health` | 8083 |

---

## 📚 Additional Resources

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [gRPC Java Documentation](https://grpc.io/docs/languages/java/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Protocol Buffers Guide](https://protobuf.dev/)

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

**Built with ❤️ by Prahlad-7**
