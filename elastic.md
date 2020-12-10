# ElasticSearch

## ELK

### ElasticSearch

a search and analytics engine;

масштабируемый полнотекстовый поисковый и аналитический движок с открытым исходным кодом. Он позволяет хранить большие объемы данных,проводить среди них быстрый поиск и аналитику \(почти в режиме реального времени\). Как правило, он используется в качестве базового механизма/технологии, которая обеспечивает работу приложений со сложными функциями и требованиями к поиску. Elasticsearch использует в качестве поисковой основы библиотеку Lucene, которая написана на языке Java и доступна для многих платформ. Все неструктурированные данные хранятся в формате JSON, что автоматически делает ES базой данных NoSQL. в отличие от других баз данных NoSQL, ES предоставляет возможности поиска и многие другие функции.

1. Масштабируемость и отказоустойчивость — Elasticsearch легко масштабируется. К уже имеющейся системе можно на ходу добавлять новые серверы, и поисковый движок сможет сам распределить на них нагрузку. При этом данные будут распределены таким образом, что при отказе какой-то из нод они не будут утеряны и сама поисковая система продолжит работу без сбоев.
2. Мультиарендность \(англ. multitenancy\) — возможность организовать несколько различных поисковых систем в рамках одного объекта Elasticsearch. Организацию можно проводить динамически.
3. Операционная стабильность — на каждое изменение данных в хранилище ведется логирование сразу на нескольких ячейках кластера для повышения отказоустойчивости и сохранности данных в случае разного рода сбоев.
4. Отсутствие схемы \(англ. schema-free\) — Elasticsearch позволяет загружать в него обычный JSON-объект. Далее индексация и добавление в базу поиска производится автоматически. Таким образом, обеспечивается быстрое прототипирование.
5. RESTful API — Elasticsearch практически полностью управляется по HTTP с помощью запросов в формате JSON.

#### Cluster

is a group of nodes \(servers\), that stores your data and provide capabilities for searching, indexing, retrieving it. Important property of the cluster is it’s **name**. It’s crucial, because node could be part of the cluster, if it’s configured to join the cluster by it’s name. Usual mistake is to forget changing it, so your node instead of joining your `elasticsearch-prod` cluster will try to join `elasticsearch-dev` and vise versa

Don't use same names in different envs, or node can join wrong cluster.

#### Node

* each instance of ES is on separate node.
* is single machine, capable of joining the cluster. Node is able to participate in indexing and searching process. 
* It is identified by UUID \(Universally Unique Identifier\), that is assigned to the node on startup. 
* Node is capable of identifying the other nodes of the cluster via unicast.

Unicast discovery configures a static list of hosts for use as seed nodes. These hosts can be specified as hostnames or IP addresses; hosts specified as hostnames are resolved to IP addresses during each round of pinging. Here is an example of a unicast configuration inside the Elasticsearch configuration file \(elasticsearch.yml\):

```text
discovery.zen.ping.unicast.hosts: ["10.0.0.3:9300", "10.0.0.4:9300", "10.0.0.5:9300"]
```

Not all of the Elasticsearch nodes in the cluster need to be present in the unicast list to discover all the nodes, but enough addresses should be configured for each node to know about an available gossip node.

**Node types**

all nodes know about each other.

* Master-eligible node This node could be selected as a master of a cluster, so it will be in control. By default, every new node is a master-eligible node. Master node is responsible for lightweight operations like creation/deletion of the index. It’s crucial for cluster health and stability.
* Data node This type of nodes will store Elasticsearch index and could perform operations as CRUD, search, aggregations. By default, every new node is a data node
* Ingest node This type of node, that is participating in the ingest pipelines \(e.g. enriching documents before indexing\). By default, every new node is an ingest node
* Tribe node Special type of coordinating node, which is able to connect to several clusters and perform searches across all connected clusters. Disabled for every new node by default

In a big cluster you need to separate these functions, especially master from data.

#### index

* is collection of indexed documents. 
* Index is identified by it’s name \(in lowercase\) and this name should be used to refer to different operations \(indexing, searching and others\) that should be executed against this index. 

#### Document

* is a basic unit, that Elasticsearch manipulates. 
* You could index the document, to be able to search it later.
* Each document consists of several fields and is expressed in a JSON format. 

**Indexing of a document = analysys**

It consists of 3 steps:

* normalizing: split by tokens, lowercase
* processing: filter stop words
* enriching: add synonims

