# Домашнее задание к занятию «Сетевое взаимодействие в Kubernetes»

### Цель задания

Научиться настраивать доступ к приложениям в Kubernetes:
- Внутри кластера через **Service** (ClusterIP, NodePort).
- Снаружи кластера через **Ingress**.

Это задание поможет вам освоить базовые принципы сетевого взаимодействия в Kubernetes — ключевого навыка для работы с кластерами.
На практике Service и Ingress используются для доступа к приложениям, балансировки нагрузки и маршрутизации трафика. Понимание этих механизмов поможет вам упростить управление сервисами в рабочих окружениях и снизит риски ошибок при развёртывании.

------

## **Задание 1: Настройка Service (ClusterIP и NodePort)**
### **Задача**
Развернуть приложение из двух контейнеров (`nginx` и `multitool`) и обеспечить доступ к ним:
- Внутри кластера через **ClusterIP**.
- Снаружи через **NodePort**.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами:
   - `nginx` (порт `80`).
   - `multitool` (порт `8080`).
   - Количество реплик: `3`.
2. **Создать Service типа ClusterIP**, который:
   - Открывает `nginx` на порту `9001`.
   - Открывает `multitool` на порту `9002`.
3. **Проверить доступность** изнутри кластера:
```bash
 kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh
 curl <service-name>:9001 # Проверить nginx
 curl <service-name>:9002 # Проверить multitool
```
4. **Создать Service типа NodePort** для доступа к `nginx` снаружи.
5. **Проверить доступ** с локального компьютера:
```bash
 curl <node-ip>:<node-port>
   ```
 или через браузер.

### **Что сдать на проверку**
- Манифесты:
  - `deployment-multi-container.yaml`
  - `service-clusterip.yaml`
  - `service-nodeport.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

---

## **Решение1**

Написал манифесты **deployment-multi-container** и **service-clusterip**:

[deployment-multi-container.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_1/deployment-multi-container.yaml)

<details>  
<summary> Код манифеста deployment-multi-container.yaml </summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mia-deployment
  labels:
    app: mia-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mia-deployment
  template:
    metadata:
      labels:
        app: mia-deployment
    spec:
      containers:
        - name: mia-nginx
          image: nginx:latest
          ports:
            - containerPort: 80
        - name: mia-multitool
          image: wbitt/network-multitool
          env:
            - name: HTTP_PORT
              value: "8080"
          ports:
            - containerPort: 8080
```

</details>

[service-clusterip.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_1/service-clusterip.yaml)

<details>  
<summary> Код манифеста service-clusterip.yaml </summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mia-service
spec:
  ports:
    - name: nginx
      port: 9000
      protocol: TCP
      targetPort: 80
    - name: multitool
      port: 9001
      protocol: TCP
      targetPort: 8080
  selector:
    app: mia-deployment
  type: ClusterIP
```

</details>

Проверил доступность nginx и multitool через тестовый под:

![1-3](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/screenshots/1-4.png)

Написал манифест **service-nodeport.yaml**:

[service-nodeport.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_1/service-nodeport.yaml)

<details>  
<summary> Код манифеста service-nodeport.yaml </summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  ports:
    - name: nginx
      port: 80
      protocol: TCP
      targetPort: 80
      nodePort: 30080
  selector:
    app: mia-deployment
  type: NodePort
```

</details>

Проверил доступность сервиса с локального компьютера:

![1-5](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/screenshots/1-5.png)

---

## **Задание 2: Настройка Ingress**
### **Задача**
Развернуть два приложения (`frontend` и `backend`) и обеспечить доступ к ним через **Ingress** по разным путям.

### **Шаги выполнения**
1. **Развернуть два Deployment**:
   - `frontend` (образ `nginx`).
   - `backend` (образ `wbitt/network-multitool`).
2. **Создать Service** для каждого приложения.
3. **Включить Ingress-контроллер**:
```bash
 microk8s enable ingress
   ```
4. **Создать Ingress**, который:
   - Открывает `frontend` по пути `/`.
   - Открывает `backend` по пути `/api`.
5. **Проверить доступность**:
```bash
 curl <host>/
 curl <host>/api
   ```
 или через браузер.

### **Что сдать на проверку**
- Манифесты:
  - `deployment-frontend.yaml`
  - `deployment-backend.yaml`
  - `service-frontend.yaml`
  - `service-backend.yaml`
  - `ingress.yaml`
- Скриншоты проверки доступа (`curl` или браузер).

---

## **Решение2**

Написал манифесты **deployment-frontend.yaml**, **deployment-backend.yaml**, **service-frontend.yaml**, **service-backend.yaml**:

[deployment-frontend.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_2/deployment-frontend.yaml)

<details>  
<summary> Код манифеста deployment-frontend.yaml </summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mia-frontend
  labels:
    app: mia-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mia-frontend
  template:
    metadata:
      labels:
        app: mia-frontend
    spec:
      containers:
        - name: mia-nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

</details>

[deployment-backend.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_2/)

<details>  
<summary> Код манифеста deployment-backend.yaml </summary>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mia-backend
  labels:
    app: mia-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mia-backend
  template:
    metadata:
      labels:
        app: mia-backend
    spec:
      containers:
        - name: mia-multitool
          image: wbitt/network-multitool
          env:
            - name: HTTP_PORT
              value: "80"
          ports:
            - containerPort: 80
```

</details>

[service-frontend.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_2/)

<details>  
<summary> Код манифеста service-frontend.yaml </summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mia-frontend-svc
spec:
  ports:
    - name: nginx
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: mia-frontend
  type: ClusterIP
```

[](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_2/)

</details>

[service-backend.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_2/)

<details>  
<summary> Код манифеста service-backend.yaml </summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mia-backend-svc
spec:
  ports:
    - name: multitool
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: mia-backend
  type: ClusterIP
```

</details>

Включил ingress-контролер, заметил, что подключился traefik. Попытался написать манифест по шаблону из ДЗ, но при выполнении команды `curl <host>/api` выходила ошибка 404. Залез в логи пода mia-backend и заметил, что в него приходит запрос `GET /api`, то есть правило среза пути `nginx.ingress.kubernetes.io/rewrite-target: /` не работает. Немного погуглив, нашел решение.

[ingress.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_2/ingress.yaml)

В данном манифесте указываем ingress-правило для сервиса frontend:

<details>  
<summary> Код манифеста ingress.yaml </summary>

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mia-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mia-frontend-svc
            port:
              number: 80
```

</details>

[middleware.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_2/)

В данном манифесте указываем какой путь срезать:

<details>  
<summary> Код манифеста middleware.yaml </summary>

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: mia-middleware
  namespace: default
spec:
  stripPrefix:
    prefixes:
      - "/api"
```

</details>

[ingressroute.yaml](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/task_2/)

В данном манифесте указываем ingress-правило для сервиса backend, в котором применяем правило среза пути:

<details>  
<summary> Код манифеста ingressroute.yaml </summary>

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: mia-route
  namespace: default
spec:
  entryPoints:
    - web
  routes:
    - match: PathPrefix(`/api`)
      kind: Rule
      middlewares:
      - name: mia-middleware
        namespace: default
      services:
        - name: mia-backend-svc
          port: 80
```

</details>

Проверил доступность **nginx** по `<host>/` и **multitool** по `<host>/api`:

![2-5](https://github.com/murtazinilyas/04-k8s-networking-mia/blob/main/screenshots/2-5.png)
