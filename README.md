# Despliegue de InvoPay en Kubernetes

Este repositorio contiene todo lo necesario para desplegar la aplicación InvoPay, compuesta por un frontend en Angular y un backend en Spring Boot, en un clúster de Kubernetes.

![Portal de Administración de InvoPay](https://i.imgur.com/g0j8q3m.jpeg)

---

## 📋 Prerrequisitos

Para desplegar esta aplicación, necesitarás las siguientes herramientas instaladas y configuradas:

* **Git:** Para clonar el repositorio.
* **Docker:** Para construir las imágenes de contenedor (solo si se realizan cambios en el código).
* **kubectl:** La herramienta de línea de comandos para interactuar con Kubernetes.
* **Minikube:** (Opcional) Para un despliegue local en tu propia máquina.

---

## 🐳 Imágenes de Docker Hub

Las imágenes de contenedor pre-construidas para este proyecto están disponibles públicamente en Docker Hub. Los manifiestos de Kubernetes ya están configurados para usarlas.

* **Frontend:** [hub.docker.com/r/kennysolo/invopay-frontend](https://hub.docker.com/r/kennysolo/invopay-frontend)
* **Backend:** [hub.docker.com/r/kennysolo/invopay-backend](https://hub.docker.com/r/kennysolo/invopay-backend)

---

## 🚀 Proceso de Despliegue

Sigue estos pasos para desplegar la aplicación en cualquier clúster de Kubernetes.

### 1. Clonar el Repositorio

```bash
git clone [https://github.com/kennysolorzano/invopay-kubernetes.git](https://github.com/kennysolorzano/invopay-kubernetes.git)
cd invopay-kubernetes

2. Configurar los Secretos
Este es el paso manual más importante. Nunca debes subir secretos a Git. En su lugar, usarás una plantilla para crear un archivo de secretos local.

a. Copia la plantilla de secretos:

cp k8s/secrets.template.yaml k8s/secrets.yaml

b. Edita k8s/secrets.yaml y añade tus valores:
Abre el archivo k8s/secrets.yaml con un editor de texto. Verás una lista de claves con valores vacíos o de ejemplo. Debes reemplazarlos con tus valores de producción reales, codificados en Base64.

¿Cómo codificar en Base64?

Para texto: echo -n 'tu-valor-secreto' | base64

Para archivos (como el JSON de Google): cat /ruta/a/tu/archivo.json | base64

c. Aplica los secretos al clúster:
Una vez que hayas guardado tus cambios en k8s/secrets.yaml, aplícalos a Kubernetes:

kubectl apply -f k8s/secrets.yaml

3. Desplegar la Aplicación
Aplica todos los manifiestos de la carpeta k8s con un solo comando. Esto creará los deployments, servicios y configuraciones necesarias.

kubectl apply -f k8s/

4. Verificar el Despliegue
Observa el estado de los pods. Espera a que ambos estén en estado Running y READY 1/1. Esto puede tardar unos minutos, especialmente la primera vez.

kubectl get pods -w

💻 Acceder a la Aplicación
El método de acceso depende de tu entorno.

Entorno Local (Minikube)
La forma más fiable de acceder a la aplicación en Minikube es creando un túnel de red directo.

Inicia el túnel:
Este comando se quedará corriendo en tu terminal para mantener la conexión.

kubectl port-forward service/frontend-service 8081:80

Abre tu navegador:
Ve a http://localhost:8081.

Entorno de Producción (Google Cloud, AWS, Azure)
En un clúster en la nube, debes exponer el frontend a internet usando un LoadBalancer.

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

Espera a que la columna EXTERNAL-IP muestre una dirección IP pública.

Abre tu navegador y ve a la EXTERNAL-IP que obtuviste.

🛠️ Flujo de Trabajo para Desarrollo Local
Si realizas cambios en el código, sigue estos pasos para actualizar tu entorno de Minikube:

Inicia Minikube: minikube start

Construye las imágenes: docker-compose build

Carga las imágenes a Minikube:

minikube image load invopay_frontend:latest
minikube image load invopay_backend:latest

Reinicia los deployments para que usen las nuevas imágenes:

kubectl rollout restart deployment frontend-deployment
kubectl rollout restart deployment backend-deployment