Можно провести условную аналогию: индекс — это база данных, а тип — таблица в этой БД. Каждый тип имеет свою схему — mapping, также как и реляционная таблица. Mapping генерирутся автоматически при индексации документа.

#### Shard

* subdivision of index
* you can define number of shard when creating index
* it's itself a fully functional index that can be placed on any node of a cluster
* 2 resons:
* it allows horizontal split your data volume
* allows to distribute and \|\| operations across shards in order to perform it faster and increase throughput \(производительность, пропускная способность\).

#### Replica

* one or more copies of index’s shards,
* provides ability to tolerate node/shard failures, cluster still could route the query to the replica in order to return results.
* replica should never been allocated on the same node, where the original shard is.

Ключ `_version` показывает версию документа. Он нужен для работы механизма оптимистических блокировок. Например, мы хотим изменить документ, имеющий версию 1. Мы отправляем измененный документ и указываем, что это правка документа с версией 1. Если кто-то другой тоже редактировал документ с версией 1 и отправил изменения раньше нас, то ES не примет наши изменения, т.к. он хранит документ с версией 2.

ES не делает различий между одиночным значением и массивом значений. Например, поле title содержит просто заголовок, а поле tags — массив строк, хотя они представлены в mapping одинаково.

ES не использует значение `_source` для поисковых операций, т.к. для поиска используются индексы. Для экономии места ES хранит сжатый исходный документ. Если нам нужен только id, а не весь исходный документ, то можно отключить хранение исходника.

size ограничивает кол-во документов в выдаче. total показывает общее число документов, подходящих под запрос. sort в выдаче содержит массив целых чисел, по которым производится сортировка.

ES с версии 2 не различает фильтры и запросы, вместо этого вводится понятие контекстов. Контекст запроса отличается от контекста фильтра тем, что запрос генерирует `_score` и не кэшируется. Поле `_score` показывает релевантность. Если запрос выпоняется в filter context, то значение `_score` всегда будет равно 1, что означает полное соответствие фильтру.

Анализаторы нужны, чтобы преобразовать исходный текст в набор токенов. Анализаторы состоят из одного Tokenizer и нескольких необязательных TokenFilters. Tokenizer может предшествовать нескольким CharFilters. Tokenizer разбивают исходную строку на токены, например, по пробелам и символам пунктуации. TokenFilter может изменять токены, удалять или добавлять новые, например, оставлять только основу слова, убирать предлоги, добавлять синонимы. CharFilter — изменяет исходную строку целиком, например, вырезает html теги.

В ES есть несколько стандартных анализаторов. Например, анализатор russian.

`_id` – the document identifier, it is not a part of the data response `_score` – the field with the score \(where it comes from is a whole lecture in itself\) `_source` – contains the document data as it was uploaded

When adding or updating the data in Elasticsearch two processes are running in the background: the periodic refresh and flush operations.

* Refresh – ensures that changes are written to transaction log.
* Flush - ensures the transaction logs are empty and all changes are persisted in the index.

#### Available types of update:

* Merge – merge the given document with the existing document.
* Script – execute the given script on existing document.
* Upsert – index if not exists. If exists either execute script or merge.

Extra flags: scripted\_upsert doc\_as\_upsert

What Elasticsearch actually does:

* Delete the old document
* Index the updated document

#### Mapping

1. It defines how Elasticsearch should treat your data. You can define the types of fields you store in the document, they ways the data is indexed and stored.
2. Creates itself automagically and works for the simple use cases. 
3. Access via REST API or Java Client

   `GET /<index>/_mapping`

4. Mapping cannot be changed when the data is already indexed. The only way to do so is to reindex the data to new index with changed mapping.
5. If you provide no mapping when creating an index it will be generated automatically for the incoming data. This feature is called dynamic mapping.
6. t might look like it but internally Elasticsearch does not work on data with nesting in it. What it does is a field based lookup – so the closest analogy is actually a Key-Value store with the ability to understand that a group of fields form a document.
7. Given this fact the nested object JSON will be translated to field-based description by path tracing.
8. While a simple nested object is not problematic the object array will be problematic. The paths generated will have array offsets in them – so unless the order of objects in the data is repeatable it will be very hard to target a specific generate field name when you specify the query. The query targetted at dynamically-mapped substructure with array of objects will not distinguish which nested object belongs to.

#### Text type

The text type is a full-text searchable datatype in Elasticsearch

When left with default settings it will \(not a complete list\):

