# Методические указания для студентов по выполнению лабораторной работы (КИМ-1.1)
## Тема: «Проектирование архитектуры ИИ-приложений»

## Оглавление

1. [Общие положения](#1-общие-положения)
2. [Лабораторная работа №1. Основы ML-инфраструктуры](#2-лабораторная-работа-1-основы-ml-инфраструктуры)
3. [Лабораторная работа №2. Пайплайн данных, бессерверный инференс и мониторинг](#3-лабораторная-работа-2-пайплайн-данных-бессерверный-инференс-и-мониторинг)
4. [Приложение А. Примеры кода](#приложение-а-примеры-кода)
5. [Приложение Б. Контрольные вопросы и критерии оценивания](#приложение-б-контрольные-вопросы-и-критерии-оценивания)


## 1. Общие положения

### 1.1. Назначение лабораторных работ

Лабораторные работы по модулю «Проектирование архитектуры ИИ-приложений» направлены на формирование у студентов практических навыков создания, развертывания и эксплуатации промышленных ML-решений в облачной среде.

**В рамках двух лабораторных работ студенты должны:**

1. Разработать контейнеризированный ML-сервис на основе FastAPI и PyTorch.
2. Развернуть сервис в облачной инфраструктуре (Yandex Cloud) с использованием Kubernetes.
3. Настроить CI/CD пайплайн для автоматического развертывания.
4. Построить пайплайн обработки данных и обучить модель в DataSphere.
5. Организовать бессерверный инференс через Cloud Functions и API Gateway.
6. Настроить мониторинг качества модели с помощью DataLens.

### 1.2. Требования к подготовке

Перед выполнением лабораторных работ студент должен:

1. Зарегистрироваться в Yandex Cloud и активировать пробный период (грант).
2. Установить и настроить Yandex Cloud CLI.
3. Иметь базовые навыки работы с Python, Docker, Git и командной строкой.
4. Ознакомиться с теоретическим материалом по контейнеризации, Kubernetes и бессерверным вычислениям.

### 1.3. Используемые технологии и сервисы

| Компонент | Назначение |
|-----------|------------|
| **Yandex Cloud CLI** | Управление облачными ресурсами |
| **Docker** | Контейнеризация приложения |
| **Yandex Container Registry** | Хранение Docker-образов |
| **Managed Service for Kubernetes** | Оркестрация контейнеров |
| **Yandex DataSphere** | Среда для обучения моделей с GPU |
| **Yandex Object Storage (S3)** | Хранение данных и моделей |
| **Yandex Cloud Functions** | Бессерверный инференс |
| **API Gateway** | Публичный доступ к функциям |
| **Managed Service for ClickHouse** | Хранение логов и метрик |
| **Yandex DataLens** | Визуализация и мониторинг |
| **GitHub Actions** | CI/CD пайплайн |

### 1.4. Структура отчёта

Каждая лабораторная работа оформляется в виде отчёта, содержащего:

1. Титульный лист с указанием темы, студента, группы и даты.
2. Описание выполненных этапов с листингами кода и скриншотами.
3. Результаты работы (скриншоты работающего сервиса, дашбордов и т.п.).
4. Ответы на контрольные вопросы.
5. Список использованных источников.

Отчёт сдаётся в электронном виде (PDF) в установленный преподавателем срок.


## 2. Лабораторная работа №1. Основы ML-инфраструктуры: от контейнера до облака

### 2.1. Цель работы

Создать и развернуть в облаке контейнеризированный веб-сервис, который принимает изображение дорожного знака и возвращает его класс, используя предварительно обученную модель. Настроить базовую инфраструктуру и CI/CD.

### 2.2. Задания к работе

#### 2.2.1. Подготовка облачной среды

**Шаг 1. Активация пробного периода Yandex Cloud**

1. Перейдите на сайт Yandex Cloud и зарегистрируйтесь.
2. Активируйте пробный период через консоль управления.
3. Привяжите платёжную карту для идентификации (списаний не будет, используется грант).

**Шаг 2. Установка и настройка Yandex Cloud CLI**

Установите Yandex Cloud CLI:

```bash
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```

Выполните инициализацию:

```bash
yc init
```

Выберите каталог (folder) и зону доступности (например, `ru-central1-a`).

**Шаг 3. Создание сервисного аккаунта**

Создайте сервисный аккаунт для управления ресурсами:

```bash
yc iam service-account create --name ml-sa --description "Сервисный аккаунт для ML-лабораторной"
```

Назначьте необходимые роли:

```bash
yc resource-manager folder add-access-binding <folder-id> \
    --role editor \
    --service-account-name ml-sa

yc resource-manager folder add-access-binding <folder-id> \
    --role container-registry.images.pusher \
    --service-account-name ml-sa

yc resource-manager folder add-access-binding <folder-id> \
    --role container-registry.images.puller \
    --service-account-name ml-sa

yc resource-manager folder add-access-binding <folder-id> \
    --role k8s.cluster-agent \
    --service-account-name ml-sa
```

Создайте авторизованный ключ:

```bash
yc iam key create --service-account-name ml-sa --output key.json
```

Добавьте ключ в профиль CLI:

```bash
yc config set service-account-key key.json
```

**Шаг 4. Создание VPC и Security Groups**

Создайте сеть и подсеть:

```bash
yc vpc network create --name ml-vpc
yc vpc subnet create --name ml-subnet \
    --zone ru-central1-a \
    --network-name ml-vpc \
    --range 10.0.0.0/24
```

Создайте Security Group, разрешающую входящий трафик на порты 80, 443 и 8080:

```bash
yc vpc security-group create --name ml-sg \
    --network-name ml-vpc \
    --rule "direction=ingress,port=80,protocol=tcp,v4-cidrs=0.0.0.0/0" \
    --rule "direction=ingress,port=443,protocol=tcp,v4-cidrs=0.0.0.0/0" \
    --rule "direction=ingress,port=8080,protocol=tcp,v4-cidrs=0.0.0.0/0"
```

**Шаг 5. Создание Container Registry**

Создайте реестр для хранения Docker-образов:

```bash
yc container registry create --name cr-ml
```

Получите идентификатор реестра и сохраните его для дальнейшего использования:

```bash
yc container registry list
```

---

#### 2.2.2. Контейнеризация ML-сервиса

**Шаг 1. Подготовка структуры проекта**

Создайте следующую структуру каталогов:

```
traffic-sign-service/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── model.py
│   └── utils.py
├── models/
│   └── resnet18_gtsrb.pth
├── requirements.txt
├── Dockerfile
└── .dockerignore
```

**Шаг 2. Разработка кода сервиса**

**Файл `app/model.py`** — загрузка модели PyTorch:

```python
import torch
import torchvision.models as models
import torchvision.transforms as transforms

MODEL_PATH = "models/resnet18_gtsrb.pth"
NUM_CLASSES = 43

def load_model():
    model = models.resnet18(pretrained=False)
    model.fc = torch.nn.Linear(model.fc.in_features, NUM_CLASSES)
    model.load_state_dict(torch.load(MODEL_PATH, map_location=torch.device('cpu')))
    model.eval()
    return model

def get_transforms():
    return transforms.Compose([
        transforms.Resize((32, 32)),
        transforms.ToTensor(),
        transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
    ])
```

**Файл `app/main.py`** — FastAPI-приложение:

```python
from fastapi import FastAPI, UploadFile, File, HTTPException
from fastapi.responses import JSONResponse
import uvicorn
from PIL import Image
import io
import json
import logging
from datetime import datetime

from app.model import load_model, get_transforms

logging.basicConfig(
    level=logging.INFO,
    format='{"time": "%(asctime)s", "level": "%(levelname)s", "message": "%(message)s"}'
)
logger = logging.getLogger(__name__)

app = FastAPI(title="Traffic Sign Recognition API", version="1.0.0")

model = load_model()
transform = get_transforms()

@app.get('/health')
async def health_check():
    return {"status": "ok"}

@app.post('/predict')
async def predict(file: UploadFile = File(...)):
    start_time = datetime.now()
    
    if not file.content_type.startswith('image/'):
        raise HTTPException(status_code=400, detail="File must be an image")
    
    try:
        contents = await file.read()
        image = Image.open(io.BytesIO(contents)).convert('RGB')
        tensor = transform(image).unsqueeze(0)
        
        with torch.no_grad():
            outputs = model(tensor)
            probabilities = torch.nn.functional.softmax(outputs, dim=1)
            confidence, predicted = torch.max(probabilities, 1)
        
        latency_ms = (datetime.now() - start_time).total_seconds() * 1000
        
        logger.info(json.dumps({
            "endpoint": "/predict",
            "predicted_class": int(predicted.item()),
            "confidence": float(confidence.item()),
            "latency_ms": round(latency_ms, 2),
            "file_size": len(contents)
        }))
        
        return JSONResponse({
            "predicted_class": int(predicted.item()),
            "confidence": float(confidence.item()),
            "latency_ms": round(latency_ms, 2)
        })
    
    except Exception as e:
        logger.error(json.dumps({"error": str(e)}))
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == '__main__':
    uvicorn.run(app, host="0.0.0.0", port=8080)
```

**Файл `requirements.txt`:**

```
fastapi==0.104.1
uvicorn[standard]==0.24.0
pillow==10.1.0
torch==2.1.0
torchvision==0.16.0
python-multipart==0.0.6
```

**Шаг 3. Создание Dockerfile**

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/
COPY models/ ./models/

EXPOSE 8080

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
```

**Шаг 4. Сборка и тестирование образа локально**

Сборка образа:

```bash
docker build -t cr.yandex/<registry-id>/traffic-sign:v1 .
```

Локальный запуск:

```bash
docker run -p 8080:8080 cr.yandex/<registry-id>/traffic-sign:v1
```

Тестирование эндпоинта:

```bash
curl -X POST -F "file=@test_sign.jpg" http://localhost:8080/predict
```

Ожидаемый ответ:

```json
{
    "predicted_class": 12,
    "confidence": 0.9723,
    "latency_ms": 45.67
}
```

---

#### 2.2.3. Публикация образа в Container Registry

**Шаг 1. Аутентификация в реестре**

Настройте Docker Credential Helper:

```bash
yc container registry configure-docker
```

**Шаг 2. Загрузка образа**

```bash
docker push cr.yandex/<registry-id>/traffic-sign:v1
```

---

#### 2.2.4. Развёртывание в Managed Service for Kubernetes

**Шаг 1. Создание кластера Kubernetes**

```bash
yc managed-kubernetes cluster create \
    --name traffic-sign-cluster \
    --network-name ml-vpc \
    --zone ru-central1-a \
    --subnet-name ml-subnet \
    --service-account-name ml-sa \
    --node-service-account-name ml-sa \
    --version 1.28 \
    --node-group name=main,scale-policy=auto,min=1,max=3
```

Получение учётных данных для `kubectl`:

```bash
yc managed-kubernetes cluster get-credentials traffic-sign-cluster --external
kubectl get nodes
```

**Шаг 2. Создание Deployment и Service**

**Файл `deployment.yaml`:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: traffic-sign
  labels:
    app: traffic-sign
spec:
  replicas: 2
  selector:
    matchLabels:
      app: traffic-sign
  template:
    metadata:
      labels:
        app: traffic-sign
    spec:
      containers:
        - name: app
          image: cr.yandex/<registry-id>/traffic-sign:v1
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "2Gi"
              cpu: "1000m"
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
```

**Файл `service.yaml`:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: traffic-sign-service
spec:
  type: LoadBalancer
  selector:
    app: traffic-sign
  ports:
    - port: 80
      targetPort: 8080
```

Применение манифестов:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

**Шаг 3. Настройка Ingress (опционально)**

Для маршрутизации по домену установите Ingress-контроллер и создайте Ingress-ресурс.

---

#### 2.2.5. Настройка CI/CD пайплайна

**Шаг 1. Создание GitHub-репозитория**

Создайте репозиторий `traffic-sign-service` на GitHub и загрузите код проекта.

**Шаг 2. Настройка секретов в GitHub**

В настройках репозитория (Settings → Secrets and variables → Actions) добавьте секреты:

| Секрет | Значение |
|--------|----------|
| `YC_SA_KEY_JSON` | Содержимое файла `key.json` |
| `YC_REGISTRY` | ID реестра Container Registry |
| `YC_CLUSTER_ID` | ID кластера Kubernetes |

**Шаг 3. Создание GitHub Actions workflow**

**Файл `.github/workflows/deploy.yml`:**

```yaml
name: Deploy to Yandex Cloud

on:
  push:
    branches: [ main ]

env:
  REGISTRY: cr.yandex/${{ secrets.YC_REGISTRY }}
  IMAGE_NAME: traffic-sign
  CLUSTER_ID: ${{ secrets.YC_CLUSTER_ID }}

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Install Yandex Cloud CLI
        run: |
          curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
          echo "$HOME/yandex-cloud/bin" >> $GITHUB_PATH
      
      - name: Authenticate in Yandex Cloud
        run: |
          echo "${{ secrets.YC_SA_KEY_JSON }}" > key.json
          yc config set service-account-key key.json
          yc config set cloud-id $(yc config get cloud-id)
          yc config set folder-id $(yc config get folder-id)
      
      - name: Configure Docker
        run: yc container registry configure-docker
      
      - name: Build, tag, and push image
        run: |
          IMAGE_TAG=${GITHUB_SHA}
          docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:$IMAGE_TAG .
          docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:$IMAGE_TAG
      
      - name: Get cluster credentials
        run: yc managed-kubernetes cluster get-credentials ${{ env.CLUSTER_ID }} --external
      
      - name: Update deployment image
        run: |
          kubectl set image deployment/traffic-sign app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          kubectl rollout status deployment/traffic-sign
```

**Шаг 4. Проверка работы CI/CD**

При пуше в ветку `main` автоматически:
1. Собирается Docker-образ с тегом, соответствующим SHA коммита.
2. Образ публикуется в Container Registry.
3. Kubernetes-деплоймент обновляется на новый образ.
4. Выполняется отслеживание статуса обновления.

---

### 2.3. Контрольные вопросы к лабораторной работе №1

1. Как вы обеспечили горизонтальное масштабирование сервиса?

2. Какие механизмы отказоустойчивости заложены в Kubernetes-деплойменте?

3. Почему вы выбрали именно такой формат сериализации модели (PyTorch SavedModel)?

4. Как бы вы организовали версионирование API?

5. В чём преимущества использования контейнеризации для ML-сервисов?

6. Как работает механизм Rolling Update в Kubernetes?

7. Какие типы Service в Kubernetes вы знаете и чем они отличаются?

8. Что такое livenessProbe и readinessProbe и как они влияют на работу сервиса?


## 3. Лабораторная работа №2. Пайплайн данных, бессерверный инференс и мониторинг качества

### 3.1. Цель работы

Построить пайплайн обработки данных и обучить модель распознавания дорожных знаков в DataSphere, организовать бессерверный инференс через Cloud Functions и API Gateway, а также настроить мониторинг качества модели с помощью DataLens.

---

### 3.2. Подготовка данных и обучение модели в DataSphere

#### 3.2.1. Создание проекта в DataSphere

1. Перейдите в Yandex Cloud и выберите сервис DataSphere.
2. Нажмите «Попробовать бесплатно» и войдите через Яндекс ID.
3. Создайте сообщество `ml-community` и привяжите платёжный аккаунт.
4. Внутри сообщества создайте проект `traffic-sign-training`.
5. Подключите Git-репозиторий с шаблоном кода для обучения.

#### 3.2.2. Настройка подключения к Object Storage

**Создание сервисного аккаунта и ключей доступа:**

```bash
yc iam service-account create --name storage-sa --description "Сервисный аккаунт для доступа к Object Storage"
yc iam access-key create --service-account-name storage-sa
```

Сохраните идентификатор ключа и секретный ключ.

**Создание S3 Connector в DataSphere:**

1. В интерфейсе проекта нажмите «Create resource» → выберите «S3 Connector».
2. Заполните поля:
   - **Name:** `gtsrb-storage`
   - **Endpoint:** `https://storage.yandexcloud.net/`
   - **Bucket:** `traffic-sign-dataset`
   - **Static access key ID:** идентификатор ключа сервисного аккаунта
   - **Static access key:** созданный секрет
   - **Mode:** Read and write

S3 Connector будет смонтирован в файловую систему проекта по пути `/mnt/gtsrb-storage/`.

#### 3.2.3. Загрузка датасета и подготовка данных

Датасет GTSRB (German Traffic Sign Recognition Benchmark) доступен по ссылке: https://zenodo.org/records/13741936/files/data.zip

В ноутбуке DataSphere напишите код для работы с данными:

```python
import os
import pandas as pd
import numpy as np
from PIL import Image
import torch
from torch.utils.data import Dataset, DataLoader
import torchvision.transforms as transforms
from sklearn.model_selection import train_test_split

# Путь к данным в смонтированном S3-бакете
DATA_PATH = "/mnt/gtsrb-storage/GTSRB/"
TRAIN_PATH = os.path.join(DATA_PATH, "Train")
TEST_PATH = os.path.join(DATA_PATH, "Test")

# Класс датасета для загрузки изображений
class GTSRBDataset(Dataset):
    def __init__(self, data_dir, transform=None):
        self.data_dir = data_dir
        self.transform = transform
        self.samples = []
        
        for class_id in range(43):
            class_dir = os.path.join(data_dir, str(class_id))
            if os.path.exists(class_dir):
                for img_file in os.listdir(class_dir):
                    if img_file.endswith('.ppm'):
                        self.samples.append((os.path.join(class_dir, img_file), class_id))
    
    def __len__(self):
        return len(self.samples)
    
    def __getitem__(self, idx):
        img_path, label = self.samples[idx]
        image = Image.open(img_path).convert('RGB')
        if self.transform:
            image = self.transform(image)
        return image, label

# Трансформации с аугментацией
train_transform = transforms.Compose([
    transforms.Resize((32, 32)),
    transforms.RandomRotation(10),
    transforms.RandomAffine(0, shear=10, scale=(0.8, 1.2)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

val_transform = transforms.Compose([
    transforms.Resize((32, 32)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

# Создание датасетов
train_dataset = GTSRBDataset(TRAIN_PATH, transform=train_transform)
test_dataset = GTSRBDataset(TEST_PATH, transform=val_transform)

train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True, num_workers=4)
test_loader = DataLoader(test_dataset, batch_size=64, shuffle=False, num_workers=4)
```

#### 3.2.4. Обучение модели

Используйте предобученную модель ResNet-18 с заменой последнего слоя на 43 класса:

```python
import torch.nn as nn
import torch.optim as optim
from torchvision import models

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

model = models.resnet18(pretrained=True)
model.fc = nn.Linear(model.fc.in_features, 43)
model = model.to(device)

criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Цикл обучения (рекомендуется 10-15 эпох)
num_epochs = 10

for epoch in range(num_epochs):
    # Обучение
    model.train()
    running_loss = 0.0
    for images, labels in train_loader:
        images, labels = images.to(device), labels.to(device)
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()
        running_loss += loss.item()
    
    # Валидация
    model.eval()
    correct = 0
    total = 0
    with torch.no_grad():
        for images, labels in test_loader:
            images, labels = images.to(device), labels.to(device)
            outputs = model(images)
            _, predicted = torch.max(outputs, 1)
            total += labels.size(0)
            correct += (predicted == labels).sum().item()
    
    accuracy = 100 * correct / total
    print(f'Epoch {epoch+1}/{num_epochs}, Loss: {running_loss/len(train_loader):.4f}, Accuracy: {accuracy:.2f}%')
```

#### 3.2.5. Сохранение модели

Сохраните обученную модель в Object Storage:

```python
# Сохранение модели в формате PyTorch
model_path = "/mnt/gtsrb-storage/models/resnet18_gtsrb.pth"
torch.save(model.state_dict(), model_path)

# Конвертация в ONNX для инференса
dummy_input = torch.randn(1, 3, 32, 32).to(device)
onnx_path = "/mnt/gtsrb-storage/models/resnet18_gtsrb.onnx"
torch.onnx.export(
    model, 
    dummy_input, 
    onnx_path,
    input_names=['input'],
    output_names=['output'],
    dynamic_axes={'input': {0: 'batch_size'}, 'output': {0: 'batch_size'}}
)

# Создание карточки модели
import json
model_card = {
    "model_name": "resnet18_gtsrb",
    "version": "1.0.0",
    "framework": "PyTorch",
    "format": "ONNX",
    "accuracy": accuracy,
    "num_classes": 43
}
with open("/mnt/gtsrb-storage/models/model_card.json", "w") as f:
    json.dump(model_card, f, indent=2)
```

---

### 3.3. Бессерверный инференс через Cloud Functions и API Gateway

#### 3.3.1. Создание Cloud Function

**Шаг 1. Создание функции**

```bash
yc serverless function create --name traffic-sign-inference
```

**Шаг 2. Подготовка кода функции**

Создайте ZIP-архив с кодом. **Файл `handler.py`:**

```python
import json
import io
import base64
import boto3
import torch
import torchvision.transforms as transforms
import numpy as np
from PIL import Image

# Конфигурация S3
S3_ENDPOINT = "https://storage.yandexcloud.net"
BUCKET_NAME = "traffic-sign-dataset"
MODEL_KEY = "models/resnet18_gtsrb.onnx"

# Кэширование модели и трансформаций
_model = None
_transform = None

def get_model():
    global _model
    if _model is None:
        import onnxruntime as ort
        
        # Скачивание модели из S3
        s3_client = boto3.client('s3', endpoint_url=S3_ENDPOINT)
        response = s3_client.get_object(Bucket=BUCKET_NAME, Key=MODEL_KEY)
        model_data = response['Body'].read()
        
        # Сохранение во временную директорию
        temp_path = "/tmp/model.onnx"
        with open(temp_path, "wb") as f:
            f.write(model_data)
        
        _model = ort.InferenceSession(temp_path)
    return _model

def get_transform():
    global _transform
    if _transform is None:
        _transform = transforms.Compose([
            transforms.Resize((32, 32)),
            transforms.ToTensor(),
            transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
        ])
    return _transform

def handler(event, context):
    """
    Функция инференса для распознавания дорожных знаков
    """
    try:
        # Получение изображения из запроса
        body = json.loads(event.get('body', '{}'))
        image_data = body.get('image')
        
        if not image_data:
            return {
                'statusCode': 400,
                'body': json.dumps({'error': 'No image provided'})
            }
        
        # Декодирование base64
        image_bytes = base64.b64decode(image_data)
        image = Image.open(io.BytesIO(image_bytes)).convert('RGB')
        
        # Предобработка
        transform = get_transform()
        tensor = transform(image).unsqueeze(0).numpy()
        
        # Инференс
        model = get_model()
        inputs = {model.get_inputs()[0].name: tensor}
        outputs = model.run(None, inputs)
        
        # Получение предсказания
        probabilities = np.exp(outputs[0]) / np.sum(np.exp(outputs[0]), axis=1, keepdims=True)
        predicted_class = int(np.argmax(probabilities[0]))
        confidence = float(np.max(probabilities[0]))
        
        return {
            'statusCode': 200,
            'body': json.dumps({
                'predicted_class': predicted_class,
                'confidence': confidence
            }),
            'headers': {'Content-Type': 'application/json'}
        }
    
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps({'error': str(e)})
        }
```

**Файл `requirements.txt` для функции:**

```
torch==2.1.0
torchvision==0.16.0
onnxruntime==1.15.1
boto3==1.28.64
Pillow==10.1.0
numpy==1.24.3
```

**Шаг 3. Развёртывание функции**

```bash
# Создание ZIP-архива
zip -r function.zip handler.py requirements.txt

# Создание версии функции
yc serverless function version create \
    --function-name traffic-sign-inference \
    --runtime python311 \
    --entrypoint handler.handler \
    --memory 1024 \
    --execution-timeout 10s \
    --service-account-name storage-sa \
    --source-path function.zip
```

#### 3.3.2. Настройка API Gateway

**Файл `spec.yaml` (OpenAPI-спецификация):**

```yaml
openapi: 3.0.0
info:
  title: Traffic Sign Recognition API
  version: 1.0.0
servers:
  - url: https://d5dxxxxxxxxxxx.apiqy.yandexcloud.net

paths:
  /predict:
    post:
      summary: Распознавание дорожного знака
      operationId: predict
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                image:
                  type: string
                  description: Base64-кодированное изображение
      responses:
        '200':
          description: Успешный ответ
          content:
            application/json:
              schema:
                type: object
                properties:
                  predicted_class:
                    type: integer
                  confidence:
                    type: number
        '429':
          description: Превышен лимит запросов
      
      x-yc-apigateway-integration:
        type: cloud-functions
        function-id: <function-id>
        tag: "$latest"
        service_account_id: <service-account-id>
        payload_format_version: "1.0"

# Rate Limiting
x-yc-apigateway-rate-limit:
  - path: /predict
    limit: 100
    period: 60
```

**Создание API-шлюза:**

```bash
yc serverless api-gateway create \
    --name traffic-sign-api \
    --spec spec.yaml
```

**Настройка аутентификации:**

Создайте API-ключ для доступа:

```bash
yc iam api-key create --service-account-name <service-account-name>
```

Используйте ключ в заголовке `X-API-Key` при тестировании.

#### 3.3.3. Тестирование инференса

```bash
curl -X POST https://<api-gateway-url>/predict \
    -H "Content-Type: application/json" \
    -H "X-API-Key: <api-key>" \
    -d '{"image": "<base64-encoded-image>"}'
```

Ожидаемый ответ:

```json
{
    "predicted_class": 12,
    "confidence": 0.9618
}
```

---

### 3.4. Мониторинг качества модели с помощью DataLens

#### 3.4.1. Сбор метрик инференса

В функцию инференса добавьте структурированное логирование. Логи автоматически собираются в Yandex Cloud Logging.

Для долгосрочного хранения метрик настройте экспорт логов в Managed Service for ClickHouse.

**Создание таблицы в ClickHouse:**

```sql
CREATE TABLE inference_logs (
    timestamp DateTime,
    predicted_class Int32,
    confidence Float32,
    latency_ms Float32,
    request_id String,
    model_version String
) ENGINE = MergeTree()
ORDER BY timestamp;
```

#### 3.4.2. Создание датасета в DataLens

1. В DataLens подключите источник данных — Managed Service for ClickHouse.
2. Создайте датасет с SQL-запросом:

```sql
SELECT 
    toStartOfHour(timestamp) as hour,
    predicted_class,
    avg(confidence) as avg_confidence,
    count() as requests_count,
    avg(latency_ms) as avg_latency
FROM inference_logs
WHERE timestamp > now() - interval 7 day
GROUP BY hour, predicted_class
```

#### 3.4.3. Создание дашбордов

Постройте следующие визуализации:

| Визуализация | Описание |
|--------------|----------|
| **Точность модели по времени** | Средняя уверенность модели в предсказаниях за последние 7 дней |
| **Распределение предсказанных классов** | Гистограмма частоты встречаемости классов |
| **Количество запросов и ошибок** | Временной ряд с количеством запросов и процентом ошибок (confidence < 0.5) |
| **Latency инференса** | График с перцентилями времени ответа (p50, p95, p99) |
| **Сравнение версий моделей** | Таблица с метриками для разных версий модели |

#### 3.4.4. Публикация дашборда

Опубликуйте дашборд с настройкой прав доступа для команды. Настройте автоматические обновления данных каждые 15 минут.

---

### 3.5. Интеграция с дополнительными сервисами (опционально)

Для расширения функциональности можно добавить интеграцию с Yandex Vision для предварительного детектирования знаков на изображении.

---

### 3.6. Контрольные вопросы к лабораторной работе №2

1. Какие этапы пайплайна данных вы реализовали и почему именно их?

2. В чём преимущества бессерверной архитектуры для инференса по сравнению с Kubernetes?

3. Какие метрики качества вы отслеживаете и как они помогают в эксплуатации?

4. Как вы организовали бы А/В-тестирование двух версий модели?

5. Как DataLens помогает в мониторинге, какие дашборды вы построили?

6. В чём отличие DataSphere от традиционных Jupyter Notebook?

7. Как работает кэширование модели в Cloud Functions и почему это важно?

8. Что такое дрейф данных (data drift) и как его можно обнаружить?


## Приложение А. Примеры кода

### А.1. Класс датасета для GTSRB

```python
class GTSRBDataset(Dataset):
    def __init__(self, data_dir, transform=None):
        self.data_dir = data_dir
        self.transform = transform
        self.samples = []
        
        for class_id in range(43):
            class_dir = os.path.join(data_dir, str(class_id))
            if os.path.exists(class_dir):
                for img_file in os.listdir(class_dir):
                    if img_file.endswith('.ppm'):
                        self.samples.append((os.path.join(class_dir, img_file), class_id))
    
    def __len__(self):
        return len(self.samples)
    
    def __getitem__(self, idx):
        img_path, label = self.samples[idx]
        image = Image.open(img_path).convert('RGB')
        if self.transform:
            image = self.transform(image)
        return image, label
```

### А.2. Dockerfile для ML-сервиса

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/
COPY models/ ./models/

EXPOSE 8080

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
```

### А.3. GitHub Actions workflow

```yaml
name: Deploy to Yandex Cloud

on:
  push:
    branches: [ main ]

env:
  REGISTRY: cr.yandex/${{ secrets.YC_REGISTRY }}
  IMAGE_NAME: traffic-sign

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Yandex Cloud CLI
        run: |
          curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
          echo "$HOME/yandex-cloud/bin" >> $GITHUB_PATH
      
      - name: Authenticate
        run: |
          echo "${{ secrets.YC_SA_KEY_JSON }}" > key.json
          yc config set service-account-key key.json
      
      - name: Configure Docker
        run: yc container registry configure-docker
      
      - name: Build and push
        run: |
          IMAGE_TAG=${GITHUB_SHA}
          docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:$IMAGE_TAG .
          docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:$IMAGE_TAG
      
      - name: Deploy to Kubernetes
        run: |
          yc managed-kubernetes cluster get-credentials ${{ secrets.YC_CLUSTER_ID }} --external
          kubectl set image deployment/traffic-sign app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          kubectl rollout status deployment/traffic-sign
```


## Приложение Б. Контрольные вопросы и критерии оценивания

### Б.1. Контрольные вопросы

**К лабораторной работе №1:**
1. Как вы обеспечили горизонтальное масштабирование сервиса?
2. Какие механизмы отказоустойчивости заложены в Kubernetes-деплойменте?
3. Почему вы выбрали именно такой формат сериализации модели?
4. Как бы вы организовали версионирование API?
5. В чём преимущества использования контейнеризации для ML-сервисов?
6. Как работает механизм Rolling Update в Kubernetes?
7. Какие типы Service в Kubernetes вы знаете и чем они отличаются?
8. Что такое livenessProbe и readinessProbe?

**К лабораторной работе №2:**
1. Какие этапы пайплайна данных вы реализовали и почему?
2. В чём преимущества бессерверной архитектуры для инференса?
3. Какие метрики качества вы отслеживаете и как они помогают?
4. Как организовать А/В-тестирование двух версий модели?
5. Как DataLens помогает в мониторинге?
6. В чём отличие DataSphere от традиционных Jupyter Notebook?
7. Как работает кэширование модели в Cloud Functions?
8. Что такое дрейф данных (data drift)?

### Б.2. Критерии оценивания

Каждая лабораторная работа оценивается по 10-балльной шкале:

| Критерий | Баллы |
|----------|-------|
| Подготовка облачной инфраструктуры | 2 |
| Контейнеризация и развёртывание | 2 |
| Настройка CI/CD пайплайна | 2 |
| Обучение модели (Лаб. 2) / Мониторинг (Лаб. 2) | 2 |
| Качество отчёта и ответы на вопросы | 2 |
| **Итого** | **10** |

**Шкала перевода в традиционную оценку:**

| Баллы | Оценка |
|-------|--------|
| 9–10 | «Отлично» |
| 7–8 | «Хорошо» |
| 5–6 | «Удовлетворительно» |
| 0–4 | «Неудовлетворительно» |


## Список использованных источников

1. Yandex Cloud. Документация интерфейса командной строки yc. — URL: https://cloud.yandex.com/ru/docs/cli/

2. Yandex Cloud. Документация Managed Service for Kubernetes. — URL: https://cloud.yandex.com/ru/docs/managed-kubernetes/

3. Yandex Cloud. Документация Container Registry. — URL: https://cloud.yandex.com/ru/docs/container-registry/

4. Yandex Cloud. Документация DataSphere. — URL: https://cloud.yandex.com/ru/docs/datasphere/

5. Yandex Cloud. Документация Cloud Functions. — URL: https://cloud.yandex.com/ru/docs/functions/

6. Yandex Cloud. Документация API Gateway. — URL: https://cloud.yandex.com/ru/docs/api-gateway/

7. Yandex Cloud. Документация DataLens. — URL: https://cloud.yandex.com/ru/docs/datalens/

8. FastAPI. Документация FastAPI. — URL: https://fastapi.tiangolo.com/

9. Docker. Документация Docker. — URL: https://docs.docker.com/

10. Kubernetes. Документация Kubernetes. — URL: https://kubernetes.io/docs/home/

11. PyTorch. Документация PyTorch. — URL: https://pytorch.org/docs/stable/index.html

12. GitHub. Документация GitHub Actions. — URL: https://docs.github.com/en/actions

13. Stallkamp J., Schlipsing M., Salmen J., Igel C. Man vs. computer: Benchmarking machine learning algorithms for traffic sign recognition // Neural Networks. — 2012. — URL: https://benchmark.ini.rub.de/?section=gtsrb&subsection=dataset

14. German Traffic Sign Recognition Benchmark (GTSRB). — URL: https://zenodo.org/records/13741936/files/data.zip
