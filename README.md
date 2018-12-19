# Run Simulation Kit

### Software Requirement

Убедитесь, что в системе установлена система контроля версий Git.

Так как Drone Employee архитектурно выполнен в микросервисной архитектуре на базе Docker и использует последнего для развертывания, нужно установить следующее программное обеспечение:

**Docker Community Edition** — основной пакет для работы с Docker.

**Docker Compose** — инструмент для запуска многоконтейнерных приложений Docker.

Для более удобного управления Docker убедитесь, что он установлен как non-root пользователь.

Для правильной работы Drone Employee для Docker должен быть обеспечен полный доступ к Docker Hub — службе, позволяющей ссылаться на репозитории, собирать образы и тестировать их. Для этого выполните:

```
$ docker login

```

Далее необходимо убедится, что в вашей системе установлены компоненты:

**Parity** - Приложение-клиент для распределенного компьютера Ethereum, написанное на Rust. Parity необходим для выполнения транзакций в сеть Ethereum, с помощью которых осуществляется коммуникация с дронами.

**IPFS** - InterPlanetary File System — одноранговый протокол связи. С его помощью организована передача объемных файлов с какой-либо информацией, собираемой дронами (данные о замерах, видеоданные и.т.д.)

Если данные компоненты не установлены приведем ниже ссылки на документации по каждому из вышеуказанных продуктов. Так же описаны предварительные настройки программных продуктов для корректной работы с DroneEmployee FrameWork.

### Parity

#### Инструкция по установки.

Также данное ПО можно установить следующей командой:

```bash
bash <(curl https://get.parity.io -L) -r stable
```

Для того что бы кошелек корректно работало с **DroneEmployee FrameWork** необходимо сделать следующий набор действий:

1. Создать аккаунт кошелька внутри приложения `parity`:

```
parity account add
```

2. Разблокировать созданный аккаунт создав конфигурационный файл `config.coml` в директории `/home/$USER/.local/share/io.etherium.parity/`  и пароль от аккаунта сохранить в отдельный файл `password.file` в директории `/home/$USER/`

Пример файла config.coml:
```coml
[parity]
mode = "last"
​
[network]
port = 30303
min_peers = 50
max_peers = 100
​
[websockets]
disable = false
port = 8546
interface = "local"
origins = ["*"]
apis = ["all"]
hosts = ["none"]
​
[rpc]
interface = "all"
apis = ["all"]
hosts = ["all"]
cors = ["*"]
​
[account]
unlock = ["ваш аккаунт"] можно узать при помощи комманды parity account list
password = ["/home/ваш пользователь/password.file"]
​
[footprint]
tracing = "off"
pruning = "fast"
pruning_history = 64
pruning_memory = 32
cache_size_db = 1024
cache_size = 4096
fast_and_loose = true
db_compaction = "ssd"
fat_db = "off"
scale_verifiers = true
num_verifiers = 6
```
Данная конфигурация позволяет оптимально быстро синхронизировать кошелек parity. 

### Install

Откройте консоль и скопируйте с Git репозиторий фреймворка в удобный для вас каталог:

```bash
$ cd ~/
$ git clone https://github.com/muchosun/DroneEmployeeDocker.git
$ cd ./DroneEmployeeDocker
```
В корневом каталоге загруженного репозитория располагается конфигурационный файл Docker Compose в формате .yaml. 

Рассмотрим подродно его структуру.

### Services

**Сервисы** — это понятие в иерархии распределенных приложений Docker: они представляют собой образы отдельных приложений внутри распределенного приложения. Фреймворк Drone Employee для организации своей работы использует приложения open-source проектов, представленные ниже.

Конфигурация сервисов в файле `docker-compose.yaml` как минимум включает в себя:

`image:` — указание на образ добавляемого в контейнер приложения;

`networks:` — идентификатор сети, ее адрес;

`command:` — внутренние параметры запуска для приложения;

Сервисы уже настроены для работы с фреймворком, поэтому изменение их конфигурации не требуется.

### Simserver
Этот сервис запускает сервер, на котором развернуто симуляционное программное обеспечением WebSim. Оно используется для выполнения тестовых полетов дронов разных моделей над различной территорией, заданной в симуляции.

### Robonomics
Инфраструктура для Ethereum, позволяющая интегрировать в распределенную сеть различные кибер-физические системы для Smart City и Industry 4.0. Это ПО организует связь между дронами и всей остальной сетью.

### Drone-proxy
Контейнер содержащий в себе сервис по обеспечению p2p связи между БПЛА и наземной станицей. В данном контейнере за основу взято готовое решение hamachi, которое позволяет добиться максимального качества при деградации соединения.

В данном разделе можно указать логин и пароль от вашей заранее созданной сети Hamachi. Как это сделать будет показано далее.

### Drone-proxy-impl
Контейнер содержащий в себе коммуникационный сервис для организации связи стека ROS и физического БПЛА для передачи команд автоматизированного управления.  Использует контейнер drone-proxy для коммуникации с БПЛА.

### Volumes
Секция подключаемых образов файловой системы для вышеописанных сервисов.

