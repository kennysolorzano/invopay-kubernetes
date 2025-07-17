# Despliegue de InvoPay en Kubernetes

Este repositorio contiene toda la configuración de Kubernetes necesaria para desplegar la aplicación InvoPay, una plataforma completa con un frontend en Angular y un backend en Spring Boot.

![Portal de Administración de InvoPay](https://i.imgur.com/g0j8q3m.jpeg)

---

## 📋 Prerrequisitos

Antes de comenzar, asegúrate de tener las siguientes herramientas instaladas y configuradas en tu sistema:

* **Git:** Para clonar el repositorio.
* **Docker:** Para construir las imágenes de contenedor si realizas cambios en el código.
* **kubectl:** La herramienta de línea de comandos para interactuar con tu clúster de Kubernetes.
* **Minikube:** (Opcional) Para un despliegue de desarrollo en tu máquina local.

---

## 🐳 Imágenes de Docker Hub

Las imágenes de contenedor pre-construidas para este proyecto están disponibles públicamente en Docker Hub. Los manifiestos de Kubernetes ya están configurados para utilizarlas.

* **Frontend:** [hub.docker.com/r/kennysolo/invopay-frontend](https://hub.docker.com/r/kennysolo/invopay-frontend)
* **Backend:** [hub.docker.com/r/kennysolo/invopay-backend](https://hub.docker.com/r/kennysolo/invopay-backend)

---

## 🚀 Proceso de Despliegue

Sigue estos pasos para desplegar la aplicación en cualquier clúster de Kubernetes.

### Paso 1: Clonar el Repositorio

Primero, clona este repositorio en tu máquina local.

```bash
git clone [https://github.com/kennysolorzano/invopay-kubernetes.git](https://github.com/kennysolorzano/invopay-kubernetes.git)
cd invopay-kubernetes

Paso 2: Configurar los Secretos
La seguridad es lo primero. La información sensible como contraseñas y llaves de API se gestiona a través de un archivo de secretos que nunca se sube a Git.

a. Crear el archivo de secretos a partir de la plantilla:

El repositorio incluye una plantilla secrets.template.yaml. Cópiala para crear tu propio archivo de secretos.

cp k8s/secrets.template.yaml k8s/secrets.yaml

(Este archivo secrets.yaml ya está incluido en el .gitignore para tu seguridad).

b. Rellenar k8s/secrets.yaml con tus valores:

Abre el archivo k8s/secrets.yaml con un editor de texto. Verás una lista de claves que debes rellenar con tus valores de producción, codificados en Base64.

¿Cómo codificar en Base64?

Para texto: echo -n 'tu-valor-secreto' | base64

Para archivos (como el JSON de Google): cat /ruta/a/tu/archivo.json | base64

c. Aplicar los secretos al clúster:

Una vez que hayas guardado tus cambios en k8s/secrets.yaml, aplícalos a tu clúster de Kubernetes.

kubectl apply -f k8s/secrets.yaml

Paso 3: Desplegar la Aplicación
Con los secretos ya en el clúster, despliega todos los componentes de la aplicación (Deployments, Services, etc.) con un solo comando.

kubectl apply -f k8s/

Paso 4: Verificar el Despliegue
Observa el estado de los pods hasta que ambos estén en estado Running y READY 1/1. Esto puede tardar unos minutos, especialmente la primera vez mientras se descargan las imágenes.

kubectl get pods -w

Una vez que ambos pods estén listos, presiona Ctrl+C para salir.

💻 Acceder a la Aplicación
El método de acceso depende de tu entorno de Kubernetes.

Opción A: Entorno Local (Minikube)
La forma más fiable de acceder a la aplicación en Minikube (especialmente en Windows con WSL) es creando un túnel de red directo. Esto soluciona problemas comunes de red y CORS.

Inicia el túnel:
Este comando se quedará corriendo en tu terminal para mantener la conexión abierta.

kubectl port-forward service/frontend-service 8081:80

Abre tu navegador:
Ve a http://localhost:8081.

Opción B: Entorno de Producción (Google Cloud, AWS, Azure)
En un clúster en la nube, la forma estándar de exponer una aplicación a internet es usando un servicio de tipo LoadBalancer.

Edita k8s/frontend-deployment.yaml:
Cambia el tipo de servicio de NodePort a LoadBalancer.

# En k8s/frontend-deployment.yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer # <-- CAMBIA ESTO

Aplica el cambio y obtén la IP externa:

kubectl apply -f k8s/frontend-deployment.yaml
kubectl get service frontend-service -w

Espera a que la columna EXTERNAL-IP muestre una dirección IP pública. Esto puede tardar unos minutos.

Abre tu navegador y ve a la http://<EXTERNAL-IP> que obtuviste.

🛠️ Flujo de Trabajo para Desarrollo
Si realizas cambios en el código fuente, sigue estos pasos para actualizar tu entorno de desarrollo en Minikube:

Reconstruye la imagen modificada:

# Ejemplo para el backend
docker-compose build backend

Carga la nueva imagen a Minikube:

minikube image load invopay_backend:latest

Reinicia el deployment para que use la nueva imagen:

kubectl rollout restart deployment backend-deployment

</markdown>
