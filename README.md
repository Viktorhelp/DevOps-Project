
# Полное руководство по настройке DevOps Pipeline

Этот документ описывает полный процесс настройки DevOps Pipeline с использованием AWS, Terraform, KOPS, Jenkins, Helm и Docker. Здесь собраны все команды для инициализации и установки.

## Требования

Перед началом убедитесь, что у вас установлены следующие инструменты:
- **AWS CLI** (настроен с учетными данными)
- **Terraform**
- **KOPS**
- **kubectl**
- **Jenkins** (установлен и настроен)
- **Docker**

## Шаг 1: Установка и настройка необходимых инструментов

1. **Установите AWS CLI**:
   ```bash
   curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
   sudo installer -pkg AWSCLIV2.pkg -target /
   aws configure
   ```

2. **Установите Terraform**:
   ```bash
   sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
   curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
   echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
   sudo apt-get update && sudo apt-get install terraform
   ```

3. **Установите KOPS**:
   ```bash
   curl -LO https://github.com/kubernetes/kops/releases/download/v1.25.0/kops-linux-amd64
   chmod +x kops-linux-amd64
   sudo mv kops-linux-amd64 /usr/local/bin/kops
   ```

4. **Установите kubectl**:
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/kubectl
   ```

5. **Установите Docker**:
   ```bash
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   sudo apt-get update
   sudo apt-get install -y docker-ce
   sudo usermod -aG docker ${USER}
   ```

6. **Установите Jenkins**:
   ```bash
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   sudo apt-get update
   sudo apt-get install jenkins
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

## Шаг 2: Настройка инфраструктуры с помощью Terraform

1. **Клонируйте репозиторий и перейдите в директорию Terraform**:
   ```bash
   git clone <your-repository-url>
   cd <repository-directory>/terraform
   ```

2. **Инициализируйте Terraform**:
   ```bash
   terraform init
   ```

3. **Примените план Terraform для создания VPC**:
   ```bash
   terraform apply
   ```
   Подтвердите выполнение, введя `yes` при запросе.

## Шаг 3: Настройка кластера Kubernetes с помощью KOPS

1. **Создайте S3 bucket для хранения состояния KOPS**:
   ```bash
   aws s3api create-bucket --bucket <your-kops-state-store> --region <your-region>
   ```

2. **Экспортируйте переменную окружения для KOPS**:
   ```bash
   export KOPS_STATE_STORE=s3://<your-kops-state-store>
   ```

3. **Создайте кластер Kubernetes с помощью KOPS**:
   ```bash
   kops create -f ../kops/cluster.yaml
   kops create secret --name <cluster-name> sshpublickey admin -i ~/.ssh/id_rsa.pub
   kops update cluster <cluster-name> --yes
   ```

4. **Проверьте состояние кластера**:
   ```bash
   kops validate cluster
   ```

## Шаг 4: Настройка и запуск Jenkins

1. **Установите необходимые плагины в Jenkins**:
   - Docker Pipeline
   - Kubernetes CLI
   - AWS Pipeline

2. **Настройте учетные данные в Jenkins**:
   - AWS credentials
   - Docker Hub credentials
   - GitHub credentials

3. **Создайте новую задачу (Pipeline) в Jenkins и используйте `Jenkinsfile`**:
   - Перейдите в директорию `jenkins/` в репозитории.
   - Настройте `Jenkinsfile` для вашего окружения и запускайте пайплайн.

## Шаг 5: Сборка Docker-образа и публикация в AWS ECS

1. **Запустите пайплайн в Jenkins, который:**
   - Скачает код из репозитория [ngx-api-utils](https://github.com/ngx-api-utils/ngx-api-utils).
   - Соберет Docker-образ с использованием мультистейдж сборки.
   - Опубликует Docker-образ в AWS ECS.

2. **Развертывание приложения в Kubernetes через Helm**:
   - Пайплайн развернет приложение в кластере Kubernetes с использованием Helm.

## Шаг 6: Настройка AWS Load Balancer

1. **Helm chart автоматически настраивает AWS Load Balancer**, который будет доступен по DNS имени.

## Очистка

1. **Удалите ресурсы Terraform**:
   ```bash
   terraform destroy
   ```
   Подтвердите выполнение, введя `yes`.

2. **Удалите кластер KOPS**:
   ```bash
   kops delete cluster --name <cluster-name> --yes
   ```

## Заключение

Это руководство охватывает полный процесс настройки DevOps Pipeline с нуля до развертывания приложения на AWS с использованием всех необходимых инструментов и технологий.
