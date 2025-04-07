<!-- <p align="center">
<img src="/src/frontend/static/icons/Hipster_HeroLogoMaroon.svg" width="300" alt="Online Boutique" />
</p> -->
![Continuous Integration](https://github.com/GoogleCloudPlatform/microservices-demo/workflows/Continuous%20Integration%20-%20Main/Release/badge.svg)

**Online Boutique** is a cloud-first microservices demo application.  The application is a
web-based e-commerce app where users can browse items, add them to the cart, and purchase them.

Google uses this application to demonstrate how developers can modernize enterprise applications using Google Cloud products, including: [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine), [Cloud Service Mesh (CSM)](https://cloud.google.com/service-mesh), [gRPC](https://grpc.io/), [Cloud Operations](https://cloud.google.com/products/operations), [Spanner](https://cloud.google.com/spanner), [Memorystore](https://cloud.google.com/memorystore), [AlloyDB](https://cloud.google.com/alloydb), and [Gemini](https://ai.google.dev/). This application works on any Kubernetes cluster.

If you’re using this demo, please **★Star** this repository to show your interest!

**Note to Googlers:** Please fill out the form at [go/microservices-demo](http://go/microservices-demo).

## Architecture

**Online Boutique** is composed of 11 microservices written in different
languages that talk to each other over gRPC.

[![Architecture of
microservices](/docs/img/architecture-diagram.png)](/docs/img/architecture-diagram.png)

Find **Protocol Buffers Descriptions** at the [`./protos` directory](/protos).

| Service                                              | Language      | Description                                                                                                                       |
| ---------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](/src/frontend)                           | Go            | Exposes an HTTP server to serve the website. Does not require signup/login and generates session IDs for all users automatically. |
| [cartservice](/src/cartservice)                     | C#            | Stores the items in the user's shopping cart in Redis and retrieves it.                                                           |
| [productcatalogservice](/src/productcatalogservice) | Go            | Provides the list of products from a JSON file and ability to search products and get individual products.                        |
| [currencyservice](/src/currencyservice)             | Node.js       | Converts one money amount to another currency. Uses real values fetched from European Central Bank. It's the highest QPS service. |
| [paymentservice](/src/paymentservice)               | Node.js       | Charges the given credit card info (mock) with the given amount and returns a transaction ID.                                     |
| [shippingservice](/src/shippingservice)             | Go            | Gives shipping cost estimates based on the shopping cart. Ships items to the given address (mock)                                 |
| [emailservice](/src/emailservice)                   | Python        | Sends users an order confirmation email (mock).                                                                                   |
| [checkoutservice](/src/checkoutservice)             | Go            | Retrieves user cart, prepares order and orchestrates the payment, shipping and the email notification.                            |
| [recommendationservice](/src/recommendationservice) | Python        | Recommends other products based on what's given in the cart.                                                                      |
| [adservice](/src/adservice)                         | Java          | Provides text ads based on given context words.                                                                                   |
| [loadgenerator](/src/loadgenerator)                 | Python/Locust | Continuously sends requests imitating realistic user shopping flows to the frontend.                                              |

## Screenshots

| Home Page                                                                                                         | Checkout Screen                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Screenshot of store homepage](/docs/img/online-boutique-frontend-1.png)](/docs/img/online-boutique-frontend-1.png) | [![Screenshot of checkout screen](/docs/img/online-boutique-frontend-2.png)](/docs/img/online-boutique-frontend-2.png) |

## Quickstart (GKE)

1. Ensure you have the following requirements:
   - [Google Cloud project](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project).
   - Shell environment with `gcloud`, `git`, and `kubectl`.

2. Clone the latest major version.

   ```sh
   git clone --depth 1 --branch v0 https://github.com/GoogleCloudPlatform/microservices-demo.git
   cd microservices-demo/
   ```

   The `--depth 1` argument skips downloading git history.

3. Set the Google Cloud project and region and ensure the Google Kubernetes Engine API is enabled.

   ```sh
   export PROJECT_ID=<PROJECT_ID>
   export REGION=us-central1
   gcloud services enable container.googleapis.com \
     --project=${PROJECT_ID}
   ```

   Substitute `<PROJECT_ID>` with the ID of your Google Cloud project.

4. Create a GKE cluster and get the credentials for it.

   ```sh
   gcloud container clusters create-auto online-boutique \
     --project=${PROJECT_ID} --region=${REGION}
   ```

   Creating the cluster may take a few minutes.

