# Модуль 1. Проектирование архитектуры ИИ-приложений

## Назначение модуля
Модуль формирует навыки проектирования и развёртывания промышленных ИИ-систем. Студенты осваивают архитектурные паттерны (монолитная, сервисная, микросервисная, гибридная), компоненты ИИ-систем (слои данных, модели, API, мониторинга), принципы масштабируемости, отказоустойчивости и наблюдаемости. Рассматривается выбор технологического стека для задач компьютерного зрения, обработки естественного языка и рекомендательных систем. Практическая часть включает работу с облачной платформой Yandex Cloud: создание платёжного аккаунта, настройку авторизации через Yandex ID, использование сервисов Cloud Functions, DataSphere, DataLens, Managed Kubernetes, Object Storage и управляемых баз данных.

## Проверяемые результаты обучения
- **PL-1.1 (Средний)** — Умеет разрабатывать и отлаживать прикладные решения на Python с использованием основных библиотек для серверного программирования (FastAPI, Flask, Django REST Framework) и многопоточности.
- **PL-1.2 (Продвинутый)** — Умеет обоснованно выбирать инструменты (включая сравнительный анализ стеков для обработки научных данных, МО и визуализации) и разрабатывать собственные расширяющие компоненты (слои, трансформеры, пайплайны, метрики) для бесшовной интеграции с существующими библиотеками машинного обучения.

## Содержание модуля
### Тематические сущности
- **Типы архитектур**: монолитная, сервисная, микросервисная, гибридная.
- **Слой данных**: хранилища, пайплайны данных, аугментация и предобработка.
- **Слой модели**: регистрация моделей, отслеживание метрик и гиперпараметров, форматы сериализации (ONNX, PyTorch, TensorFlow SavedModel), каналы развёртывания.
- **Слой API**: REST/gRPC/GraphQL, эндпоинты, версионирование, аутентификация/авторизация (API keys, JWT, Yandex ID), Rate limiting, таймауты, ретраи.
- **Принципы проектирования**: масштабируемость (горизонтальное и вертикальное масштабирование), отказоустойчивость (retry с экспоненциальной задержкой, fallback-модель, кэшированный ответ, репликация и мультизональность), наблюдаемость (логи, метрики, трассировка, структурированное логирование).
- **Выбор стека технологий**:
  - CV: PyTorch, TensorFlow, OpenCV, GPU (NVIDIA), TPU, нейропроцессоры, Albumentations, MMDetection, YOLO.
  - NLP: трансформеры, токенизаторы, эмбеддинги, fine-tuning, LLM, RAG, промпт-инжиниринг.
  - Рекомендательные системы: коллаборативная фильтрация, контентная, гибридная, векторный поиск, хранение пользовательских профилей.
  - Общие слои: контейнеризация Docker, оркестрация Kubernetes, CI/CD для ML.
- **Облачная платформа Yandex Cloud**:
  - Основные сущности: регионы и зоны доступности, Compute Cloud, IAM, сервисные аккаунты, роли, VPC, подсети, Security Groups, NAT, KMS, Secret Manager.
  - Популярные сервисы: Object Storage, Managed Databases (PostgreSQL, ClickHouse), Yandex Managed Service for Kubernetes, Yandex Vision, SpeechKit, Translate.
- **Авторизация через Yandex ID**: OAuth 2.0 / OpenID Connect, получение Access Token / Refresh Token, использование токена для API-вызовов.
- **Создание тестового платёжного аккаунта**: типы аккаунтов (пробный (грант), платный), активация пробного периода, привязка платёжной карты, создание сервисного аккаунта с правами для работы через CLI/SDK.
- **Практика работы с сервисами Yandex Cloud**:
  - Cloud Functions: создание функции на Python/Go/Node.js, интеграция с API Gateway для публичного доступа, логирование и мониторинг функции.
  - DataSphere: проекты и ноутбуки, работа с Git внутри проекта, подключение к S3, PostgreSQL, ClickHouse, запуск тренировочных заданий с GPU, совместная работа команды.
  - DataLens: подключение источников данных, создание датасетов, визуализация, публикация дашбордов с контролем доступа, использование в мониторинге качества модели.

