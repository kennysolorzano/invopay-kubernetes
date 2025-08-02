# 🚀 Despliegue de InvoPay en Kubernetes

[![Licencia: MIT](https://img.shields.io/badge/Licencia-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker Hub Frontend](https://img.shields.io/badge/Docker%20Hub-invopay--frontend-blue?logo=docker)](https://hub.docker.com/r/kennysolo/invopay-frontend)
[![Docker Hub Backend](https://img.shields.io/badge/Docker%20Hub-invopay--backend-blue?logo=docker)](https://hub.docker.com/r/kennysolo/invopay-backend)

Repositorio oficial con toda la configuración de Kubernetes necesaria para desplegar la aplicación **InvoPay**, una solución compuesta por un frontend en Angular y un backend en Spring Boot, sobre un entorno de producción en AWS EKS con una base de datos RDS.

---

## 📑 Tabla de Contenidos
- [📋 Prerrequisitos](#-prerrequisitos)
- [📝 Resumen del Proceso: De Local a la Nube](#️-resumen-del-proceso-de-local-a-la-nube)
- [🛠️ Proceso de Despliegue Detallado](#️-proceso-de-despliegue-detallado)
  - [Fase 1: Configurar el Entorno Local](#fase-1-configurar-el-entorno-local)
  - [Fase 2: Desplegar la Infraestructura en AWS](#fase-2-desplegar-la-infraestructura-en-aws)
  - [Fase 3: Configurar la Conectividad de Red](#fase-3-configurar-la-conectividad-de-red)
  - [Fase 4: Desplegar la Aplicación](#fase-4-desplegar-la-aplicación)
- [🗃️ Migración de Base de Datos (Opcional)](#️-migración-de-base-de-datos-opcional)
- [💻 Acceder a la Aplicación](#-acceder-a-la-aplicación)
- [🗑️ Limpieza de Recursos](#️-limpieza-de-recursos)
- [📄 Licencia](#-licencia)

---

## 📋 Prerrequisitos

### 1. Herramientas de Software
- **Git** — Para clonar el repositorio.
- **AWS CLI** — Para interactuar con tu cuenta de AWS.
- **eksctl** — La herramienta oficial para crear y gestionar clústeres de EKS.
- **kubectl** — Para gestionar los recursos de Kubernetes.

### 2. Permisos de AWS (IAM)
Para ejecutar este despliegue, necesitas un usuario de IAM (no el usuario root de la cuenta) con la política `AdministratorAccess`.

**¿Por qué se necesitan permisos de administrador?**
La herramienta `eksctl` crea una infraestructura de red compleja en tu nombre (VPCs, Subnets, Gateways, Roles de IAM, Grupos de Seguridad, Instancias EC2 para los nodos, etc.). Otorgar `AdministratorAccess` al usuario de la CLI asegura que `eksctl` tenga todos los permisos necesarios para completar estas tareas sin fallar. Para un entorno de producción más estricto, se pueden definir políticas más granulares.

---

## 📝 Resumen del Proceso: De Local a la Nube

El proceso de migración de un entorno local a uno de producción en la nube como AWS EKS implica varios pasos clave que esta guía detalla:

1. **Publicar las Imágenes**  
   Sube las imágenes construidas localmente a Docker Hub para que puedan ser accedidas desde AWS.

2. **Adaptar los Manifiestos**  
   Asegúrate de que los archivos `.yaml` de Kubernetes apunten a las URLs públicas de Docker Hub.

3. **Crear la Infraestructura**  
   Usa `eksctl` para crear un clúster EKS gestionado.

4. **Configurar la Red**  
   Conecta el clúster con la base de datos RDS mediante VPC Peering.

5. **Gestionar Secretos de Producción**  
   Usa archivos `.env` para inyectar variables de entorno en los pods de Kubernetes.

6. **Exponer la Aplicación**  
   Usa el tipo de servicio `LoadBalancer` para obtener una IP pública desde AWS.

---

## 🛠️ Proceso de Despliegue Detallado

### Fase 1: Configurar el Entorno Local
```bash
# Configura la AWS CLI
aws configure
# Clona este repositorio
git clone https://github.com/kennysolorzano/invopay-kubernetes.git
cd invopay-kubernetes
```

### Fase 2: Desplegar la Infraestructura en AWS
```bash
eksctl create cluster   --name invopay-cluster   --region us-east-2   --nodegroup-name standard-workers   --node-type t3.small   --nodes 1   --managed
```

```bash
# Verifica la conexión al clúster
kubectl get nodes
```

### Fase 3: Configurar la Conectividad de Red

1. Identifica las VPCs (EKS y RDS) y sus CIDRs.
2. Crea una conexión de VPC Peering.
3. Actualiza las tablas de rutas de ambas VPCs.
4. Modifica el grupo de seguridad de RDS para aceptar conexiones desde el CIDR del clúster.

### Fase 4: Desplegar la Aplicación
```bash
# Crea archivo de entorno
touch .env.prod
# Agrega DB_URL, DB_USER, DB_PASSWORD

# Crea los secretos
kubectl create secret generic backend-secrets --from-env-file=.env.prod
kubectl create secret generic google-credentials --from-file=./credentials/techforb-finsuite-key.json

# Despliega los manifiestos
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml
```

---

## 🗃️ Migración de Base de Datos (Opcional)
Puedes usar herramientas como DBeaver o `mysqldump` para migrar tus datos de una base local a la base de datos RDS en AWS.

---

## 💻 Acceder a la Aplicación
```bash
# Espera a que el servicio publique la IP externa
kubectl get service frontend-service -w
```

Abre la dirección que aparece en `EXTERNAL-IP` en tu navegador.

---

## 🗑️ Limpieza de Recursos
```bash
eksctl delete cluster --name invopay-cluster --region us-east-2
```

---

## 📄 Licencia

Este proyecto está licenciado bajo los términos de la Licencia MIT.

Eso significa que puedes utilizar, copiar, modificar, fusionar, publicar, distribuir, sublicenciar y/o vender copias del software, siempre y cuando incluyas el aviso de copyright original.

Consulta el archivo [LICENSE](./LICENSE) para más información.