5. Deploy Online Boutique to the cluster.

   ```sh
   kubectl apply -f ./release/kubernetes-manifests.yaml
   ```

6. Wait for the pods to be ready.

   ```sh
   kubectl get pods
   ```

   After a few minutes, you should see the Pods in a `Running` state:

   ```
   NAME                                     READY   STATUS    RESTARTS   AGE
   adservice-76bdd69666-ckc5j               1/1     Running   0          2m58s
   cartservice-66d497c6b7-dp5jr             1/1     Running   0          2m59s
   checkoutservice-666c784bd6-4jd22         1/1     Running   0          3m1s
   currencyservice-5d5d496984-4jmd7         1/1     Running   0          2m59s
   emailservice-667457d9d6-75jcq            1/1     Running   0          3m2s
   frontend-6b8d69b9fb-wjqdg                1/1     Running   0          3m1s
   loadgenerator-665b5cd444-gwqdq           1/1     Running   0          3m
   paymentservice-68596d6dd6-bf6bv          1/1     Running   0          3m
   productcatalogservice-557d474574-888kr   1/1     Running   0          3m
   recommendationservice-69c56b74d4-7z8r5   1/1     Running   0          3m1s
   redis-cart-5f59546cdd-5jnqf              1/1     Running   0          2m58s
   shippingservice-6ccc89f8fd-v686r         1/1     Running   0          2m58s
   ```

7. Access the web frontend in a browser using the frontend's external IP.

   ```sh
   kubectl get service frontend-external | awk '{print $4}'
   ```

   Visit `http://EXTERNAL_IP` in a web browser to access your instance of Online Boutique.

8. Congrats! You've deployed the default Online Boutique. To deploy a different variation of Online Boutique (e.g., with Google Cloud Operations tracing, Istio, etc.), see [Deploy Online Boutique variations with Kustomize](#deploy-online-boutique-variations-with-kustomize).

9. Once you are done with it, delete the GKE cluster.

   ```sh
   gcloud container clusters delete online-boutique \
     --project=${PROJECT_ID} --region=${REGION}
   ```

   Deleting the cluster may take a few minutes.

## Additional deployment options