## Контрольно-измерительные материалы
- [Лабораторная работа](./lab.md)
- [Доклад](./report.md)

## Ресурсы
### Основная литература
1. Лутц М. Изучаем Python. — 5-е изд. — СПб.: Диалектика, 2021. — 1360 с. — ISBN 978-5-907144-85-0.
2. Жерон О. Прикладное машинное обучение с помощью Scikit-Learn и TensorFlow. — 2-е изд. — М.: ДМК Пресс, 2023. — 688 с. — ISBN 978-5-93700-123-7.
3. Маккини У. Python и анализ данных. — 3-е изд. — М.: ДМК Пресс, 2022. — 560 с. — ISBN 978-5-97060-998-9.
4. Рашка С., Мирджалили В. Машинное обучение с PyTorch и Scikit-Learn. — М.: ДМК Пресс, 2023. — 790 с. — ISBN 978-5-93700-145-9.
5. Molnar C. Interpretable Machine Learning: A Guide for Making Black Box Models Explainable. — 2nd ed. — 2022. — 350 p. — URL: https://christophm.github.io/interpretable-ml-book/
6. Гинько А. Ю. Анализ и визуализация данных в Yandex DataLens: Подробное руководство: от новичка до эксперта. — М.: ДМК Пресс, 2023. — 358 с. — ISBN 978-5-93700-171-9.
7. Моррис К. Программирование инфраструктуры: как создаются адаптивные облачные системы. — 2-е изд. — СПб.: БХВ-Петербург, 2024. — 415 с. — ISBN 978-5-9775-1901-4.
8. Вьяс Дж., Лав К. Kubernetes изнутри. — М.: ДМК Пресс, 2023. — 378 с. — ISBN 978-5-93700-153-5.
9. Kingsley M. S. Cloud Technologies and Services: Theoretical Concepts and Practical Applications. — 1st ed. — Springer, 2024. — URL: https://link.springer.com/book/10.1007/978-3-031-33669-0

### Официальная документация
- **Yandex Cloud** (общая) — https://cloud.yandex.com/docs
- **Kubernetes** — https://kubernetes.io/docs/
- **Docker** — https://docs.docker.com/
- **FastAPI** — https://fastapi.tiangolo.com/

**Документация по сервисам Yandex Cloud (дополнительно):**
- Обзор платформы — https://cloud.yandex.com/ru/docs/overview/
- Ролевая модель и IAM — https://cloud.yandex.com/ru/docs/overview/roles-and-resources
- Полный список сервисов — https://cloud.yandex.com/ru/docs/overview/concepts/services
- Yandex Compute Cloud — https://cloud.yandex.com/ru/docs/compute/
- Yandex Virtual Private Cloud (VPC) — https://cloud.yandex.com/ru/docs/vpc/
- Identity and Access Management (IAM) — https://cloud.yandex.com/ru/docs/iam/
- Yandex Object Storage — https://cloud.yandex.com/ru/docs/storage/
- Yandex Managed Databases (PostgreSQL, ClickHouse и др.) — https://cloud.yandex.com/ru/docs/managed-databases/
- Yandex Managed Service for Kubernetes — https://cloud.yandex.com/ru/docs/managed-kubernetes/
- Yandex Cloud Functions — https://cloud.yandex.com/ru/docs/functions/
- Yandex API Gateway — https://cloud.yandex.com/ru/docs/api-gateway/
- Yandex DataSphere — https://cloud.yandex.com/ru/docs/datasphere/
- Yandex DataLens — https://cloud.yandex.com/ru/docs/datalens/
- Yandex Key Management Service (KMS) — https://cloud.yandex.com/ru/docs/kms/
- Yandex Lockbox (Secret Manager) — https://cloud.yandex.com/ru/docs/lockbox/
- Yandex Cloud CLI — https://cloud.yandex.com/ru/docs/cli/
- Yandex Cloud Logging — https://cloud.yandex.com/ru/docs/logging/