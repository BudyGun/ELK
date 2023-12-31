# Домашнее задание к занятию «ELK» - Чумаков Константин

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

Желаем успехов в выполнении домашнего задания.

---

## Дополнительные ресурсы

При выполнении задания используйте дополнительные ресурсы:
- [docker-compose elasticsearch + kibana](11-03/docker-compose.yaml);
- [поднимаем elk в docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docker.html);
- [поднимаем elk в docker с filebeat и docker-логами](https://www.sarulabs.com/post/5/2019-08-12/sending-docker-logs-to-elasticsearch-and-kibana-with-filebeat.html);
- [конфигурируем logstash](https://www.elastic.co/guide/en/logstash/7.17/configuration.html);
- [плагины filter для logstash](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html);
- [конфигурируем filebeat](https://www.elastic.co/guide/en/beats/libbeat/5.3/config-file-format.html);
- [привязываем индексы из elastic в kibana](https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html);
- [как просматривать логи в kibana](https://www.elastic.co/guide/en/kibana/current/discover.html);
- [решение ошибки increase vm.max_map_count elasticsearch](https://stackoverflow.com/questions/42889241/how-to-increase-vm-max-map-count).

**Примечание**: если у вас недоступны официальные образы, можете найти альтернативные варианты в DockerHub, например, [такой](https://hub.docker.com/layers/bitnami/elasticsearch/7.17.13/images/sha256-8084adf6fa1cf24368337d7f62292081db721f4f05dcb01561a7c7e66806cc41?context=explore).

### Задание 1. Elasticsearch 

Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный. 

*Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name*.


#### Решение  


Устанавливаю зависимости:
```
apt update && apt install gnupg apt-transport-https
```
Добавляю gpg-ключ:
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```
Добавляю репозиторий в apt с яндекс зеркала:
```
echo "deb [trusted=yes] https://mirror.yandex.ru/mirrors/elastic/7/ stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
```
Обнавляю пакеты:
```
sudo apt update
```
Устанавливаем elastic:
```
sudo apt install elasticsearch
```
Обновляю конфиги systemd:
```
sudo systemctl daemon-reload
```
Включаем юнит в автозагрузку:
```
sudo systemctl enable elasticsearch.service
```
Запускаю сервис:
```
systemctl start elasticsearch.service
```

Далее меняю имя сервиса в файле elasticsearch.yml , раскомментирую строчку и меняю имя на своё - cluster.name: myhome-cluster

```
sudo nano /etc/elasticsearch/elasticsearch.yml
```
Проверяю что сервер запустился командой:
```
curl 'localhost:9200/_cluster/health?pretty'
```

![1](https://github.com/BudyGun/ELK/blob/main/img/elk1.png)

### Задание 2. Kibana

Установите и запустите Kibana.

*Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty*.

#### Решение  
Установка кибаны:
```
sudo apt install kibana
```
Обновляем конфиги systemd
```
sudo systemctl daemon-reload
```
включаем юнит
```
sudo systemctl enable kibana.service
```
Запускаем сервис:
```
sudo systemctl start kibana.service
```
Настройки в файле /etc/kibana/kibana.yml:
```
sudo nano /etc/kibana/kibana.yml
```
Открываем интерфейс в мир
```
server.host: "0.0.0.0"
```
Рестартуем сервис:
```
sudo systemctl restart kibana
```
На странице http://127.0.0.1:5601/app/dev_tools#/console, в консоли выполняю запрос GET /_cluster/health?pretty
```
GET /_cluster/health?pretty
```

![1](https://github.com/BudyGun/ELK/blob/main/img/elk2.png)
---

### Задание 3. Logstash

Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch. 

*Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.*

#### Решение  

Устанавливаю nginx и logstash из стандартного репозитория c помощью команд sudo apt install...  
Настраиваю конфигурационный файл nginx.conf  
Комментирую сущестующую строку #accesslog....  
Добавляю запись в конфигурационном файле:
```
access_log syslog:server=127.0.0.1:6000;
```
Тем самым настраиваю nginx на передачу access-логов в мой logstash сервер (127.0.0.1) по 6000-му порту.
![1](https://github.com/BudyGun/ELK/blob/main/img/elk3.png)

Создаю конфигурационный файл logstash.conf на сервере по пути:
```
/etc/logstash/conf.d/logstash.conf
```
с содержимым, указывая в input порт, который принимает от nginx логи, а в output сервер эластик и порт, на который отправляются логи с созданным индексом nginx-index:  
```
input {
  syslog {
    port => 6000
    tags => "nginx"
  }
}

output {
  elasticsearch {
    hosts => ["http://127.0.0.1:9200"]
    index => "nginx-index"
  }
}
```
![1](https://github.com/BudyGun/ELK/blob/main/img/elk4.png)

Перезапускаю все сервисы - эластик, кибана, логстэш, нджинкс.  
Захожу в кибану через вебинтерфейс. Раздел Stack Manadjment - Index Patterns. Вижу созданный индекс, добавляю его.  

Делаю запись в accesslog сервера nginx - обращаюсь через curl  к nginx к несуществующим страницам, чтобы остались логи:  
```
curl localhost/nginx
```
```
curl localhost/nginx2
```
![1](https://github.com/BudyGun/ELK/blob/main/img/elk5.png)

Звхожу в кибану, открываю Discovery, вижу что логи появились:

![1](https://github.com/BudyGun/ELK/blob/main/img/elk6.png)


---

### Задание 4. Filebeat. 

Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat. 

*Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.*  

#### Решение  
Устанавливаю Filebeat стандартными командами из репозитория:  

```
sudo apt install filebeat
```
Обновляем конфиги systemd  
```
sudo systemctl daemon-reload
```
Включаю юнит в автозагрузку  
```
sudo systemctl enable filebeat.service
```

Меняю конфиг filebeat в файле 
```
/etc/filebeat/filebeat.yml
```

```
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
.......
output.logstash
  hosts: ["127.0.0.1:5044"]
```
меняю конфиг логстэш:
```
input {
  beats {
    port => 5044
  }
}
```

Меняю конфиг nginx, закоментирую строку передачи лога в logstash и раскомментирую строку сбора логов  

![1](https://github.com/BudyGun/ELK/blob/main/img/elk7.png)

Перезапускаю все сервисы.  
Захожу в кибану, добавляю индексы аналогично предыдущему заданию, только для filebeat.  
Делаю запрос через curl к nginx.

Вижу в дискавери логи.  
![1](https://github.com/BudyGun/ELK/blob/main/img/elk8.png)



## Дополнительные задания (со звёздочкой*)
Эти задания дополнительные, то есть не обязательные к выполнению, и никак не повлияют на получение вами зачёта по этому домашнему заданию. Вы можете их выполнить, если хотите глубже шире разобраться в материале.

### Задание 5*. Доставка данных 

Настройте поставку лога в Elasticsearch через Logstash и Filebeat любого другого сервиса , но не Nginx. 
Для этого лог должен писаться на файловую систему, Logstash должен корректно его распарсить и разложить на поля. 

*Приведите скриншот интерфейса Kibana, на котором будет виден этот лог и напишите лог какого приложения отправляется.*
