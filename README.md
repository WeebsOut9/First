# Docker/Docker Compose/Grafana/Prometheus

* Перед началой установки, нужно установить Linux Oracle на VirtualBox при этом нужно учесть некоторые пункты:

      Выделить 2+ ядер.
      Выделать 4096+ МБ оперативы.
      Не создавать в сетевых расположения (виртуалка не будет работать).
      При установки нужно выбрать английский язык.

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

Устанавливаем docker

             sudo yum install docker-ce docker-ce-cli containerd.io
            
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

Установка git

             sudo yum install git

Клонирование репозитория Grafana Stack для Docker с GitHub.
      
            sodo git clone https://github.com/skl256/grafana_stack_for_docker.git

Затем, удоляем файл выброном пути

            sudo rm -r grafana_stack_for_docker/grafana.yaml
            
Перетаскиваем готовый файл в нужную папку

            sudo mv Downloads/docker-compose grafana_stack_for_docker

Для проверки пишем

            ls
            
![1](https://github.com/user-attachments/assets/2f7ba9bb-e496-484b-8ec2-d5821b4d5d17)

Переминовываем файл

            sudo mv docker-compose docker-compose.yaml

И снова проверяем 

            ls

![2](https://github.com/user-attachments/assets/56dad9c4-f0ee-4963-840a-b69755790896)

Создание каталога для конфигурации Grafana внутри общей директории в Docker Swarm.

            sudo mkdir -p /mnt/common_volume/swarm/grafana/config

Создание нескольких директорий для хранения данных конфигурации и информации для Grafana, Prometheus, Loki, Promtail.

            sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}

Изменение владельца этих директорий на текущего пользователя с помощью команд id -u и id -g, которые возвращают UID и GID текущего пользователя.

            sudo chown -R (id −u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}

Создание пустого файла grafana.ini для дальнейшей конфигурации Grafana.

            sodo touch /mnt/common_volume/grafana/grafana-config/grafana.ini
            
Копирование всех файлов из локальной директории config в созданный каталог конфигурации.

            sodo cp config/* /mnt/common_volume/swarm/grafana/config/
            
Переименование файла grafana.yaml в docker-compose.yaml, который используется для запуска контейнеров Docker через Docker Compose.

            sodo mv grafana.yaml docker-compose.yaml

Запуск сервисов, описанных в docker-compose.yaml, в фоновом режиме.

            sudo docker compose up -d

остонавлием докер компос так как он не даст редактировать новые данные

            sudo docker compose stop

Затем нужно будет перейти в раздел config для этого пишем следующее cd grafana_stack_for_docker/config если вы уже находитесь в разделе grafana, то напишете следующее cd config

Открываем файл prometheus.yaml в текстовом редакторе vi с правами суперпользователя

             sudo vi prometheus.yaml

Далее нужно исправить targets: на exporter:9100

Было:

![3](https://github.com/user-attachments/assets/52b416c0-b5c2-4e9d-8c9f-89c8ee1fffc3)

Стало:

![4](https://github.com/user-attachments/assets/2e7553a7-b0a4-43bd-a58f-7f656fef7e10)

Чтобы сохранить нажимаем ESC 

            :wq!

запускаем докер

            cd
            
            cd grafana_stack_for_docker/config
            
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


      Создание дашбордов После успешного подключения Prometheus к Grafana вы можете создавать дашборды для визуализации метрик: 
      Перейдите в раздел "Dashboards" (Дашборды) -> "New Dashboard" (Новый дашборд). 
      Добавьте панель (panel) с графиками, выбрав Prometheus в качестве источника данных. 
      В поле для запроса Prometheus введите нужные метрики (например, up, node_cpu_seconds_total, и другие), чтобы вывести соответствующую информацию.


      Импорт готовых дашбордов Чтобы ускорить процесс, можно импортировать уже готовые дашборды из сообщества Grafana: 
      Перейдите в раздел "Dashboards" -> "Import" (Импорт). 
      Введите ID дашборда из официального каталога (например, ID 1860 для популярных метрик по Prometheus). 
      После импорта настройте панель для работы с вашим источником данных Prometheus


      Добавьте источник данных Prometheus Чтобы Grafana могла отображать данные из Prometheus: 
      В интерфейсе Grafana перейдите в раздел "Configuration" (Конфигурация) -> "Data Sources" (Источники данных). 
      Нажмите кнопку "Add Data Source" (Добавить источник данных). В списке источников данных выберите Prometheus. 
      В поле URL укажите адрес, по которому работает Prometheus (например: http://localhost:9090). 
      Нажмите кнопку "Save & Test" (Сохранить и протестировать), чтобы проверить подключение.


Захом в connection там где мы писали http://prometheus:9090 пишем http://victoriametrics:8428 И заменяем имя из "Prometheus-2" в "Vika" нажимаем на dashboards add visualition выбираем "Vika" снизу меняем на "code" Переходим в терминал и пишем

           echo -e "# TYPE OILCOINT_metric1 gauge\nOILCOINT_metric1 0" | curl --data-binary @- http://localhost:8428/api/v1/import/prometheus

Заменяем нолиек на любые другие цифры  

• Команда отправляет бинарные данные (например, метрики в формате Prometheus) на локальный сервер, который слушает на порту 8428.

           curl -G 'http://localhost:8428/api/v1/query' --data-urlencode 'query=OILCOINT_metric1'

И так делаем 3 раза 

Тепреь вписываеи в строку 

            http://localhost:8428

Нажимаем на 
            vmui - Web UI

Копируем переменную OILCOINT_metric1 и вставляем в query

Нажимаем run

![image](https://github.com/user-attachments/assets/a4c698bc-0717-4458-872f-67c0ec99adfe)

![image](https://github.com/user-attachments/assets/fdf929a3-5ece-4b28-84ff-9aa3272dda65)

![image](https://github.com/user-attachments/assets/9e307642-edbe-4081-87f5-cd45b31fdf6b)

![image](https://github.com/user-attachments/assets/5bb5be52-f3b8-4542-aeb6-148b9e321f71)

Копируем переменную OILCOINT_metric1 и вставляем в code

![image](https://github.com/user-attachments/assets/b0170b3a-229e-4c21-83c5-8b134723b342)

![image](https://github.com/user-attachments/assets/23b40bc7-a10c-40a4-975a-c508ea060146)

![image](https://github.com/user-attachments/assets/b6fcc295-2296-43f9-9faa-517c83f537da)

![image](https://github.com/user-attachments/assets/ab43437b-b229-4944-b7af-1fe4e5716ae8)








Переход в каталог, куда был склонирован репозиторий.

            sodo cd grafana_stack_for_docker



            


            

  




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
