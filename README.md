# 🚀 Despliegue de InvoPay en Kubernetes [![Frontend en Docker Hub](https://img.shields.io/badge/Docker%20Hub-invopay--frontend-blue?logo=docker)](https://hub.docker.com/r/kennysolo/invopay-frontend) [![Backend en Docker Hub](https://img.shields.io/badge/Docker%20Hub-invopay--backend-blue?logo=docker)](https://hub.docker.com/r/kennysolo/invopay-backend) [![Licencia MIT](https://img.shields.io/badge/Licencia-MIT-green)](LICENSE)

Repositorio oficial con la configuración necesaria para desplegar **InvoPay**, una solución compuesta por un frontend en Angular y un backend en Spring Boot, sobre un entorno Kubernetes.

---

## 📑 Tabla de Contenidos
- [📋 Prerrequisitos](#-prerrequisitos)
- [🐳 Imágenes en Docker Hub](#-imágenes-en-docker-hub)
- [🛠️ Proceso de Despliegue](#️-proceso-de-despliegue)
  - [1️⃣ Clonar el repositorio](#1️⃣-clonar-el-repositorio)
  - [2️⃣ Configurar los Secretos](#2️⃣-configurar-los-secretos)
  - [3️⃣ Desplegar la aplicación](#3️⃣-desplegar-la-aplicación)
  - [4️⃣ Verificar el despliegue](#4️⃣-verificar-el-despliegue)
- [💻 Acceder a la aplicación](#-acceder-a-la-aplicación)
  - [🔹 Opción A — Minikube (Desarrollo)](#-opción-a--minikube-desarrollo)
  - [🔹 Opción B — Clúster en la Nube (Producción)](#-opción-b--clúster-en-la-nube-producción)
- [🔄 Flujo de Trabajo para Desarrollo](#-flujo-de-trabajo-para-desarrollo)
- [📄 Licencia](#-licencia)

---

## 📋 Prerrequisitos

Asegúrate de contar con las siguientes herramientas instaladas:

- **Git** — Para clonar el repositorio.
- **Docker** — Para construir imágenes si modificas el código fuente.
- **kubectl** — Para gestionar los recursos de Kubernetes.
- **Minikube** *(Opcional)* — Para pruebas en entorno local.

---

## 🐳 Imágenes en Docker Hub

Las siguientes imágenes públicas están listas para su uso en Kubernetes:

| Componente | Imagen |
|------------|--------|
| **Frontend** | [kennysolo/invopay-frontend](https://hub.docker.com/r/kennysolo/invopay-frontend) |
| **Backend** | [kennysolo/invopay-backend](https://hub.docker.com/r/kennysolo/invopay-backend) |

---

## 🛠️ Proceso de Despliegue

### 1️⃣ Clonar el repositorio

```bash
git clone https://github.com/kennysolorzano/invopay-kubernetes.git
cd invopay-kubernetes
```

### 2️⃣ Configurar los Secretos

1. Copia la plantilla:
```bash
cp k8s/secrets.template.yaml k8s/secrets.yaml
```

2. Codifica y reemplaza los valores en `k8s/secrets.yaml`:

- Para texto plano:
```bash
echo -n 'tu-valor-secreto' | base64
```

- Para un archivo (ej.: JSON de credenciales):
```bash
cat /ruta/a/tu/archivo.json | base64 | tr -d '\n'
```

3. Aplica los secretos al clúster:
```bash
kubectl apply -f k8s/secrets.yaml
```

---

### 3️⃣ Desplegar la aplicación

```bash
kubectl apply -f k8s/
```

---

### 4️⃣ Verificar el despliegue

Monitorea los pods hasta que estén en **Running**:
```bash
kubectl get pods -w
```

Presiona `Ctrl + C` para salir cuando todo esté en ejecución.

---

## 💻 Acceder a la aplicación

### 🔹 Opción A — Minikube (Desarrollo)

1. Reenvía el puerto del frontend:
```bash
kubectl port-forward service/frontend-service 8081:80
```

2. Accede desde tu navegador:
```
http://localhost:8081
```

---

### 🔹 Opción B — Clúster en la Nube (Producción)

1. Cambia el tipo de servicio a **LoadBalancer** en `k8s/frontend-deployment.yaml`:
```yaml
spec:
  type: LoadBalancer
```

2. Aplica los cambios:
```bash
kubectl apply -f k8s/frontend-deployment.yaml
```

3. Consulta la IP pública:
```bash
kubectl get service frontend-service -w
```

Accede desde el navegador cuando la **EXTERNAL-IP** esté disponible.

---

## 🔄 Flujo de Trabajo para Desarrollo

Si realizas cambios en el código:

1. Reconstruye la imagen:
```bash
docker-compose build backend
```

2. Carga la nueva imagen en Minikube:
```bash
minikube image load invopay_backend:latest
```

3. Reinicia el deployment para aplicar los cambios:
```bash
kubectl rollout restart deployment backend-deployment
```

---

## 📄 Licencia

Este proyecto está licenciado bajo la **Licencia MIT**, lo que significa que eres libre de usar, modificar y distribuir el código, siempre y cuando mantengas el aviso de derechos de autor y la licencia en las copias del proyecto. Puedes consultar el texto completo en el archivo [LICENSE](LICENSE).