- **Terraform**: [See these instructions](/terraform) to learn how to deploy Online Boutique using [Terraform](https://www.terraform.io/intro).
- **Istio / Cloud Service Mesh**: [See these instructions](/kustomize/components/service-mesh-istio/README.md) to deploy Online Boutique alongside an Istio-backed service mesh.
- **Non-GKE clusters (Minikube, Kind, etc)**: See the [Development guide](/docs/development-guide.md) to learn how you can deploy Online Boutique on non-GKE clusters.
- **AI assistant using Gemini**: [See these instructions](/kustomize/components/shopping-assistant/README.md) to deploy a Gemini-powered AI assistant that suggests products to purchase based on an image.
- **And more**: The [`/kustomize` directory](/kustomize) contains instructions for customizing the deployment of Online Boutique with other variations.

## Documentation

- [Development](/docs/development-guide.md) to learn how to run and develop this app locally.


# CI/CD для Hipster Shop с GitHub Actions и ArgoCD

Данный документ описывает настройку и использование CI/CD пайплайна для проекта Hipster Shop, который автоматизирует процессы тестирования, сборки и развертывания приложения в Kubernetes с использованием GitHub Actions и ArgoCD.

## Архитектура решения

- **Репозиторий**: GitHub (https://github.com/darkzorro79/microservices-demo)
- **CI система**: GitHub Actions
- **CD система**: ArgoCD
- **Контейнерный реестр**: GitHub Container Registry (ghcr.io)
- **Оркестрация**: Kubernetes с Ingress Controller и cert-manager
- **Упаковка**: Helm Chart

## Этапы CI/CD пайплайна

### 1. Этап тестирования
- Автоматически запускается при коммите в ветки main/master
- Выполняет unit-тесты для микросервисов
- Тесты запускаются в изолированной среде в GitHub Actions

### 2. Этап сборки и публикации
- Собирает Docker-образы для каждого микросервиса
- Публикует образы в GitHub Container Registry (ghcr.io)
- Теги образов: короткий хеш коммита и latest
- Обновляет Helm values с новыми тегами образов

### 3. Этап развертывания
- ArgoCD отслеживает изменения в Helm chart
- Автоматически синхронизирует состояние кластера с желаемым состоянием
- Обеспечивает безопасный доступ через Ingress с TLS-сертификатом

## Инструкция по настройке CI/CD

### Предварительные требования
- Кластер Kubernetes с установленным Ingress-контроллером
- Установленный cert-manager
- Доступ к GitHub с правами на создание репозиториев и секретов
- Настроенный ArgoCD в кластере

### Шаг 1: Настройка GitHub и репозитория

1. Форкните репозиторий:
   ```bash
   git clone https://github.com/darkzorro79/microservices-demo.git
   cd microservices-demo
   ```

2. Создайте Personal Access Token (PAT) в GitHub:
   - Перейдите в Settings → [Personal access tokens](https://github.com/settings/tokens)
   - Создайте новый токен с разрешениями `repo` и `write:packages`

3. Добавьте токен в секреты репозитория:
   - Перейдите в настройки репозитория → Secrets and variables → Actions
   - Создайте секрет с именем `PAT` и значением вашего токена

4. Создайте workflow файл:
   ```bash
   mkdir -p .github/workflows
   nano .github/workflows/ci-cd.yml
   ```

5. Добавьте содержимое CI/CD workflow (см. файл .github/workflows/ci-cd.yml)

### Шаг 2: Настройка Kubernetes и ArgoCD

1. Создайте namespace для приложения:
   ```bash
   kubectl create namespace hipster-shop
   ```

2. Создайте Secret для доступа к GitHub Container Registry:
   ```bash
   kubectl create secret docker-registry ghcr-secret \
     --namespace hipster-shop \
     --docker-server=ghcr.io \
     --docker-username=YOUR_GITHUB_USERNAME \
     --docker-password=YOUR_GITHUB_TOKEN
   ```

3. Создайте ArgoCD Application:
   ```bash
   nano argocd-app.yaml
   ```

4. Добавьте в файл содержимое конфигурации ArgoCD Application:
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: hipster-shop
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: https://github.com/darkzorro79/microservices-demo.git
       targetRevision: main
       path: helm-chart
       helm:
         values: |
           frontend:
             host: hipster-shop.otservice.ru
             externalService: false
             ingress:
               enabled: true
               annotations:
                 kubernetes.io/ingress.class: nginx
                 cert-manager.io/cluster-issuer: letsencrypt-production
               tls:
                 - secretName: hipster-shop-tls
                   hosts:
                     - hipster-shop.otservice.ru
               hosts:
                 - host: hipster-shop.otservice.ru
                   paths:
                     - path: /
                       pathType: Prefix
           global:
             imagePullSecrets:
               - name: ghcr-secret
     destination:
       server: https://kubernetes.default.svc
       namespace: hipster-shop
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
       - CreateNamespace=true
   ```

5. Примените файл конфигурации ArgoCD:
   ```bash
   kubectl apply -f argocd-app.yaml
   ```

### Шаг 3: Проверка работы CI/CD пайплайна

1. Внесите изменения в код и отправьте их в репозиторий:
   ```bash
   git add .
   git commit -m "Test CI/CD pipeline"
   git push origin main
   ```

2. Проверьте статус GitHub Actions:
   ```
   https://github.com/YOUR_USERNAME/microservices-demo/actions
   ```

3. Проверьте статус синхронизации ArgoCD:
   ```bash
   kubectl get applications -n argocd
   ```

4. Проверьте доступность приложения:
   ```bash
   kubectl get ingress -n hipster-shop
   ```

5. Откройте приложение в браузере:
   ```
   https://hipster-shop.otservice.ru
   ```

## Преимущества реализованного CI/CD

1. **Полная автоматизация**: весь процесс от коммита кода до развертывания в Kubernetes происходит автоматически
2. **GitOps подход**: все конфигурации хранятся в Git, что обеспечивает версионирование и аудит изменений
3. **Быстрое восстановление**: ArgoCD автоматически синхронизирует состояние кластера с желаемым состоянием
4. **Безопасность**: автоматическое получение TLS-сертификатов через cert-manager
5. **Масштабируемость**: легко адаптируется под добавление новых микросервисов

## Дополнительные рекомендации

1. **Мониторинг**: настройте Prometheus и Grafana для отслеживания работы приложения
2. **Логирование**: настройте сбор логов через EFK/ELK стек
3. **Уведомления**: добавьте интеграцию с Slack/Telegram для уведомлений о статусе CI/CD
4. **Среды**: расширьте пайплайн для поддержки dev/staging/prod окружений

---

Данный CI/CD пайплайн обеспечивает надежный и автоматизированный процесс доставки приложения, следуя принципам GitOps и DevOps.
