"# LIS" 
# 🏥 LIS Project Initialization Guide

## 📁 Recommended Project Structure (Monorepo)

```
lis-platform/
├── backend/
│   ├── services/
│   │   ├── patient-service/           # Patient demographics & registration
│   │   ├── order-service/             # Lab order management
│   │   ├── specimen-service/          # Sample tracking & lifecycle
│   │   ├── result-service/            # Test results & validation
│   │   ├── billing-service/           # Billing & insurance
│   │   ├── integration-service/       # FHIR/HL7 EMR integration
│   │   ├── notification-service/      # Alerts & notifications (Kafka consumer)
│   │   └── ai-service/                # AI/ML workflows (validation, QC)
│   ├── libs/
│   │   ├── common-domain/             # Shared domain models
│   │   ├── common-security/           # JWT, OAuth2, RBAC
│   │   ├── fhir-mapper/               # FHIR R4 resource mappers
│   │   ├── kafka-events/              # Event schemas & producers/consumers
│   │   └── api-gateway/               # Spring Cloud Gateway
│   └── infrastructure/
│       ├── docker-compose.yml         # Local development stack
│       ├── kubernetes/                # K8s manifests
│       └── terraform/                 # Cloud infrastructure
├── frontend/
│   ├── apps/
│   │   ├── patient-portal/            # Patient-facing app
│   │   ├── clinician-dashboard/       # Lab technician & doctor UI
│   │   └── admin-console/             # System administration
│   └── packages/
│       ├── ui-components/             # Shared Shadcn/UI components
│       ├── api-client/                # Generated API clients
│       └── shared-types/              # TypeScript types
├── docs/
│   ├── api/                           # OpenAPI/Swagger specs
│   ├── architecture/                  # ADRs, diagrams
│   └── fhir-mappings/                 # FHIR resource documentation
├── scripts/
│   └── setup/                         # Initialization scripts
└── .github/
    └── workflows/                     # CI/CD pipelines
```

---

## 🚀 Step 1: Initialize Backend Services (Spring Boot)

### Prerequisites
```bash
# Install Java 17+, Maven/Gradle, Docker
java --version  # Should be 17 or higher
docker --version
```

### Create Parent POM (Maven Multi-Module)

**File: `backend/pom.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <groupId>com.healthcare</groupId>
    <artifactId>lis-platform</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>libs/common-domain</module>
        <module>libs/common-security</module>
        <module>libs/fhir-mapper</module>
        <module>libs/kafka-events</module>
        <module>services/patient-service</module>
        <module>services/order-service</module>
        <module>services/specimen-service</module>
        <module>services/result-service</module>
        <module>services/integration-service</module>
        <module>services/notification-service</module>
        <module>services/ai-service</module>
    </modules>

    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.0</spring-cloud.version>
        <hapi-fhir.version>6.10.0</hapi-fhir.version>
        <kafka.version>3.6.0</kafka.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- Spring Cloud -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <!-- HAPI FHIR -->
            <dependency>
                <groupId>ca.uhn.hapi.fhir</groupId>
                <artifactId>hapi-fhir-structures-r4</artifactId>
                <version>${hapi-fhir.version}</version>
            </dependency>

            <!-- Kafka -->
            <dependency>
                <groupId>org.springframework.kafka</groupId>
                <artifactId>spring-kafka</artifactId>
                <version>3.1.0</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### Create Sample Microservice: Patient Service

```bash
cd backend/services
mkdir -p patient-service/src/main/{java/com/healthcare/patient,resources}
```

**File: `patient-service/pom.xml`**

```xml
<project>
    <parent>
        <groupId>com.healthcare</groupId>
        <artifactId>lis-platform</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../../pom.xml</relativePath>
    </parent>

    <artifactId>patient-service</artifactId>

    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <!-- Database -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>

        <!-- HAPI FHIR -->
        <dependency>
            <groupId>ca.uhn.hapi.fhir</groupId>
            <artifactId>hapi-fhir-structures-r4</artifactId>
        </dependency>

        <!-- Internal Dependencies -->
        <dependency>
            <groupId>com.healthcare</groupId>
            <artifactId>common-domain</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>com.healthcare</groupId>
            <artifactId>kafka-events</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
```

**File: `patient-service/src/main/resources/application.yml`**

```yaml
server:
  port: 8081

spring:
  application:
    name: patient-service
  datasource:
    url: jdbc:postgresql://localhost:5432/lis_patients
    username: lis_user
    password: lis_password
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

### Initialize Shared Libraries

**File: `backend/libs/kafka-events/src/main/java/com/healthcare/events/PatientEvent.java`**

```java
package com.healthcare.events;

import lombok.Data;
import java.time.Instant;

@Data
public class PatientEvent {
    private String eventId;
    private String eventType; // PATIENT_CREATED, PATIENT_UPDATED
    private String patientId;
    private String mrn; // Medical Record Number
    private Instant timestamp;
    private PatientEventData data;
}

@Data
class PatientEventData {
    private String firstName;
    private String lastName;
    private String dateOfBirth;
    private String gender;
    private String email;
    private String phone;
}
```

---

## 🎨 Step 2: Initialize Frontend (React + TypeScript)

### Using Vite + pnpm Workspace

```bash
# Install pnpm (if not already)
npm install -g pnpm

# Initialize frontend workspace
mkdir frontend && cd frontend
pnpm init

# Create workspace configuration
cat > pnpm-workspace.yaml << EOF
packages:
  - 'apps/*'
  - 'packages/*'
EOF
```

### Create Main Dashboard App

```bash
cd apps
pnpm create vite clinici