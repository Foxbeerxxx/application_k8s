# Домашнее задание к занятию "`Настройка приложений и управление доступом в Kubernetes`" - `Татаринцев Алексей`

---

### Задание 1



1. `Пишу манифест deployment.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: web-content
          mountPath: /usr/share/nginx/html
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 5
      - name: multitool
        image: wbitt/network-multitool:alpine-minimal
        env:
        - name: HTTP_PORT
          value: "8080"
        ports:
        - containerPort: 8080
      volumes:
      - name: web-content
        configMap:
          name: web-content
---
apiVersion: v1
kind: Service
metadata:
  name: web-app-svc
  labels:
    app: web-app
spec:
  selector:
    app: web-app
  ports:
  - name: http
    port: 80
    targetPort: 80

```


2. `Пишу манифест configmap-web.yaml`

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-content
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8"/>
      <title>Страница из ConfigMap</title>
    </head>
    <body>
      <h1>Привет от Kubernetes!</h1>
      <p>Содержимое отдано через ConfigMap.</p>
    </body>
    </html>

```
3. `Запускаю манифесты и проверяю`

```
kubectl apply -f configmap-web.yaml
kubectl apply -f deployment.yaml
kubectl port-forward svc/web-app-svc 8080:80

можно через curl
curl -s http://localhost:8080 | head
```
![2](https://github.com/Foxbeerxxx/application_k8s/blob/main/img/img2.png)

![1](https://github.com/Foxbeerxxx/application_k8s/blob/main/img/img1.png)


---

### Задание 2

1. `Подготовка сертификата и можно из файлов сгенерировать манифест`
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"

# Сгенерировать манифест секрета из файлов
kubectl create secret tls tls-secret \
  --cert=tls.crt --key=tls.key \
  --dry-run=client -o yaml > secret-tls.yaml

```
![3](https://github.com/Foxbeerxxx/application_k8s/blob/main/img/img3.png)

2. `Применение манифестов`

```
kubectl apply -f secret-tls.yaml
kubectl apply -f ingress-tls.yaml
kubectl get ingress web-app-ing
```
![4](https://github.com/Foxbeerxxx/application_k8s/blob/main/img/img4.png)


3. `Добавляю в хост 127.0.0.1 myapp.example.com`
4. `Проверяю`
```
kubectl get ingress web-app-ing
curl -kI https://myapp.example.com/
```
![5](https://github.com/Foxbeerxxx/application_k8s/blob/main/img/img5.png)

---

### Задание 3


1. `Пишу манифест role-pod-reader.yaml с содержимым`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["get", "list", "watch"]

```

2. `Пишу манифест rolebinding-developer.yaml`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-pod-reader
  namespace: default
subjects:
- kind: User
  name: developer
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

```
3. `Включаю RBAC в MicroK8s`
```
sudo microk8s enable rbac
```
4. `Генерация сертификата пользователя`
```
# ключ и CSR
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"

# подпись CA кластера MicroK8s
sudo openssl x509 -req -in developer.csr \
  -CA /var/snap/microk8s/current/certs/ca.crt \
  -CAkey /var/snap/microk8s/current/certs/ca.key \
  -CAcreateserial -out developer.crt -days 365
```
![6](https://github.com/Foxbeerxxx/application_k8s/blob/main/img/img6.png)


5. `Запускаем`

```
kubectl apply -f role-pod-reader.yaml
kubectl apply -f rolebinding-developer.yaml
```
`Список подов от имени developer`

```
kubectl get pods -n default --as=developer
```
![7](https://github.com/Foxbeerxxx/application_k8s/blob/main/img/img7.png)


6. `Описание и логи`

```
POD=$(kubectl get pod -n default -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod $POD -n default --as=developer
kubectl logs $POD -n default --as=developer

вывод:

alexey@dellalexey:~/dz/application_k8s/3$ POD=$(kubectl get pod -n default -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod $POD -n default --as=developer
Name:             web-app-6c4d49df56-7g72f
Namespace:        default
Priority:         0
Service Account:  default
Node:             dellalexey/192.168.1.140
Start Time:       Wed, 24 Sep 2025 19:16:16 +0300
Labels:           app=web-app
                  pod-template-hash=6c4d49df56
Annotations:      cni.projectcalico.org/containerID: bf753998120986a7cc60963aa569172ee2823798ea9efc671edccad44d2a14cb
                  cni.projectcalico.org/podIP: 10.1.212.219/32
                  cni.projectcalico.org/podIPs: 10.1.212.219/32
Status:           Running
IP:               10.1.212.219
IPs:
  IP:           10.1.212.219
Controlled By:  ReplicaSet/web-app-6c4d49df56
Containers:
  nginx:
    Container ID:   containerd://57e56e1e9dd76d366c4e1fa3f7e92d6131dc13090514429dfb8fa532482511ea
    Image:          nginx:1.25-alpine
    Image ID:       docker.io/library/nginx@sha256:516475cc129da42866742567714ddc681e5eed7b9ee0b9e9c015e464b4221a00
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 24 Sep 2025 19:16:24 +0300
    Ready:          True
    Restart Count:  0
    Readiness:      http-get http://:80/ delay=3s timeout=1s period=5s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from web-content (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lxp88 (ro)
  multitool:
    Container ID:   containerd://3142d07d5f62e6cb236e02ee91c50dad70b1083ff5b12efcfc261d40b123c63b
    Image:          wbitt/network-multitool:alpine-minimal
    Image ID:       docker.io/wbitt/network-multitool@sha256:33914dbb8d3cf046d3971e71f77bd8af0cdd4f7e67a8a3449e0420242f55963c
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 24 Sep 2025 19:16:25 +0300
    Ready:          True
    Restart Count:  0
    Environment:
      HTTP_PORT:  8080
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-lxp88 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  web-content:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      web-content
    Optional:  false
  kube-api-access-lxp88:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>

```
![8](https://github.com/Foxbeerxxx/application_k8s/blob/main/img/img8.png)