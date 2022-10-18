## Grafana + loki logging stack

Стек docker-compose для упрощённого сбора и просмотра логов контейнеров с grafana loki.

### Installing

1. Скопировать [.env.example](/.env.example) в `.env`

#### With promtail

1. Добавить новый лейбл к контейнеру, чьи логи необходимо отсылать в loki

	Например, достаточно просто добавить новый лейбл в `docker-compose`-файл искомого стека:
	```yaml
	services:
	  my-service:
	    labels:
	      - "loki.log=true"
	```

2. Запустить весь стек `docker-compose`

	```shell
	docker compose up -d
	```

#### With docker plugin

1. Установить [лог-драйвер grafana loki для docker](https://grafana.com/docs/loki/latest/clients/docker-driver/)
   
    ```shell
    docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
    ```

   По факту, данный драйвер работает аналогично стандартному драйверу `json-file`, который позволяет просматривать логи стандартной командой `docker logs`, но при этом дополнительно пытается отправить логи в инстанс `loki`.

2. Активировать лог-драйвер для контейнера

	К примеру, можно определить данный конфиг в искомом `docker-compose`-стеке:
	```yaml
	x-loki-log-config: &loki-log-config
	  logging:
 	    driver: loki
        options:
          loki-url: "http://loki.docker.localhost/loki/api/v1/push"
          loki-retries: 5
          loki-batch-size: 400
          max-size: "50m"
          max-file: "10"
    ```

	а затем просто подключить к конкретному контейнеру:

	```yaml
	services:
	  my-service:
	  	<<: *loki-log-config
	```

3. Убрать профиль `promtail` из значения `COMPOSE_PROFILES` в файле [.env](/.env)
4. Запустить стек `grafana` + `loki`

	```shell
	docker compose up -d
	```

### Difference between driver and promtail

Использование драйвера - [рекомендованный способ доставки логов в loki](https://grafana.com/docs/loki/latest/clients/promtail/configuration/#example-docker-config). К тому же, он имеет очень много предопределённых лейблов для фильтрации контейнеров в `grafana`. Но его необходимо вручную ставить на всех машинах, на которых необходимо настроить подобный сбор логов.
