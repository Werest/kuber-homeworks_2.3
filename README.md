# kuber-homeworks_2.3

# Домашнее задание к занятию «Настройка приложений и управление доступом в Kubernetes»

### Примерное время выполнения задания

120 минут

### Цель задания

Научиться:
- Настраивать конфигурацию приложений с помощью **ConfigMaps** и **Secrets**
- Управлять доступом пользователей через **RBAC**

Это задание поможет вам освоить ключевые механизмы Kubernetes для работы с конфигурацией и безопасностью. Эти навыки необходимы для уверенного администрирования кластеров в реальных проектах. На практике навыки используются для:
- Хранения чувствительных данных (Secrets)
- Гибкого управления настройками приложений (ConfigMaps) 
- Контроля доступа пользователей и сервисов (RBAC)

------

## **Подготовка**
### **Чеклист готовности**
- Установлен Kubernetes (MicroK8S, Minikube или другой)
- Установлен `kubectl`
- Редактор для YAML-файлов (VS Code, Vim и др.)
- Утилита `openssl` для генерации сертификатов

------

### Инструменты, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S
2. [Инструкция](https://minikube.sigs.k8s.io/docs/start/) по установке Minikube
3. [Инструкция](https://kubernetes.io/docs/tasks/tools/) по установке kubectl
4. [Инструкция](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) по установке VS Code

### Дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/configuration/secret/) Secret.
2. [Описание](https://kubernetes.io/docs/concepts/configuration/configmap/) ConfigMap.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.
4. [Описание](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) RBAC.
5. [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/).
6. [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b).

------

## **Задание 1: Работа с ConfigMaps**
### **Задача**
Развернуть приложение (nginx + multitool), решить проблему конфигурации через ConfigMap и подключить веб-страницу.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами
   - `nginx`
   - `multitool`
3. **Подключить веб-страницу** через ConfigMap
4. **Проверить доступность**

### **Что сдать на проверку**
- Манифесты:
  - `deployment.yaml`
  - `configmap-web.yaml`
- Скриншот вывода `curl` или браузера
---
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>Nginx + Multitool App</title>
    </head>
    <body>
        <div class="container">
            <div class="header">
                <h1>Приложение развернуто успешно!</h1>
            </div>
            <div class="info">
                <h2>Компоненты приложения:</h2>
                <ul>
                    <li><strong>Nginx</strong> - веб-сервер</li>
                    <li><strong>Multitool</strong> - утилита для отладки</li>
                    <li><strong>ConfigMap</strong> - для конфигурации</li>
                </ul>
                <p>Время развертывания: <span id="datetime"></span></p>
            </div>
            <p>ConfigMap успешно подключен и работает!</p>
        </div>
        <script>
            document.getElementById('datetime').textContent = new Date().toLocaleString();
        </script>
    </body>
    </html>
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-multitool-app
  labels:
    app: nginx-multitool
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-multitool
  template:
    metadata:
      labels:
        app: nginx-multitool
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config-volume
          mountPath: /usr/share/nginx/html
          readOnly: true
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
        - name: HTTP_PORT
          value: "8080"
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo 'Multitool is running'; sleep 60; done"]
      volumes:
      - name: nginx-config-volume
        configMap:
          name: nginx-config
          items:
          - key: index.html
            path: index.html
```

<img width="1157" height="762" alt="image" src="https://github.com/user-attachments/assets/5b90de25-bba7-4e11-9717-3926aa282dfc" />

---
## **Задание 2: Настройка HTTPS с Secrets**  
### **Задача**  
Развернуть приложение с доступом по HTTPS, используя самоподписанный сертификат.

### **Шаги выполнения**  
1. **Сгенерировать SSL-сертификат**
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
```
2. **Создать Secret**
3. **Настроить Ingress**
4. **Проверить HTTPS-доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `secret-tls.yaml`
  - `ingress-tls.yaml`
- Скриншот вывода `curl -k`
---
**secret-tls.yaml**
```
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURHVENDQWdHZ0F3SUJBZ0lVQS9hLzlqdGZKeGg5dzFDZVZPSk1uSkRwc0trd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0hERWFNQmdHQTFVRUF3d1JiWGxoY0hBdVpYaGhiWEJzWlM1amIyMHdIaGNOTWpVeE1URXdNVEF3TWpNNApXaGNOTWpZeE1URXdNVEF3TWpNNFdqQWNNUm93R0FZRFZRUUREQkZ0ZVdGd2NDNWxlR0Z0Y0d4bExtTnZiVENDCkFTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTjZ4b2cvSm10N1lHeWh0Y2x4QVcwd1QKTzF3Wk8xUXlGU3FSZk9vdzEzb2RYa2xBeFlhRi9vRzVERVhLOVRweFMxeHVVdmRrR1hPamY0U3preXMvR2lqdwpzblgxd3BZWnFhYWQyWVNUb3NSVGExbGZnaWxaWEhFT09JT0ZkS29HQnVQSzZ1eUNiZ2ZvQ0xWUFFjYUFIUmZNCmxEWXVkdkgyQkFkMDE3TWtUa1hXMFp0ajlmSlhBeUFtMVBuNFVFTlFZODJ1aHpDdWs2TEdVbWlMczY4MjN6NW8KYU51RWhYMzcvU1VpSUwvaVhuQmhwN3FEaWtNcXNOUThka05zNFhFUEJuUEt1YktmdW5lSHlHS2k2d0FBeWRhOQp3Y3ZtbjhqcEhNMkZLTUFZSzVEcHRMeGRmR0JHSWJsTDl1WGRSRnduRmo0N1pkR1hnYmNZUVZ5OU1uRmVUaWtDCkF3RUFBYU5UTUZFd0hRWURWUjBPQkJZRUZLZTBZaU53MlRBbUhnMG91NTZ1WGsrRWRiY2RNQjhHQTFVZEl3UVkKTUJhQUZLZTBZaU53MlRBbUhnMG91NTZ1WGsrRWRiY2RNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdEUVlKS29aSQpodmNOQVFFTEJRQURnZ0VCQUpKQTZXRnZsVTRmREdRSnNCSHE0eExOclkxbVlma2k4KzBtbUs3ZDZKZUV5K0VPClM4aGYveXVaRUlIZFRvelVSRFpudjYyWmdJajFRMXF6MmYwek9MUHN4MS9GVDNQbzlLWjdpc3VJNVB3S0lZWVYKQlMvYmcrMEo4YW5NUFFQQU1aUnIxUWd0V29hcXlGaGpYRDNjN0hoQXVXcnpoeVl3dVhLRlJxanNmL3lrYjhNNQptMW9oUk1kaFlMU2lmak02ZWRJeHp3RTlVWlAvdEprMVd6U0pDQndsZG9IbHh6WTRYQkVYQkRrVjdSWVpXR29wCk90QlFvRERtTkxmNTBZMWlQMXFna0Uxc3dZakJmTkJ1N1FyVlF5MHk1VHd0bEhRb3FTbWJjSWxVZHg3ZDFYWUoKN1lEeUpTcnBVZlA3UFRhdUhBTFF2RDl4Tm85UGF0UThsYUEvSzlNPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2Z0lCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktnd2dnU2tBZ0VBQW9JQkFRRGVzYUlQeVpyZTJCc28KYlhKY1FGdE1FenRjR1R0VU1oVXFrWHpxTU5kNkhWNUpRTVdHaGY2QnVReEZ5dlU2Y1V0Y2JsTDNaQmx6bzMrRQpzNU1yUHhvbzhMSjE5Y0tXR2FtbW5kbUVrNkxFVTJ0Wlg0SXBXVnh4RGppRGhYU3FCZ2JqeXVyc2dtNEg2QWkxClQwSEdnQjBYekpRMkxuYng5Z1FIZE5lekpFNUYxdEdiWS9YeVZ3TWdKdFQ1K0ZCRFVHUE5yb2N3cnBPaXhsSm8KaTdPdk50OCthR2piaElWOSsvMGxJaUMvNGw1d1lhZTZnNHBES3JEVVBIWkRiT0Z4RHdaenlybXluN3AzaDhoaQpvdXNBQU1uV3ZjSEw1cC9JNlJ6TmhTakFHQ3VRNmJTOFhYeGdSaUc1Uy9ibDNVUmNKeFkrTzJYUmw0RzNHRUZjCnZUSnhYazRwQWdNQkFBRUNnZ0VBQ1l1ZFpkWVhSM05GcWJ4VVBUQk1nWVNuQkJYNFNTMzY0b2ZaTGZ3aE5sU2wKTHBoaVJ4ZDkvdDg0ejVLSkk2UEl6a25UMEZNWDlMRVd4WkJCY0FaOHgzRzJ3by9oMHZkL1lqc01vRzVJR1NoMgp0dDVkU01xeDU4NHJYYmJXRnZZYWJEL3R4U3k3ak9iMUhabHJ1VGVUVjZwdkZXbjFNdjZmc2xhZnFzZjArM0VLCjhqeEQxYno0VHdXZnlGUGNnS2MrbkgvREhFNUVKY0kybTlUZW1neWRlMDQraHQxdzR2d2pkNk5GNW0vZ1I5RFgKNktFT1NQS0FHSW5JZkVhcG9MejNxaEsxb1dxL25mTlM4eWQxTTMvTUNtYW1qanNYaWZqdDVYbEJrVzdBYWtRbQpUT2t0UUtKTXFzNnVid3d2M3FIZkFMUFc3MEV0a0pVcWMxRXVkNjhIRVFLQmdRRDBmb3RGSFJCL3Bya2V2VVJ2CndiZXpxVXdFV0M1L2RGcmVBR0FDcmYrQzVaV2svRmdVVU1Fcmd0bzVwZVR0dWkwVk9iR0ZRV2xFNjFqSjVxZnUKSHBFZUNQTGRIYzh6WlU4bjlhRlBNd21FTUhveW4reUN6NVA2R1FqeXJ3c054SXFXclYzcW5Mcmt3L0J4ZkdEbwo2QnUrRVJFcmNsNkV5ZWNpY05qY2Q2a2Z1UUtCZ1FEcExIVE4yN1RrZGp2UzRTNlo0NnBwYkxLbXZzLzdrdE5FCkF3L3A3SCtMcHNiMHdRQW02VVFlWUMxYmNpNGhXbEp4WnZRaUpWbkJZenV1Yy9pN3ZiTU8wSEdTRjYzY1ArcnMKWFBtOUtLSVBkeEFuVTZhL3J2dkZMYjZoaEMyaFRMQzRQSDl5UCtOaFpyMHVTc3d2UlZ6dDE0blYyR0lYNlhJago0TmR5N3ZGNThRS0JnUUR3MUxHRXU3Tmp1aXVCTHZmNWlYelZPUDhMREgySXlHazFZbk13VUlwdU9vMmovWGtaClVjNWxlTGhTcnFtSXphbll6WUtpZXlNaFl4MkxpMnVCTDVUTVhBNDJaa1dTSDhuUDE3RTBYVmFUWWxrTUF1R3EKbktQclo3dWJxWGFlOGlsOWNIb2hQbzhPQmorUzJleHZueXFzbHcwdG1iT200MmFNVTFJYUlMaHo2UUtCZ0NQVgp5VDI1L2xyK01NT0FDQUZ1azhvUUFvVE4zbmp6WDdTY0k5MzNrc0tMcHhnR2NWZUprZ1o1RVg5MFZOS2Jad0EyCnFsNTFEdzVCaWZLTnNEUnFPeEtUeG1DRmN4cmVWK1JyaFpZa29JTnY4UE9hVDQ0MS9rdVhkQ3l1ZTdUR2JJSmsKQ2RJdEwyelF0VkpmL0hGblg1ak4vMGoyTUYwc2EzWklIOVJ5RkhxQkFvR0JBTGwxMm1jS0xYTUFMOHYzdE1NQwo3anFLMmhuNVZiallxOGhvaUM4UTNibm5BM0RSRVZOd2VuNUFHYzhUSEpYMmpIU2VoYUJyYUZNYzVnVXpBL0oxCmV6UnBZNUF3QlZINmFHY0ppcnMvNWwvbGdIa0llNEovOCtrczdEUklCVzg2dnVQTDR5SEd6cmY0V0pXTWtEM1UKVVBVWTVzVTQyaWVnWTVaMVhvYWFuYmVyCi0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K

```
**ingress-tls.yaml**
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: https-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80

```
Пришлось сделать так, иначе получал ошибку - **Could not resolve host: myapp.example.com**
```
echo "127.0.0.1 myapp.example.com" | sudo tee -a /etc/hosts
```
<img width="1000" height="965" alt="image" src="https://github.com/user-attachments/assets/a0fb1540-241a-4c11-9e02-6a6f12ba55d5" />



---
## **Задание 3: Настройка RBAC**  
### **Задача**  
Создать пользователя с ограниченными правами (только просмотр логов и описания подов).

### **Шаги выполнения**  
1. **Включите RBAC в microk8s**
```bash
microk8s enable rbac
```
2. **Создать SSL-сертификат для пользователя**
```bash
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN={ИМЯ ПОЛЬЗОВАТЕЛЯ}"
openssl x509 -req -in developer.csr -CA {CA серт вашего кластера} -CAkey {CA ключ вашего кластера} -CAcreateserial -out developer.crt -days 365
```
3. **Создать Role (только просмотр логов и описания подов) и RoleBinding**
4. **Проверить доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `role-pod-reader.yaml`
  - `rolebinding-developer.yaml`
- Команды генерации сертификатов
- Скриншот проверки прав (`kubectl get pods --as=developer`)

---
## Шаблоны манифестов с учебными комментариями
### **1. Deployment с ConfigMap (nginx + multitool)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
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
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config # ПОДКЛЮЧЕНИЕ ConfigMap
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config # УКАЖИТЕ имя созданного ConfigMap
```
### **2. ConfigMap для веб-страницы**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-content # ИЗМЕНИТЕ: Укажите имя ConfigMap
  namespace: default # ОПЦИОНАЛЬНО: Укажите namespace, если не default
data:
  # КЛЮЧЕВОЙ МОМЕНТ: index.html будет подключен как файл
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Страница из ConfigMap</title> # ИЗМЕНИТЕ: Заголовок страницы
    </head>
    <body>
      <h1>Привет от Kubernetes!</h1> # ДОБАВЬТЕ: Свой контент страницы
    </body>
    </html>
```

### **3. Secret для TLS-сертификата**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret # ИЗМЕНИТЕ при необходимости
type: kubernetes.io/tls
data:
  tls.crt: # ЗАМЕНИТЕ на base64-код сертификата (cat tls.crt | base64 -w 0)
  tls.key: # ЗАМЕНИТЕ на base64-код ключа (cat tls.key | base64 -w 0)
```
### **4. Role для просмотра подов**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-viewer # ИЗМЕНИТЕ: Название роли
  namespace: default # ВАЖНО: Role работает только в указанном namespace
rules:
- apiGroups: [""] # КЛЮЧЕВОЙ МОМЕНТ: "" означает core API group
  resources: # РАЗРЕШЕННЫЕ РЕСУРСЫ:
    - pods # Доступ к просмотру подов
    - pods/log # Доступ к логам подов
  verbs: # РАЗРЕШЕННЫЕ ДЕЙСТВИЯ:
    - get # Просмотр отдельных подов
    - list # Список всех подов
    - watch # Мониторинг изменений
    - describe # Просмотр деталей
# ДОПОЛНИТЕЛЬНО: Можно добавить больше правил для других ресурсов
```
---

## **Правила приёма работы**
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать:
   - Скриншоты вывода команд `kubectl`
   - Скриншоты результатов выполнения
   - Тексты манифестов или ссылки на них
3. Для заданий с TLS приложите команды генерации сертификатов

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).
