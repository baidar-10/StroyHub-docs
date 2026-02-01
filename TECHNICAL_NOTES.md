# Server Sizing: CPU / RAM / RPS

## Входные данные

Нужно определить:

* **RPS** (avg / peak ×3–5)
* **Active users** (обычно 5–10% от всех)
* **Latency target** (p95)
* **Тип запросов** (CPU или I/O bound)
* **Стек** (Go / Java / Node)

---

## Пользователи → RPS

```
RPS = ActiveUsers × ActionsPerSecond
```

**Пример**

```
10 000 users
5% active → 500
1 action / 2 sec → 250 RPS
```

---

## CPU

```
CPU cores ≈ RPS × avg_cpu_time (sec) × 1.5
```

**Пример**

```
250 RPS × 0.008s = 2 cores
С запасом → 4 vCPU
```

> avg_cpu_time берётся из load-теста или APM

---

## RAM

Считается от состояния, не от RPS

```
RAM = OS + runtime + app + cache + connections
```

**Ориентиры**

* OS: 0.5–1 GB
* Go: 100–400 MB
* Node: 300–800 MB
* JVM: Xmx + 30%
* Cache: по данным

👉 Обычно **4–8 GB**

---

## Диск (IOPS)

* Логи / БД / очереди → **SSD / NVMe**
* HDD >100 IOPS не держит

---

## Сеть

```
Bandwidth = RPS × avg_response_size
```

**Пример**

```
250 RPS × 50 KB ≈ 100 Mbps
```

👉 бери **1 Gbps**

---

## Типовой сервер

| Ресурс  | Значение       |
| ------- | -------------- |
| CPU     | 4–8 vCPU       |
| RAM     | 8 GB           |
| Disk    | NVMe 50–100 GB |
| Network | 1 Gbps         |

---

## Best Practice

1. Load test (k6 / JMeter)
2. Проверить CPU / RAM / p95
3. Добавить 30–50% запас
4. Включить autoscaling
