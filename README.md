# E-commerce Microservices Platform

## Project Overview
A robust e-commerce platform built with Spring Boot microservices architecture, implementing advanced features like API Gateway, Service Discovery, Circuit Breaker pattern, and containerized deployment on AWS.

## Architecture Components

### Core Services
1. **API Gateway Service**
   - Routes requests to appropriate microservices
   - Implements Spring Cloud Gateway
   - Port: 9090

2. **Config Server**
   - Centralized configuration management
   - Git-based configuration storage
   - Port: 9296

3. **Service Registry (Eureka)**
   - Service discovery and registration
   - Dynamic service lookup
   - Port: 8761

4. **Order Service**
   - Order processing and management
   - Order status tracking
   - Port: 8082
   
5. **Payment Service**
   - Payment processing
   - Transaction management
   - Port: 8083

6. **Product Service**
   - Product catalog management
   - Inventory tracking
   - Port: 8080

### New Features to be Implemented
1. **Resilience4j Integration**
   ```java
   @CircuitBreaker(name = "productService", fallbackMethod = "fallbackMethod")
   @RateLimiter(name = "productService")
   @Retry(name = "productService")
   ```

2. **Spring Cloud LoadBalancer**
   ```yaml
   spring:
     cloud:
       loadbalancer:
         ribbon:
           enabled: false
         configurations:
           default:
             eager-load:
               enabled: true
   ```

3. **Prometheus Monitoring**
   ```yaml
   management:
     endpoints:
       web:
         exposure:
           include: prometheus
     metrics:
       export:
         prometheus:
           enabled: true
   ```

## Technical Stack
- Java 17
- Spring Boot 3.x
- Spring Cloud
- MySQL
- MongoDB
- Docker
- Kubernetes
- AWS
- Jenkins
- Prometheus & Grafana

## Project Structure
```
ecommerce-microservices/
├── API-Gateway/
├── Config-Server/
├── Order-Service/
├── Payment-Service/
├── Product-Service/
├── Service-Registry/
├── kubernetes/
│   ├── deployments/
│   ├── services/
│   └── configmaps/
└── jenkins/
    └── Jenkinsfile
```

## Setup Instructions

### Prerequisites
- JDK 17
- Maven 3.8+
- Docker Desktop
- AWS CLI
- kubectl
- Git

### Local Development Setup

1. **Clone the Repository**
   ```bash
   git clone https://github.com/hemanthsaich/Ecommerce-Microservices.git
   cd Ecommerce-Microservices
   ```

2. **Build Services**
   ```bash
   mvn clean package -DskipTests
   ```

3. **Start Services in Order**
   ```bash
   # Start Service Registry first
   cd Service-Registry
   mvn spring-boot:run

   # Start Config Server
   cd ../Config-Server
   mvn spring-boot:run

   # Start other services
   cd ../Product-Service
   mvn spring-boot:run
   ```

### Docker Deployment

1. **Build Docker Images**
   ```bash
   docker build -t ecommerce/service-registry ./Service-Registry
   docker build -t ecommerce/config-server ./Config-Server
   docker build -t ecommerce/api-gateway ./API-Gateway
   docker build -t ecommerce/product-service ./Product-Service
   docker build -t ecommerce/order-service ./Order-Service
   docker build -t ecommerce/payment-service ./Payment-Service
   ```

2. **Run Containers**
   ```bash
   docker-compose up -d
   ```

### AWS Deployment

1. **Create EKS Cluster**
   ```bash
   eksctl create cluster \
     --name ecommerce-cluster \
     --region us-east-1 \
     --nodegroup-name standard-workers \
     --node-type t3.medium \
     --nodes 3
   ```

2. **Apply Kubernetes Configurations**
   ```bash
   kubectl apply -f kubernetes/configmaps/
   kubectl apply -f kubernetes/deployments/
   kubectl apply -f kubernetes/services/
   ```

## Monitoring Setup

### Prometheus Configuration

1. **Add Dependencies**
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   <dependency>
       <groupId>io.micrometer</groupId>
       <artifactId>micrometer-registry-prometheus</artifactId>
   </dependency>
   ```

2. **Configure Prometheus**
   ```yaml
   scrape_configs:
     - job_name: 'ecommerce-microservices'
       metrics_path: '/actuator/prometheus'
       static_configs:
         - targets: ['localhost:8080', 'localhost:8082', 'localhost:8083']
   ```

## CI/CD Pipeline

### Jenkins Pipeline Configuration
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'your-registry'
        KUBE_CONFIG = credentials('eks-config')
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/ecommerce-services")
                    docker.push()
                }
            }
        }
        
        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f kubernetes/'
            }
        }
    }
}
```

## API Documentation

### Service Endpoints

#### Product Service
- GET `/product/all` - Get all products
- POST `/product/create` - Create new product
- GET `/product/{id}` - Get product by ID

#### Order Service
- POST `/order/create` - Create new order
- GET `/order/{id}` - Get order details
- PUT `/order/update` - Update order status

#### Payment Service
- POST `/payment/process` - Process payment
- GET `/payment/{orderId}` - Get payment status

### Request/Response Examples

#### Create Product
```json
POST /product/create
{
    "name": "Sample Product",
    "price": 99.99,
    "quantity": 100
}
```

## Error Handling

### Circuit Breaker Configuration
```yaml
resilience4j.circuitbreaker:
  instances:
    productService:
      slidingWindowSize: 10
      failureRateThreshold: 50
      waitDurationInOpenState: 10000
      permittedNumberOfCallsInHalfOpenState: 3
```

## Security

- JWT Authentication
- Service-to-service communication security
- Role-based access control
- AWS security groups configuration

## Contributing
1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License
This project is licensed under the MIT License.
