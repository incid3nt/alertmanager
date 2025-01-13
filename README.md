# Домашнее задание к занятию «Prometheus. Часть 2» - Еноктаев Олег



---

### Задание 1

Создайте файл с правилом оповещения, как в лекции, и добавьте его в конфиг Prometheus.

Погасите node exporter, стоящий на мониторинге, и прикрепите скриншот раздела оповещений Prometheus, где оповещение будет в статусе Pending

1. nano /etc/prometheus/netology-test.yml
```
groups:  # Список групп

  - name: netology-test  # Имя группы
    rules:  # Список правил текущей группы
      - alert: InstanceDown  # Название текущего правила
        expr: up == 0  # Логическое выражение
        for: 1m  # Время ожидания перед срабатыванием оповещения
        labels:
          severity: critical  # Уровень критичности события
        annotations:  # Описания
          description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.'  # Полное описание алерта
          summary: Instance {{ $labels.instance }} down  # Краткое описание алерта
```
чтобы правило заработало, добавляем в prometheus запись о правиле в раздел rule_files:
nano /etc/prometheus.yml
```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           - localhost:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
   - "netology-test.yml"
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090", "localhost:9100",  "192.168.1.27:9100", "localhost:9323"]
```
Гасим node-exporter:
```
systemctl stop node-exporter
```
![alert](https://github.com/incid3nt/alertmanager/blob/main/img/chrome_qNjUNUPTtS.png)
---

### Задание 2

Установите Alertmanager и интегрируйте его с Prometheus.

Прикрепите скриншот Alerts из Prometheus, где правило оповещения будет в статусе Fireing, и скриншот из Alertmanager, где будет видно действующее правило оповещения



---

### Задание 3

Активируйте экспортёр метрик в Docker и подключите его к Prometheus.

 приложите скриншот браузера с открытым эндпоинтом, а также скриншот списка таргетов из интерфейса Prometheus.*



### Задание 4

Создайте свой дашборд Grafana с различными метриками Docker и сервера, на котором он стоит.
