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

5. `Заполните здесь этапы выполнения, если требуется ....`
6. 

```
Поле для вставки кода...
....
....
....
....
```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота 2](ссылка на скриншот 2)`


---

### Задание 3

`Приведите ответ в свободной форме........`

1. `Заполните здесь этапы выполнения, если требуется ....`
2. `Заполните здесь этапы выполнения, если требуется ....`
3. `Заполните здесь этапы выполнения, если требуется ....`
4. `Заполните здесь этапы выполнения, если требуется ....`
5. `Заполните здесь этапы выполнения, если требуется ....`
6. 

```
Поле для вставки кода...
....
....
....
....
```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота](ссылка на скриншот)`

### Задание 4

`Приведите ответ в свободной форме........`

1. `Заполните здесь этапы выполнения, если требуется ....`
2. `Заполните здесь этапы выполнения, если требуется ....`
3. `Заполните здесь этапы выполнения, если требуется ....`
4. `Заполните здесь этапы выполнения, если требуется ....`
5. `Заполните здесь этапы выполнения, если требуется ....`
6. 

```
Поле для вставки кода...
....
....
....
....
```

`При необходимости прикрепитe сюда скриншоты
![Название скриншота](ссылка на скриншот)`
