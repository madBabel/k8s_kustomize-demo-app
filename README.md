# k8s_kustomice_nginx

Este proyecto contiene la configuración de despliegue en Kubernetes utilizando **Kustomize** para los microservicios `ms_api` y `ms_web`, con un **reverse proxy** basado en **NGINX** y soporte para **Contour** mediante `HTTPProxy`.

## 📂 Estructura del proyecto

```
k8s_kustomice_nginx/
├── base/                  # Manifiestos base
│   ├── kustomization.yaml
│   ├── namespace.yaml
│   ├── HTTPProxy.yaml
│   ├── ms_api/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── ms_web/
│       ├── deployment.yaml
│       └── service.yaml
└── overlays/
    ├── dev/               # Overlay para entorno de desarrollo
    └── prod/              # Overlay para entorno de producción
```

## 🚀 Prerrequisitos

Antes de ejecutar, asegúrate de tener instalado:

- [kubectl](https://kubernetes.io/docs/tasks/tools/)  
- [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/)  
- Un cluster de Kubernetes (local con [minikube](https://minikube.sigs.k8s.io/docs/) o remoto)  
- (Opcional) [Contour](https://projectcontour.io/) para probar el recurso `HTTPProxy`  

Verifica las versiones:

```bash
kubectl version --client
kustomize version
```

## ⚙️ Despliegue en Kubernetes


### 1. Aplicar configuración con Kustomize

Ejecuta el despliegue en el namespace definido en el overlay (`dev` o `prod`):

```bash
kubectl apply -k overlays/dev
```

O en el caso de prod

```bash
kubectl apply -k overlays/prod
```

Esto desplegará:

- `ms_api` (Deployment + Service)  
- `ms_web` (Deployment + Service)  
- `HTTPProxy` para exponer el tráfico a través de Contour/NGINX  

### 2. Verificar recursos

```bash
kubectl get all -n demo-app
```

### 4. Revisar logs

```bash
kubectl logs -l app=ms-api -n demo-app
kubectl logs -l app=ms-web -n demo-app
```

## 🌐 Probar con Contour y HTTPProxy



Si tienes instalado **Contour**, puedes probar el tráfico HTTPProxy haciendo un **port-forward** al `envoy`:

Para instalarlo, puedes hacerlo con:
```bash
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```
Y luego exponer el servicio `envoy`:

```bash
kubectl -n projectcontour port-forward service/envoy 8888:80
```

Luego puedes probar los endpoints con `curl`:

```bash
curl -H "Host: demo.local" http://localhost:8888/web
curl -H "Host: demo.local" http://localhost:8888/api
```


## 🧹 Limpiar recursos

Para eliminar todo el despliegue del entorno `dev` o `prod`:

```bash
kubectl delete -k overlays/dev
kubectl delete -k overlays/prod
```
