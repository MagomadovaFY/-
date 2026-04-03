# Лабораторная работа №1
## Реализация RPC-сервиса с использованием gRPC

**Студент:** [Ваше ФИО]
**Вариант:** 11
**Тип RPC:** Client streaming RPC

## Цель работы
Освоить принципы удаленного вызова процедур (RPC) и реализовать клиент-серверное приложение с использованием gRPC.

## Описание задания
Сервис MetricsCollector с методом CollectMetrics(stream Metric) для сбора потока метрик с клиентских приложений.

## Листинг .proto файла
```protobuf
syntax = "proto3";

```
### Листинг сервера (server.py)
import grpc
from concurrent import futures
import metrics_pb2
import metrics_pb2_grpc

class MetricsCollectorServicer(metrics_pb2_grpc.MetricsCollectorServicer):
    
    def CollectMetrics(self, request_iterator, context):
        metrics_count = 0
        
        print("Начало сбора метрик от клиента...")
        
        for metric in request_iterator:
            metrics_count += 1
            print(f"  Получена метрика: {metric.name} = {metric.value} (источник: {metric.source})")
        
        print(f"Сбор завершён. Всего получено метрик: {metrics_count}")
        
        return metrics_pb2.CollectMetricsResponse(
            message=f"Успешно собрано {metrics_count} метрик",
            total_metrics=metrics_count,
            status="OK"
        )

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    metrics_pb2_grpc.add_MetricsCollectorServicer_to_server(
        MetricsCollectorServicer(), server
    )
    server.add_insecure_port('[::]:50051')
    print("Сервер запущен на порту 50051...")
    server.start()
    server.wait_for_termination()

if __name__ == '__main__':
    serve()

### Листинг client.py (полный)
import grpc
import time
import random
import metrics_pb2
import metrics_pb2_grpc

def generate_metrics():
    metric_names = ["cpu_usage", "memory_usage", "disk_io", "network_rx", "network_tx"]
    sources = ["server01", "server02", "server03"]
    
    for i in range(10):
        metric = metrics_pb2.Metric(
            name=random.choice(metric_names),
            value=random.uniform(0, 100),
            timestamp=int(time.time()),
            source=random.choice(sources)
        )
        print(f"Отправка: {metric.name} = {metric.value:.2f} (источник: {metric.source})")
        yield metric
        time.sleep(0.5)

def run():
    channel = grpc.insecure_channel('localhost:50051')
    stub = metrics_pb2_grpc.MetricsCollectorStub(channel)
    
    print("=== Отправка потока метрик на сервер ===\n")
    
    try:
        response = stub.CollectMetrics(generate_metrics())
        print(f"\n=== Ответ сервера ===")
        print(f"Сообщение: {response.message}")
        print(f"Всего метрик: {response.total_metrics}")
        print(f"Статус: {response.status}")
    except grpc.RpcError as e:
        print(f"Ошибка RPC: {e.code()} - {e.details()}")

if __name__ == '__main__':
    run()

### Листинг metrics.proto
syntax = "proto3";
package metrics;
service MetricsCollector {
    rpc CollectMetrics(stream Metric) returns (CollectMetricsResponse) {}
}
message Metric {
    string name = 1;
    double value = 2;
    int64 timestamp = 3;
    string source = 4;
}
message CollectMetricsResponse {
    string message = 1;
    int32 total_metrics = 2;
    string status = 3;
}

# Выводы
В ходе работы был реализован gRPC-сервис для сбора метрик с использованием client streaming RPC. Сервер успешно принял и обработал 10 метрик, клиент получил подтверждение. Освоены навыки работы с Protocol Buffers, генерации кода и реализации gRPC-взаимодействия на Python.


---

## 📁 Структура репозитория
grpc_metrics_lab/
├── README.md
├── metrics.proto
├── server.py
├── client.py
├── metrics_pb2.py
├── metrics_pb2_grpc.py
├── venv/
└── screenshots/
├── server.png
├── client.png
├── proto.png
└── code.png