* Automatically tokenize the text using Unicode Text Segmentation algorithm
* Automatically lower-case all tokens
* Generate two fields in Elasticsearch – the one with the field name equal to one found in JSON, and the second one with .keyword postfix.

**Term** – the tokenized piece of text associated with the field name it belongs to e.g.: product:laptop \(the “product” is the field name, the “laptop” is the value\).

```text
{
  "text": "quick brown fox"
}
```

terms:

```text
text: quick
text: brown
text: fox
```

**Term query**

Matches the document based on a exact match of the provided value to the value in given document property. Works also on other data types.

```text
{
    "query": {
        "term": {
            "text": "fox"
        }
    }
}
```

**Range query**

```text
{
    "query": {
        "range": {
            "regularPrice": {
                "gt": 200,
                "lte": 300
            }
        }
    }
}
```

**Prefix query**

**Regexp query**

**Bool Query**

```text
{
    "bool": {
        "must": [],
        "filter": [],
        "must_not": [],
        "should": []
    }
}
```

### Logstash:

a server‑side data processing pipeline that ingests data from multiple sources simultaneously, transforms it, and then sends it to a "stash" like Elasticsearch

is an open source data collection engine with real-time pipelining capabilities. Logstash can dynamically unify data from disparate sources and normalize the data into destinations of your choice.

Logstash — это механизм сбора данных с открытым исходным кодом с возможностями конвейерной обработки данных в реальном времени.Logstash может динамически идентифицировать данные из различных источников и нормализовать их, с помощью выбранных фильтров.

Logstash — это сервер с открытым исходным кодом, который обрабатывает данные из множества источников одновременно, преобразует их, а затем отправляет в «stash» \(это Elasticsearch\). Простыми словами, Logstash — это инструмент для нормализации, фильтрации и сбора логов.

Logstash поддерживает множество входных типов данных — это могут быть журналы, метрики разных сервисов и служб. При получении он их структурирует, фильтрует, анализирует, идентифицирует информацию \(например, геокоординаты IP-адреса\), упрощая тем самым последующий анализ.

Cбор данных из различных источников с помощью logstash Конфигурационные файлы составляются в формате JSON. Типичная конфигурация logstash представляет из себя несколько входящих потоков информации \(input\), несколько фильтров для этой информации \(filter\) и несколько исходящих потоков \(output\). Выглядит это как один конфигурационный файл, который в простейшем варианте выглядит вот так:

```text
input {
}
filter {
}
output {
}
```

Конвейер Logstash имеет два обязательных элемента: input и output. И необязательный — filter.

#### Input

Данный метод является входной точкой для логов. В нём определяет по каким каналам будут логи попадать в Logstash.

Основные типы input:

* file
* tcp
* udp

#### filter

В данном блоке настраиваются основные манипуляции с логами. Это может быть и разбивка по key=value, и удаление ненужных параметров, и замена имеющихся значений, и использование DNS запросов для ip-адресов или названий хостов.

Нормализация логов Пример конфигурационного файла для основной нормализации логов:

```text
filter {
  grok {
    type => "some_access_log"
    patterns_dir => "/path/to/patterns/"
    pattern => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"
  }
}
```

Построчное описание:

* type =&gt; "apache\_access" - тип/описание лога.Здесь надо указать тот тип \(type\), который прописан в блоке input для которого будет происходить обработка;
* patterns\_dir =&gt; "/path/to/patterns/" - путь к каталогу, содержащим шаблоны обработки логов. Все файлы находящиеся в указанной папке будут загружены Logstash, так что лишние файлы там не желательны;
* pattern =&gt; "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" - указывается шаблон разборки логов. Шаблон можно использовать либо в конфигурационном файле, либо из файла шаблонов. 

**Про шаблоны**

С помощью grok фильтра можно структурировать большую часть логов — syslog, apache, nginx, mysql итд, записанных в определённом формате.

Logstash имеет более 120 шаблонов готовых регулярных выражений \(regex\). Так что написание фильтров для обработки большинства логов не должно вызвать особого страха или недопонимания. Формат шаблонов относительно простой - NAME PATTERN, то есть построчно указывается имя шаблона и ему соответствующее регулярное выражение.

#### output

В этом блоке указываются настройки для исходящих сообщений. Аналогично предыдущим блокам, здесь можно указывать любое количество исходящих подблоков.

* stdout
* file
* elasticsearch

