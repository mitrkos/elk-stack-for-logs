## Инструкция

Все работает через docker-compose, конфиги и докерфайлы лежат в соответствующих директориях. 

Поток обрботки данных:  
Лог файлы -> logstash -> elasticsearch -> kibana  

Нужно правильно прописать input в logstash/pipeline/logstash.conf и не забыть про docker-compose.yml  

Сразу отмечу следующие моменты:  
* logstash умеет читать просто файлы .gz, но не .tar.  
* Есть проблема с logratate из-за опции copytruncate, т.к. создается новый файл и, если чтение настроено на все файлы в папке с лог-файлами, произойдет дублирование логов. Для ее решения есть 2 варианта:  
    1. Убрать эту опцию, если она была выставлена не намерено.  
    2. Вычитать сначала все существиющие логи сначала, а затем переконфигурировать logstash на чтение текущего файла для записи (настройка input pipeline logstash/pipeline/logstash.conf).  

Сначала поднимаем все с помощью docker-compose:
> docker-compose up

Сначала все сгонфигурировано с помощью пользователя с привилегированным доступом, для повышения безопасности лучше сгенерировать непривелигированных пользователей:
> docker-compose exec -T elasticsearch bin/elasticsearch-setup-passwords auto --batch

Затем удаляем строку ELASTIC_PASSWORD: changeme из docker-compose.yml.  

Прописываем новых пользователей:  
* kibana -> kibana/config/kibana.yml
* logstash_system -> logstash/config/logstash.yml
* elastic -> logstash/pipeline/logstash.conf (output)

> docker-compose restart kibana logstash

Теперь можно заходить в kibana пользователем elastic. (http://localhost:5601/) 

Заходим в настройки (шестеренка) -> Elasticsearch/Index Management - проверяем, что данные загружаются.  

Затем в Kibana/Index Patterns/Create index. Я обычно создаю 3 паттерна:
 * logs-*-processed - для хорошо обработанных логов,
 * logs-*-other - для всех остальных.
 * logs-* - для всех в одном месте.

Затем заходим в компас (слева). В этом разделе происходит основная работа с логами. 

Здесь можно выбрать в каких индексах искать (по паттерну выше), фильтровать по дате, делать запросы с фильтрами, выбирать отображаемые поля (там есть с переводом).  
Запросы вида:
> module:ReM AND log_level:DEBUG AND NOT (method:one.vmpool.info OR method:one.zone.raftstatus)

Также можно заглянуть в визуализацию данных по логам.

В целом все.
