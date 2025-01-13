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

2. Скачаем alertmanager 
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
```
распакуем:
```
tar -xvf alertmanager-0.24.0.linux-amd64.tar.gz
```
Скопируем содержимое архива в папки:
```
cp ./alertmanager /usr/local/bin/
cp ./amtool /usr/local/bin/
```
Скопируем config в папку в Prometheus:
```
cp ./alertmanager.yml /etc/prometheus/
```
Передаем пользователю Prometheus права на файл:
```
chown -R prometheus:prometheus /etc/prometheus/alertmanager.yml
```
Создаем сервис для работы с Node-Exporter:
nano /etc/systemd/system/prometheus-alertmanager.service
```
[Unit]
Description=Alermanager Service - [Еноктаев Олег]
After=network.target
[Service]
EnvironmentFile=-/etc/default/alertmanager
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/prometheus/alertmanager.yml --storage.path=/var/lib/prometheus/alertmanager $ARGS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

Пропишем автозапуск:
```
systemctl enable prometheus-alertmanager
```
Запустим сервис:
```
systemctl start prometheus-alertmanager
```
Проверим статус:
```
systemctl start prometheus-alertmanager
```
Подключим alertmanager к prometheus:
Добавим в config-файл Prometheus подключение к Alertmanager:
nano /etc/prometheus/prometheus.yml
Приведем раздел Alertmanager configuration к виду:
```
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           - localhost:9093
```
перезапустим prometheus:
```
systemctl restart prometheus
```

![alert](https://github.com/incid3nt/alertmanager/blob/main/img/chrome_ONttIykuvW.png)

Настройка оповещений: 
откроем конфиг файл alertmanager:
```
nano /etc/prometheus/alertmanager.yml
```
```
global:

route:
  group_by: ['alertname']  # Группировка оповещений по имени
  group_wait: 30s           # Время ожидания перед отправкой первого оповещения
  group_interval: 10m       # Интервал между уведомлениями о новых сработках
  repeat_interval: 60m      # Интервал повтора уведомлений
  receiver: 'email'         # Способ доставки оповещений

receivers:
  - name: 'email'
    email_configs:
      - to: 'yourmailto@todomain.com'
        from: 'yourmailfrom@fromdomain.com'
        smarthost: 'mailserver:25'
        auth_username: 'user'
        auth_identity: 'user'
        auth_password: 'paS$w0rd'  # Рекомендуется заменить на переменную окружения
```

![alert](https://github.com/incid3nt/alertmanager/blob/main/img/chrome_DHALmL5Lbv.png)
---

### Задание 3

Активируйте экспортёр метрик в Docker и подключите его к Prometheus.

 приложите скриншот браузера с открытым эндпоинтом, а также скриншот списка таргетов из интерфейса Prometheus.*

3. 

### Задание 4

Создайте свой дашборд Grafana с различными метриками Docker и сервера, на котором он стоит.
