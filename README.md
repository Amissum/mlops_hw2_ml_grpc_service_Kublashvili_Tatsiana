# mlops_hw2_ml_gprc_service_Kublashvili_Tatsiana

Минимальный gRPC-сервис для обслуживания ML-модели.  
Сервис предоставляет два эндпоинта:

- **/health** — проверка готовности сервиса и версии модели;
- **/predict** — получение предсказания модели и confidence.

Модель загружается из файла по пути `MODEL_PATH` и использует версию, указанную в переменной `MODEL_VERSION`.

---

## 1. Структура проекта

```text
ml_grpc_service/
├── protos/
│   └── model.proto
├── server/
│   └── server.py
├── client/
│   └── client.py
├── models/
│   └── model.pkl
├── requirements.txt
├── Dockerfile
└── README.md
```

## 2. Установка и запуск локально (без Docker-контейнеризации)
### 2.1. Установка зависимостей
```bash
pip install -r requirements.txt
```

### 2.2. (опционально) генерация gRPC-кода из протокола
```bash
python -m grpc_tools.protoc \
  -I=./protos \
  --python_out=. \
  --grpc_python_out=. \
  protos/model.proto
```

### 2.3. Запуск gRPC-сервера
```bash
export MODEL_PATH=/app/models/model.pkl
export MODEL_VERSION=v1.0.0
python -m server.server
```

## 3. Сборка и запуск в Docker
```bash
# 1. Сборка образа
docker build -t grpc-ml-service .

# 2. Запуск контейнера
docker run --rm -p 50051:50051 \
  -e MODEL_PATH=/app/models/model.pkl \
  -e MODEL_VERSION=v1.0.0 \
  grpc-ml-service

```
После запуска сервис доступен на localhost:50051.

## 4. Примеры вызовов

**/health**

Через grpcurl:
```bash
grpcurl -plaintext localhost:50051 mlservice.v1.PredictionService.Health
```
Пример ожидаемого ответа:
```bash
{
  "status": "ok",
  "modelVersion": "v1.0.0"
}
```

**/predict**

Через Python-клиента (вызывать из корневой директории репозитория):
```bash
python -m client.client
```
Пример ожидаемого ответа:
```bash
{
  {
  "prediction": "0",
  "confidence": 0.9816,
  "modelVersion": "v1.0.0"
}

}
```

## 5. Скриншоты запусков команд внутри Docker-контейнера:
результат вызова /health после запуска контейнера
![both_health_and_predict.png](https://raw.githubusercontent.com/bucketio/img2/main/2025/12/07/1765141707089-68137103-e3ab-43e9-af82-791da88c9e6b.png 'both_health_and_predict.png')
результат вызова /predict после запуска контейнера
![health_command.png](https://raw.githubusercontent.com/bucketio/img17/main/2025/12/07/1765141708458-45820723-82c8-4d6d-93aa-73899021a3b3.png 'health_command.png')
результат вызовов /health и /predict после запуска контейнера
![predict_command.png](https://raw.githubusercontent.com/bucketio/img10/main/2025/12/07/1765141709580-374f15e5-cb11-4028-95d8-ec235cf14578.png 'predict_command.png')
