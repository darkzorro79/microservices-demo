# Helm chart for Online Boutique

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
