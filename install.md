# Инструкция по развертыванию системы мониторинга (Prometheus, SNMP Exporter, Grafana)

Ниже представлена пошаговая инструкция со всеми командами для установки и настройки инфраструктуры мониторинга с нуля.

---

## Шаг 1. Установка и настройка Prometheus

# Переходим во временную директорию и скачиваем архив Prometheus нужной версии: ```bash cd /tmp wget 
   [https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz](https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz)

# Распаковываем скачанный архив и переходим в распакованную папку
tar xvf prometheus-2.45.0.linux-amd64.tar.gz
cd prometheus-2.45.0.linux-amd64

# Копируем основные исполняемые файлы в системный каталог и создаем необходимые директории для конфигураций и базы данных
sudo cp prometheus promtool /usr/local/bin/
sudo mkdir -p /etc/prometheus /var/lib/prometheus

# Копируем рабочий конфигурационный файл Prometheus в системную папку
sudo cp prometheus/prometheus.yml /etc/prometheus/prometheus.yml

# Создаем и открываем файл системной службы Prometheus для автозапуска
sudo nano /etc/systemd/system/prometheus.service

# Вставляем конфигурацию службы
[Unit]
Description=Prometheus
After=network.online.target

[Service]
User=root
Restart=on-failure
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

# Перезагружаем демон systemd, включаем службу в автозагрузку и запускаем её
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus

## Шаг 2. Установка и настройка SNMP Exporter (для сетевых устройств)

# Переходим во временную папку и скачиваем SNMP Exporter
cd /tmp
wget [https://github.com/prometheus/snmp_exporter/releases/download/v0.25.0/snmp_exporter-0.25.0.linux-amd64.tar.gz](https://github.com/prometheus/snmp_exporter/releases/download/v0.25.0/snmp_exporter-0.25.0.linux-amd64.tar.gz)

# Распаковываем архив и переходим внутрь
tar xvf snmp_exporter-0.25.0.linux-amd64.tar.gz
cd snmp_exporter-0.25.0.linux-amd64

# Копируем бинарный файл в системный путь, а файл конфигурации (snmp.yml) — в каталог Prometheus
sudo cp snmp_exporter /usr/local/bin/
sudo cp snmp.yml /etc/prometheus/snmp.yml

# Создаем системную службу для SNMP Exporter
sudo nano /etc/systemd/system/snmp_exporter.service

# Вставляем конфигурацию службы
[Unit]
Description=SNMP Exporter
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/snmp_exporter --config.file=/etc/prometheus/snmp.yml

[Install]
WantedBy=multi-user.target

# Перезагружаем демон и запускаем SNMP Exporter
sudo systemctl daemon-reload
sudo systemctl enable --now snmp_exporter

## Шаг 3. Установка и настройка Grafana

# Устанавливаем необходимые системные зависимости и добавляем официальный GPG-ключ репозитория Grafana
sudo apt-get update
sudo apt-get install -y software-properties-common wget apt-transport-https
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - [https://apt.grafana.com/gpg.key](https://apt.grafana.com/gpg.key) | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

# Добавляем репозиторий Grafana в список источников apt
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] [https://apt.grafana.com](https://apt.grafana.com) stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

# Обновляем списки пакетов и устанавливаем саму Grafana:
sudo apt-get update
sudo apt-get install -y grafana

# Запускаем сервер Grafana и добавляем его в автозагрузку
sudo systemctl daemon-reload
sudo systemctl enable --now grafana-server


