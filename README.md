
# Laboratorio: Instalación de Prometheus, Grafana y Aplicación Node.js para Métricas PromQL en un Dashboard

Este laboratorio proporciona una guía paso a paso para instalar **Prometheus**, **Grafana** y la aplicación **Node.js** utilizando **Helm**. La aplicación Node.js proporcionará datos que se pueden visualizar en el dashboard de Grafana utilizando consultas PromQL.

## Requisitos previos

- **Kubernetes** con acceso de administrador.
- **Helm** instalado.
- Acceso a la terminal para ejecutar comandos `kubectl` y `helm`.

## 1. Creación de Namespaces

Primero, crearemos los namespaces donde desplegaremos Prometheus, Grafana y la aplicación Node.js.

```bash
kubectl create namespace prometheus
kubectl create namespace grafana
kubectl create namespace app
```

## 2. Instalación de Prometheus

### Paso 1: Añadir el repositorio de Helm para Prometheus

Ejecuta el siguiente comando para añadir el repositorio de Helm para Prometheus.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Paso 2: Desplegar Prometheus en el Namespace `prometheus`

Usa el siguiente comando para instalar Prometheus en el namespace `prometheus`.

```bash
helm install prometheus prometheus-community/prometheus --namespace prometheus
```

### Paso 3: Verificar el Despliegue

Puedes verificar que Prometheus esté corriendo con el siguiente comando:

```bash
kubectl get all -n prometheus
```

## 3. Instalación de Grafana

### Paso 1: Añadir el repositorio de Helm para Grafana

Ejecuta el siguiente comando para añadir el repositorio de Helm para Grafana.

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### Paso 2: Desplegar Grafana en el Namespace `grafana`

Instala Grafana en el namespace `grafana` utilizando Helm.

```bash
helm install grafana grafana/grafana --namespace grafana
```

### Paso 3: Exponer Grafana (opcional)

Para acceder a Grafana desde tu navegador, puedes exponer el servicio utilizando un **LoadBalancer** o un **Port-Forward**. Aquí mostramos cómo hacer port-forwarding:

```bash
kubectl port-forward svc/grafana 3000:80 -n grafana
```

Luego, accede a Grafana en [http://localhost:3000](http://localhost:3000). El nombre de usuario predeterminado es `admin` y la contraseña puede obtenerse con el siguiente comando:

```bash
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

## 4. Instalación de la Aplicación Node.js

### Paso 1: Desplegar la Aplicación Node.js

Ejecuta el siguiente comando para instalar la aplicación Node.js usando el chart **nodejs-app-aroldev-infra**:

```bash
helm install nodejs-app ./nodejs-app-aroldev-infra --namespace app
```

Esto desplegará la aplicación en el namespace `app`, proporcionando métricas que podrás visualizar en Grafana.

### Paso 2: Verificar el Despliegue

Puedes verificar que la aplicación Node.js esté corriendo correctamente con el siguiente comando:

```bash
kubectl get all -n app
```

## 5. Configuración de Dashboard en Grafana

### Paso 1: Conectar Grafana con Prometheus

Una vez que hayas iniciado sesión en Grafana:

1. Dirígete a **Configuration** > **Data Sources**.
2. Haz clic en **Add data source**.
3. Selecciona **Prometheus**.
4. En el campo **URL**, ingresa la dirección del servicio Prometheus dentro del cluster: `http://prometheus-server.prometheus.svc.cluster.local:9090`.
5. Guarda la configuración.

### Paso 2: Crear un Dashboard con Métricas PromQL

1. Ve a **Create** > **Dashboard**.
2. Haz clic en **Add New Panel**.
3. En el campo **Query**, introduce las siguientes métricas de **PromQL** como ejemplo:

   - **Carga promedio del sistema (load average)**:

     ```promql
     node_load1
     ```

   - **Uso de CPU por nodo**:

     ```promql
     sum(rate(node_cpu_seconds_total{mode!="idle"}[5m])) by (instance)
     ```

   - **Memoria usada**:

     ```promql
     node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
     ```

4. Personaliza los gráficos y ajusta las opciones de visualización.
5. Guarda el dashboard.

### Paso 3: Guardar y compartir el Dashboard

Guarda el dashboard y asígnale un nombre significativo para su uso posterior. También puedes exportar el dashboard como JSON para compartirlo o reutilizarlo en otros entornos.

## 6. Verificación

Una vez que todo esté configurado:

- Asegúrate de que el servicio de Prometheus esté recopilando métricas correctamente.
- Accede a tu nuevo dashboard en Grafana para visualizar las métricas de PromQL configuradas, incluyendo las de la aplicación Node.js.

## 7. Desinstalación

Si deseas desinstalar Prometheus, Grafana y la aplicación Node.js, puedes hacerlo con los siguientes comandos:

```bash
helm uninstall prometheus --namespace prometheus
helm uninstall grafana --namespace grafana
helm uninstall nodejs-app --namespace app
kubectl delete namespace prometheus
kubectl delete namespace grafana
kubectl delete namespace app
```

## Conclusión

Este laboratorio cubre los pasos para la instalación de **Prometheus**, **Grafana** y una aplicación **Node.js** utilizando Helm. También incluye la creación de un dashboard básico en Grafana para visualizar métricas utilizando consultas PromQL. ¡Explora más métricas y configura dashboards personalizados según tus necesidades!