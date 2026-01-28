— Перейти в [[HUB]] —
> [!INFO] Что такое **Grafana** и **Prometheus**?
> Популярная связка инструментов для мониторинга.
> **Prometheus** — собирает и хранит метрики производительности с серверов и приложений;
> **Grafana** — визуализирует эти данные, превращая их в графики и дашборды.

Установка дистрибутивов:
```bash
apt-get update && apt-get install grafana prometheus prometheus-node_exporter prometheus-alertmanager
```

> [!INFO] vim /etc/prometheus/prometheus.yml
```yaml
rule_files:
  - "alert.rules.yml"  # создадим файл позже

# конфиг интеграции с alertmanager
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093  # где работает alertmanager

scrape_configs:
  # правило для сбора метрик с самого prometheus
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # правило для сбора метрик с node exporter
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```
После внесенных изменений, перезапускаем Prometheus:
```bash
systemctl restart prometheus
```

### Создаем файлик alert
> [!VOTE] /etc/prometheus/alert.rules.yml
```bash
groups:
  - name: example_alerts
    rules:
      - alert: InstanceDown
        expr: up{job="node"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "instance {{ $labels.instance }} dont working"
          description: "{{ $labels.instance }} job {{ $labels.job }} doesnt ask for anything 1min"
    
      - alert: HighCpuUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% for more than 5 minutes"

      - alert: LowMemory
        expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100) < 10
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low memory on {{ $labels.instance }}"
          description: "Available memory is only {{ $value }}%"
```
### Ports on HTTP
> [!INFO] ports
> Метрики Prometheus: http://<IP_Server>:9100/metrics
> Цели Prometheus: http://<IP_server>:9090/targets
> Alertmanager: http://<IP_server>:9093
> Grafana: http://<IP_server>:3000

Данные для первого входа в Grafana:
```bash
login: admin
passowrd: admin
```
После первого входа Вас попросит сменить пароль

В Grafana, жмем шестерню слева в углу:
```bash
Connections -> Data Sources -> Prometheus
Ставим наш IP и порт на котором работает Prometheus:
http://<IP_Server>:9090
[Save & test]
Должно быть успешно
```
Далее, импортируем Dashboard:
```bash
Dasboards -> New -> Import
И в Find and Import dshboards for common applications пишем:
> 1860 
жмем [Load] и затем снова [Load] но уже тот, что ниже
```