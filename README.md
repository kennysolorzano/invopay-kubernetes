# 🚀 Despliegue de InvoPay en Kubernetes

[![Licencia: MIT](https://img.shields.io/badge/Licencia-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker Hub Frontend](https://img.shields.io/badge/Docker%20Hub-invopay--frontend-blue?logo=docker)](https://hub.docker.com/r/kennysolo/invopay-frontend)
[![Docker Hub Backend](https://img.shields.io/badge/Docker%20Hub-invopay--backend-blue?logo=docker)](https://hub.docker.com/r/kennysolo/invopay-backend)

Repositorio oficial con toda la configuración de Kubernetes necesaria para desplegar la aplicación **InvoPay**, una solución compuesta por un frontend en Angular y un backend en Spring Boot, sobre un entorno de producción en AWS EKS con una base de datos RDS.

---

## 📑 Tabla de Contenidos
- [📋 Prerrequisitos](#-prerrequisitos)
- [Arquitectura en AWS](#arquitectura-en-aws)
- [🐳 Imágenes en Docker Hub](#-imágenes-en-docker-hub)
- [🛠️ Proceso de Despliegue en AWS EKS](#️-proceso-de-despliegue-en-aws-eks)
  - [Fase 1: Configurar Herramientas Locales](#fase-1-configurar-herramientas-locales)
  - [Fase 2: Crear el Clúster de Kubernetes (EKS)](#fase-2-crear-el-clúster-de-kubernetes-eks)
  - [Fase 3: Configurar la Red (VPC Peering)](#fase-3-configurar-la-red-vpc-peering)
  - [Fase 4: Desplegar la Aplicación](#fase-4-desplegar-la-aplicación)
- [🗃️ Migración de Base de Datos (Opcional)](#️-migración-de-base-de-datos-opcional)
- [💻 Acceder a la Aplicación](#-acceder-a-la-aplicación)
- [🗑️ Limpieza de Recursos](#️-limpieza-de-recursos)
- [📄 Licencia](#-licencia)

---

## 📋 Prerrequisitos
- **Una cuenta de AWS** con permisos de administrador.
- **Git** — Para clonar el repositorio.
- **AWS CLI** — Para interactuar con tu cuenta de AWS.
- **eksctl** — Para crear y gestionar el clúster de EKS.
- **kubectl** — Para gestionar los recursos de Kubernetes.

---

## Arquitectura en AWS
Este despliegue crea la siguiente arquitectura:
- Un **Clúster de EKS** que aloja los contenedores del frontend y el backend.
- Una **Base de Datos RDS** para MySQL, desacoplada del clúster para mayor estabilidad.
- Una conexión **VPC Peering** para comunicar de forma segura el clúster y la base de datos.
- Un **Load Balancer** de AWS para exponer el frontend a internet.

---

## 🐳 Imágenes en Docker Hub
Los manifiestos usan las siguientes imágenes públicas:

| Componente | Imagen en Docker Hub | Versión |
|------------|----------------------|---------|
| **Frontend** | `kennysolo/invopay-frontend` | `1.0.0` |
| **Backend** | `kennysolo/invopay-backend` | `1.0.2` |

---

## 🛠️ Proceso de Despliegue en AWS EKS

### Fase 1: Configurar Herramientas Locales

1. **Configura la AWS CLI:**
    ```bash
    aws configure
    ```

2. **Clona el repositorio:**
    ```bash
    git clone https://github.com/kennysolorzano/invopay-kubernetes.git
    cd invopay-kubernetes
    ```

### Fase 2: Crear el Clúster de Kubernetes (EKS)

```bash
eksctl create cluster \
--name invopay-cluster \
--region us-east-2 \
--nodegroup-name standard-workers \
--node-type t3.small \
--nodes 1 \
--managed
```

### Fase 3: Configurar la Red (VPC Peering)

1. Identifica las VPCs de EKS y RDS y sus rangos CIDR.
2. Crea y acepta una conexión de VPC Peering desde la consola de AWS.
3. Actualiza las rutas y grupos de seguridad para permitir tráfico en el puerto 3306.

### Fase 4: Desplegar la Aplicación

1. Crea un archivo `.env.prod` con las variables necesarias.
2. Ejecuta:
    ```bash
    kubectl create secret generic backend-secrets --from-env-file=.env.prod
    kubectl create secret generic google-credentials --from-file=./credentials/techforb-finsuite-key.json
    kubectl apply -f k8s/backend-deployment.yaml
    kubectl apply -f k8s/frontend-deployment.yaml
    ```

---

## 🗃️ Migración de Base de Datos (Opcional)

### Fase A: Crear el Esquema
Despliega temporalmente el backend con `ddl-auto: update`.

### Fase B: Importar los Datos

```bash
mysqldump -u [usuario] -p'[contraseña]' --no-create-info --skip-triggers [nombre_db_local] > backup-data-only.sql

kubectl apply -f k8s/importer-pod.yaml
kubectl cp backup-data-only.sql mysql-importer:/tmp/backup.sql
kubectl exec -it mysql-importer -- /bin/bash
mysql -h [endpoint-rds] -u [usuario-rds] -p'[contraseña-rds]' database-app < /tmp/backup.sql
exit
kubectl delete pod mysql-importer
```

---

## 💻 Acceder a la Aplicación

```bash
kubectl get service frontend-service -w
```
Accede a la `EXTERNAL-IP` en tu navegador.

---

## 🗑️ Limpieza de Recursos

```bash
eksctl delete cluster --name invopay-cluster --region us-east-2
```

---

## 📄 Licencia

Este proyecto está licenciado bajo la Licencia MIT. Eres libre de usar, modificar y distribuir el código. Para más detalles, consulta el archivo [LICENSE](./LICENSE).