```text
output {
  elasticsearch {
    type => "custom_log"
    cluster => "es_logs"
    embedded => false
    host => "192.168.1.1"
    port => "19300"
    index => "logs-%{+YYYY.MM.dd}" 
  }
}
```

* cluster =&gt; "es\_logs" - название кластера указанного в cluster.name в конфигурационном файле Elasticsearch;
* embedded =&gt; false - указывает какую базу Elasticsearch использовать внутреннюю или стороннюю;
* port =&gt; "19300" -транспортный port Elasticsearch;
* host =&gt; "192.168.1.1" - IP-адрес Elasticsearch;
* index =&gt; "logs-%{+YYYY.MM.dd}" " - название индекса куда будут записываться логи.
* e-mail alert

### Kibana:

lets users visualize data with charts and graphs in Elasticsearch.

Kibana - платформа для анализа и визуализации с открытым исходным кодом, предназначенная для работы с Elasticsearch. Kibana можно использовать для поиска, просмотра и взаимодействия с данными, хранящимися в индексах Elasticsearch. Вы можете легко выполнять расширенный анализ данных и визуализировать результаты в различных диаграммах, таблицах и картах.

Kibana упрощает понимание больших объемов данных. Его простой, браузерный интерфейс позволяет быстро создавать и обмениваться динамическими панелями мониторинга, отображающими изменения в запросах Elasticity в режиме реального времени.

#### Dashboard

* a collection of vizualisations and searches
* can be shared

#### Types of search:

* For full text search, just type a string that you want to search for. The search will be executed against all available fields, i.e cancer. In this case you will see all documents containing this term
* If you want to search for a phrase - several terms next to each other, you need to embrace them with double quotes, i.e “heart attack”
* If you want to search in a particular field, you could write it as following – httpCode:200
* If you want to search for a range of values, you could write them in a format of field:\[START TO END\]
* For more complex queries you could use Boolean operators \(AND, OR, NOT\), i.e httpCode:200 AND “heart attack” 

### Beats

* are open source data shippers that you install as agents on your servers to send different types of operational data to Elasticsearch. 
* Beats can send data directly to Elasticsearch or send it to Elasticsearch via Logstash, which you can use to parse and transform the data.

Elastic Beats - набор программ-коллекторов данных с низкими требованиями к ресурсам, которые устанавливаются на клиентских устройствах для сбора системных журналов и файлов. Имеется широкий выбор коллекторов, а также возможность написать свой коллектор.

#### Filebeat

транслирует на сервер информацию из динамических обновляемых журналов системы и файлов, содержащих текстовую информацию. Для аналогичных действий с журналами Windows-систем используется Winlogbeat.

#### Packetbeat

сетевой анализатор пакетов, который отправляет информацию о сетевой активности между серверами приложений. Он перехватывает сетевой трафик, декодирует протоколы и извлекает необходимые данные.

#### Metricbeat

собирает метрики операционных систем, характеризующие, например, использование CPU и памяти, количество переданных пакетов в сети, состояние системы и сервисов, запущенных на сервере.

#### Auditbeat

собирает данные инфраструктуры аудита Linux и проверяет целостность файлов. Auditbeat передает эти события в реальном времени стеку ELK \(Elasticsearch, Logstash и Kibana\) для дальнейшего анализа.

#### Heartbeat

предназначен для мониторинга доступности сервисов в реальном времени. Используя список URL-адресов, Heartbeat проверяет их доступность и время отклика и пердает эти данные стеку ELK.

## Installation

* need java 8 and higher
* edit elasticsearch.yml
* cluster.name
* path.data
* path.logs
* network.host
* htttp:port
* -Xms and -Xmx both = 50% of RAM to cache properly
* edit kibana.yml

There are several things that are recommended to tweak OS, which is running your installation of Elasticsearich. Let’s go through them: 1. Modern OS uses swaps as a method of saving RAM and provide some sort of operating for machines with lower RAM size, however it could affect stability of the node and most importantly performance if the JVM heap will be swapped out to the disk. We would need to disable it completely. On Linux you could do it temporarily by doing so `sudo swapoff –a`. For proper disabling, you need to edit your /etc/fstab and remove swap related lines 2. Elasticsearch uses a lot of file handles and descriptors during it’s work. Make sure to increase the size of it to more than 200k, by doing something like this: `ulimit –n 200000` \(this will only change it for the current user session, you want to take a look into `/etc/security/limits.conf` for a proper changing of this parameter\)

