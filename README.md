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

```YAAML
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
4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

------
