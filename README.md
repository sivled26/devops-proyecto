# 🚀 Proyecto Semestral — Innovatech Chile
### ISY1101 Introducción a Herramientas DevOps | Evaluación Parcial N°3

---

## 📋 Descripción

Este proyecto corresponde a la EP3 de la asignatura ISY1101, donde se implementa una arquitectura de **orquestación y automatización en la nube** utilizando AWS EKS (Elastic Kubernetes Service).

La aplicación está compuesta por 3 servicios:

| Servicio | Tecnología | Puerto | Repositorio ECR |
|---|---|---|---|
| Frontend | Nginx | 80 | `innovatech-frontend` |
| Backend Ventas | Spring Boot | 8080 | `innovatech-backend` |
| Backend Despachos | Spring Boot | 8081 | `innovatech-backend2` |

---

## 🏗️ Arquitectura

```
GitHub (push a main)
        │
        ▼
GitHub Actions (CI/CD)
        │
   ┌────┴────┐
   │  Build  │  Docker build de cada servicio
   │  Push   │  Push a Amazon ECR
   │  Deploy │  kubectl set image → EKS
   └────┬────┘
        │
        ▼
Amazon EKS Cluster (devops-eks)
        │
   ┌────┴──────────────────┐
   │  Namespace: default   │
   │                       │
   │  [front-despacho]     │  2 réplicas → LoadBalancer (público)
   │  [back-ventas]        │  2 réplicas → ClusterIP (interno)
   │  [back-despachos]     │  2 réplicas → ClusterIP (interno)
   └───────────────────────┘
        │
        ▼
Amazon ECR (imágenes Docker)
Amazon CloudWatch (logs y métricas)
```

---

## 📁 Estructura del Repositorio

```
proyecto-semestral/
├── .github/
│   └── workflows/
│       └── deploy.yml          # Pipeline CI/CD GitHub Actions
├── back-Ventas_SpringBoot/
│   └── Springboot-API-REST/
│       └── Dockerfile
├── back-Despachos_SpringBoot/
│   └── Springboot-API-REST-DESPACHO/
│       └── Dockerfile
├── front_despacho/
│   └── Dockerfile
├── k8s/
│   ├── back-ventas.yaml        # Deployment + Service back-ventas
│   ├── back-despachos.yaml     # Deployment + Service back-despachos
│   ├── front-despacho.yaml     # Deployment + Service frontend
│   └── hpa.yaml                # Horizontal Pod Autoscaler (x3)
├── docker-compose.yml          # Para desarrollo local
└── README.md
```

---

## ✅ Requisitos Previos

Tener instalado en tu PC:

- [AWS CLI](https://aws.amazon.com/cli/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Git](https://git-scm.com/)
- Cuenta AWS Academy con laboratorio activo

---

## 🖥️ Ejecución Local (con Docker Compose)

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/proyecto-semestral.git
cd proyecto-semestral

# 2. Levantar todos los servicios
docker-compose up --build

# 3. Acceder a los servicios
# Frontend:         http://localhost:80
# Backend Ventas:   http://localhost:8080
# Backend Despachos: http://localhost:8081
```

---

## ☁️ Despliegue en AWS EKS (paso a paso)

### 1. Configurar credenciales AWS

```bash
aws configure
# AWS Access Key ID:     [de AWS Academy]
# AWS Secret Access Key: [de AWS Academy]
# Default region:        us-east-1

aws configure set aws_session_token [SESSION_TOKEN de AWS Academy]

# Verificar
aws sts get-caller-identity
```

### 2. Conectar kubectl al clúster EKS

```bash
aws eks update-kubeconfig --region us-east-1 --name devops-eks

# Verificar nodos
kubectl get nodes
```

### 3. Crear repositorios en Amazon ECR

```bash
aws ecr create-repository --repository-name innovatech-frontend --region us-east-1
aws ecr create-repository --repository-name innovatech-backend --region us-east-1
aws ecr create-repository --repository-name innovatech-backend2 --region us-east-1
```

### 4. Autenticarse en ECR y subir imágenes

```bash
# Login a ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 075692576990.dkr.ecr.us-east-1.amazonaws.com

# Build y Push Frontend
docker build -t innovatech-frontend ./front_despacho
docker tag innovatech-frontend:latest 075692576990.dkr.ecr.us-east-1.amazonaws.com/innovatech-frontend:latest
docker push 075692576990.dkr.ecr.us-east-1.amazonaws.com/innovatech-frontend:latest

# Build y Push Backend Ventas
docker build -t innovatech-backend ./back-Ventas_SpringBoot/Springboot-API-REST
docker tag innovatech-backend:latest 075692576990.dkr.ecr.us-east-1.amazonaws.com/innovatech-backend:latest
docker push 075692576990.dkr.ecr.us-east-1.amazonaws.com/innovatech-backend:latest

# Build y Push Backend Despachos
docker build -t innovatech-backend2 ./back-Despachos_SpringBoot/Springboot-API-REST-DESPACHO
docker tag innovatech-backend2:latest 075692576990.dkr.ecr.us-east-1.amazonaws.com/innovatech-backend2:latest
docker push 075692576990.dkr.ecr.us-east-1.amazonaws.com/innovatech-backend2:latest
```

### 5. Crear Secrets en Kubernetes

```bash
kubectl create secret generic back-ventas-secret --from-literal=SPRING_PROFILES_ACTIVE=prod
kubectl create secret generic back-despachos-secret --from-literal=SPRING_PROFILES_ACTIVE=prod

# Verificar
kubectl get secrets
```

### 6. Desplegar los servicios en EKS

```bash
kubectl apply -f k8s/back-ventas.yaml
kubectl apply -f k8s/back-despachos.yaml
kubectl apply -f k8s/front-despacho.yaml

# Verificar pods
kubectl get pods

# Verificar servicios y URL pública
kubectl get services
```

### 7. Configurar Autoscaling (HPA)

```bash
# Instalar Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Aplicar HPA
kubectl apply -f k8s/hpa.yaml

# Verificar
kubectl get hpa
```

---

## 🔄 Pipeline CI/CD (GitHub Actions)

El pipeline se activa automáticamente con cada `push` a la rama `main` y ejecuta:

```
1. Checkout del código
2. Configurar credenciales AWS
3. Login a Amazon ECR
4. Build y Push de las 3 imágenes Docker
5. Configurar kubectl → conectar a EKS
6. Deploy → kubectl set image en los 3 deployments
7. Rollout status → verificar que el deploy fue exitoso
8. Verificar pods
```

### Secrets requeridos en GitHub Actions

Ir a **Settings → Secrets and variables → Actions** y agregar:

| Secret | Descripción |
|---|---|
| `AWS_ACCESS_KEY_ID` | Credencial AWS Academy |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS Academy |
| `AWS_SESSION_TOKEN` | Token de sesión AWS Academy |
| `AWS_REGION` | `us-east-1` |
| `EKS_CLUSTER` | `devops-eks` |

---

## 📊 Autoscaling — Justificación

Se configuró **Horizontal Pod Autoscaler (HPA)** en los 3 servicios con umbral de **50% CPU**:

| Servicio | Min Pods | Max Pods | Umbral CPU |
|---|---|---|---|
| back-ventas | 2 | 6 | 50% |
| back-despachos | 2 | 6 | 50% |
| front-despacho | 2 | 4 | 50% |

El umbral del 50% permite que Kubernetes reaccione antes de que el servicio se sature, dando tiempo suficiente para que nuevos pods arranquen antes de que los usuarios noten degradación en el rendimiento.

---

## 📝 Ver Logs

```bash
# Logs del frontend
kubectl logs -l app=front-despacho --tail=50

# Logs del backend ventas
kubectl logs -l app=back-ventas --tail=50

# Logs del backend despachos
kubectl logs -l app=back-despachos --tail=50

# Logs en tiempo real
kubectl logs -f deployment/back-ventas
```

---

## 🔍 Comandos útiles de verificación

```bash
# Ver todos los pods
kubectl get pods

# Ver servicios y URLs
kubectl get services

# Ver HPA en acción
kubectl get hpa

# Ver nodos del clúster
kubectl get nodes

# Describir un pod con error
kubectl describe pod NOMBRE-DEL-POD

# Ver logs de un pod específico
kubectl logs NOMBRE-DEL-POD --tail=30
```

---

## 👥 Autores

- DELVIS SANTIAGO 
- DIEGO ROJAS

Duoc UC —Introducción a Herramientas DevOps — 2025
