# «Конфигурация приложений» - Дрибноход Давид Николаевич

### Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу

1. Создать Deployment приложения, состоящего из контейнеров nginx и multitool.

#### Ответ:

Создаю новый namespace ```kubectl create namespace 12-08-hw```

Создал deployment.yaml и применил его

![image](https://github.com/DrDavidN/12-08-hw/assets/128225763/8998992d-95c4-4949-a1ac-3e3ed4ab2ea2)

Второ контейнер не запустился так как используется занятый порт, решим эту проблему через ConfigMap указав в deployment переменную HTTP_PORT

2. Решить возникшую проблему с помощью ConfigMap.

#### Ответ:

Создал ConfigMap.yaml и применил его
![image](https://github.com/DrDavidN/12-08-hw/assets/128225763/1dfe4083-a1db-4c41-93cd-ee3163abb2de)

3. Продемонстрировать, что pod стартовал и оба конейнера работают.

#### Ответ:

После применения настроек, оба контейнера стартовали

![image](https://github.com/DrDavidN/12-08-hw/assets/128225763/06e5553d-6217-4c68-a1a4-bf8401387793)

4. Сделать простую веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.

#### Ответ:

Сделал простую веб-страницу и подключу её к Nginx с помощью ConfigMap. Для этого модернизирую Deployment, добавив в него volumeMounts ссылающийся на путь по умолчанию для nginx, где находится индексная страница - /usr/share/nginx/html/, а также сошлюсь на сам ConfigMap. Подключаю Service и применяю ConfigMap.

![image](https://github.com/DrDavidN/12-08-hw/assets/128225763/5006f97b-a209-4f78-819e-5e6119d42a68)

Текст индексной страницы, написанной мной в ConfigMap и текст индексной страницы из контейнера пода одинаковы, следовательно она взята именно из содержимого ConfigMap.

5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

#### Ответ:

deployment.yaml

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool
  namespace: 12-08-hw
spec:
  selector:
    matchLabels:
      app: nmt
  replicas: 1
  template:
    metadata:
      labels:
        app: nmt
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
        ports:
        - containerPort: 80
        volumeMounts:
            - name: nginx-index-file
              mountPath: /usr/share/nginx/html/
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
          - name: HTTP_PORT
            valueFrom:
              configMapKeyRef:
                name: multitool-maps
                key: HTTP_PORT
      volumes:
      - name: nginx-index-file
        configMap:
          name: multitool-maps
```

configmap.yaml

```YAML

apiVersion: v1
kind: ConfigMap
metadata:
  name: multitool-maps
  namespace: 12-08-hw
data:
  HTTP_PORT: '1180'
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi! This is a configmap Index file </h1>
    </html>
```

service.yaml

```YAML

apiVersion: v1
kind: Service
metadata:
  name: nginx-multitool-svc
  namespace: 12-08-hw
spec:
  selector:
    app: nmt
  ports:
    - protocol: TCP
      name: nginx
      port: 80
      targetPort: 80
    - protocol: TCP
      name: multitool
      port: 8080
      targetPort: 1180
```
------

### Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS 

1. Создать Deployment приложения, состоящего из Nginx.
2. Создать собственную веб-страницу и подключить её как ConfigMap к приложению.
3. Выпустить самоподписной сертификат SSL. Создать Secret для использования сертификата.

#### Ответ:

Создаю сертификат
![image](https://github.com/DrDavidN/12-08-hw/assets/128225763/8309501b-2b3b-433f-b720-7c45e6333141)

Создаю secret
![image](https://github.com/DrDavidN/12-08-hw/assets/128225763/32291aae-6bd7-4362-a437-49bc6f8f66bd)

4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS. 

Создаю ingress и service и применяю их, проверяю доступ
![image](https://github.com/DrDavidN/12-08-hw/assets/128225763/e162f5ef-fa2b-4f98-904d-1ccd0e53b6d0)
![image](https://github.com/DrDavidN/12-08-hw/assets/128225763/a4cd554e-cbf4-4035-9ecb-adfd44b8495c)

Сайт доступен.

5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

#### Ответ:

nginx_deployment.yaml

```YAML

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-only
  namespace: 12-08-hw
  labels:
    app: nginx-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-frontend
  template:
    metadata:
      labels:
        app: nginx-frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
        volumeMounts:
        - name: nginx-mount
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-mount
        configMap:
          name: nginx-maps
```

nginx_configmap.yaml

```YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-maps
  namespace: 12-08-hw
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <body>
    <h1>This is a modified test page!</h1>
    </body>
    </html>
```

nginx_ingress.yaml

```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  namespace: 12-08-hw
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myingress.com
    secretName: ingress-cert
  rules:
  - host: myingress.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

nginx_service.yaml

```YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: 12-08-hw
spec:
  type: ClusterIP
  selector:
    app: nginx-frontend
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 80
```

nginx_secret.yaml

```YAML
apiVersion: v1
kind: Secret
metadata:
  name: ingress-cert
  namespace: 12-08-hw
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUQzekNDQXNlZ0F3SUJBZ0lVZjFNbkZGeStCY09FNmhoREJFTEY2NDNSMzFZd0RRWUpLb1pJaHZjTkFRRUwKQlFBd2ZqRUxNQWtHQTFVRUJoTUNVbFV4RHpBTkJnTlZCQWdNQmsxdmMyTnZkekVQTUEwR0ExVUVCd3dHVFc5eg>
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2d0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktrd2dnU2xBZ0VBQW9JQkFRRG9JZ0ZnTkgxbXJTTWUKZkd3Vm4vOUxhMUEyL093VDFBYWo5dmFYeFZOZ2VYK01uNlN4YjVIUFdEQlVDVWFOUUJ3aldjTkRvK1RSd2JNZg>
```
------
