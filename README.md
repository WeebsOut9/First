# Docker/Docker Compose/Grafana/Prometheus

* Перед началой установки, нужно установить Linux Oracle на VirtualBox при этом нужно учесть некоторые пункты:

      Выделить 2+ ядер.
      Выделать 4096+ МБ оперативы.
      Не создавать в сетевых расположения (виртуалка не будет работать).
      При установки нужно выбрать английский язык.

Клонирование репозитория Grafana Stack для Docker с GitHub.
      
            git clone https://github.com/skl256/grafana_stack_for_docker.git

Переход в каталог, куда был склонирован репозиторий.

            cd grafana_stack_for_docker

Создание каталога для конфигурации Grafana внутри общей директории в Docker Swarm.

            sudo mkdir -p /mnt/common_volume/swarm/grafana/config

Создание нескольких директорий для хранения данных конфигурации и информации для Grafana, Prometheus, Loki, Promtail.

            sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}

Изменение владельца этих директорий на текущего пользователя с помощью команд id -u и id -g, которые возвращают UID и GID текущего пользователя.

            sudo chown -R (id −u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}

Создание пустого файла grafana.ini для дальнейшей конфигурации Grafana.

            touch /mnt/common_volume/grafana/grafana-config/grafana.ini

Копирование всех файлов из локальной директории config в созданный каталог конфигурации.

            cp config/* /mnt/common_volume/swarm/grafana/config/

Переименование файла grafana.yaml в docker-compose.yaml, который используется для запуска контейнеров Docker через Docker Compose.

            mv grafana.yaml docker-compose.yaml

Установка wget для скачивания файлов.

            sudo yum install wget
            
В случае возникновения такой ошибки:
  
  ![1](https://github.com/user-attachments/assets/fe17df20-0281-49c7-a890-8d0067462568)
  
  решение:

            Пишем:
                   su root 
                   nano /etc/sudoers

              В появившемся окне под 
 
                   root ALL=(ALL)  ALL 
 
              Пишем 
 
                   user_name ALL=(ALL)  ALL
  
              Затем жмем Ctrl+X, подтверждаем на Y+Enter.
  
Скачивание файла репозитория Docker CE для CentOS и размещение его в директории /etc/yum.repos.d/. (везде жмем Y)

            sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo

Включение и немедленный запуск службы Docker.
            
            sudo systemctl enable docker --now

Установка утилиты curl (на CentOS/RHEL) для выполнения HTTP-запросов.
            
            sudo yum install curl

Использование команды curl для получения последней версии Docker Compose с GitHub API. Фильтрация ответа для извлечения тега с версией.
            
            COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d" -f4)

Загрузка последней версии Docker Compose с официального репозитория GitHub и сохранение её в /usr/bin/docker-compose.
            
            sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose

Предоставление прав на выполнение файла docker-compose.

            sudo chmod +x /usr/bin/docker-compose

Проверка установленной версии Docker Compose.

            docker-compose --version

Самое важное, по пути: /mnt/common_volume/swarm/grafana/config в файле prometheus.ini через vi нужно добавить вот эти строки, так это нужно будет для дальнейшей работы:

      node-exporter: 
      image: prom/node-exporter 
      volumes: 
        - /proc:/host/proc:ro 
        - /sys:/host/sys:ro 
        - /:/rootfs:ro 
      container_name: exporter 
      hostname: exporter 
      command: 
        - --path.procfs=/host/proc 
        - --path.sysfs=/host/sys 
        - --collector.filesystem.ignored-mount-points 
        - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/) 
      ports: 
        - 9100:9100 
      restart: unless-stopped 
      environment: 
        TZ: "Europe/Moscow" 
      networks: 
        - default

Запуск сервисов, описанных в docker-compose.yaml, в фоновом режиме.

            sudo docker compose up -d

После всех действий перечисленных выше пишем в поисковую строку вашего браузера "http://localhost:3000"

Пороль: 
      
            admin
            
Логин: 
      
            admin

Код графаны: 3000

Код промитеуса: 9090














      

      

      

      

      

      

  
      


* После всех действий и запуска докера, устанавливаем Prometheus в Grafana. 2. Установка Prometheus в Grafana

Prometheus и Grafana часто используются вместе для мониторинга и визуализации данных. Prometheus собирает метрики, а Grafana визуализирует их в виде удобных дашбордов. 

Ниже приведены основные шаги для интеграции Prometheus в Grafana:

    Установите и настройте Prometheus Перед тем как интегрировать Prometheus в Grafana, необходимо убедиться, что Prometheus установлен и настроен для сбора данных. Если Prometheus уже настроен и работает, убедитесь, что он собирает метрики с нужных источников (экспортеров).

    Запустите Grafana и откройте интерфейс После установки и запуска Grafana перейдите в веб-интерфейс по умолчанию (обычно это порт 3000 на локальном хосте, например: http://localhost:3000). Войдите в систему с помощью стандартных учетных данных (логин и пароль по умолчанию — admin/admin).

    Добавьте источник данных Prometheus Чтобы Grafana могла отображать данные из Prometheus: В интерфейсе Grafana перейдите в раздел "Configuration" (Конфигурация) -> "Data Sources" (Источники данных). Нажмите кнопку "Add Data Source" (Добавить источник данных). В списке источников данных выберите Prometheus. В поле URL укажите адрес, по которому работает Prometheus (например: http://localhost:9090). Нажмите кнопку "Save & Test" (Сохранить и протестировать), чтобы проверить подключение.

    Создание дашбордов После успешного подключения Prometheus к Grafana вы можете создавать дашборды для визуализации метрик: Перейдите в раздел "Dashboards" (Дашборды) -> "New Dashboard" (Новый дашборд). Добавьте панель (panel) с графиками, выбрав Prometheus в качестве источника данных. В поле для запроса Prometheus введите нужные метрики (например, up, node_cpu_seconds_total, и другие), чтобы вывести соответствующую информацию.

    Импорт готовых дашбордов Чтобы ускорить процесс, можно импортировать уже готовые дашборды из сообщества Grafana: Перейдите в раздел "Dashboards" -> "Import" (Импорт). Введите ID дашборда из официального каталога (например, ID 1860 для популярных метрик по Prometheus). После импорта настройте панель для работы с вашим источником данных Prometheus
