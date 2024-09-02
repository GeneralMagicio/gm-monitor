# Gm-Monitor

*A generalized monitoring stack to be installed on individual servers that we want to monitor.*

Consists of:

- cAdvisor: Exports docker container metrics for consumption by Prometheus
- Node-Exporter: Exports server system metrics for consumption by Prometheus
- Promtail: Exports application logs for consumption by Loki

## 1 - Configure and run the monitoring stack
Clone the repo
```
git clone https://github.com/GeneralMagicio/gm-monitor.git
```
Copy `.env.template` to `.env` and fill in the values

```
cp .env.template .env $$ nano .env
```

## 2 - Promtail configuration

*Promtail is the log client that will be shipping the logs from host machines to the Loki Server*

1. Modify the `.env` file to add a new volume for a persistent log directory
```
HOST_LOG_FOLDER=/home/gmadmin/myapp/logs
```
2. Modify the `.env` to reflect the folder name of the app we want to monitor
```
## The generic name of the app (The same name of the app folder when it is cloned from GH)
APP_NAME=QAcc-BE
```
## Operate

### Start
```
docker-compose up -d
```

### Stop
```
docker-compose down
```

### Restart a service
```
docker-compose up -d containername
```

## Initialize the Monitor to run on our host

**NOTE:** If the host is configured to run caddy, you will need to open the necessary ports for the monitoring stack.

Run the below Command:
```
sudo bash init.sh
```
## Further configuration
Have a look at the two config files.

1. Check `docker-compose.yml`:

- Correct network to observe other containers:
```
networks:
  external_network:
    external:
      name: giveth-all_giveth # Same name of the network that is running in the docker-compose of the app we want to monitor
```

2. Check `promtail-config.yml` if you send more than one promtail exporter to your Loki instance:
- The `tenant_id` should be unique
- The labels for `job name`and `job`should be unique
- Add more `static_configs` as you see fit (Depending on how many apps you need to monitor)

```
scrape_configs:
  - job_name: ${INSTANCE}-${APP_NAME}-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: ${INSTANCE}-${APP_NAME}-logs
          __path__: /var/app/logs/app1

  - job_name: ${INSTANCE}-${APP_NAME2}-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: ${INSTANCE}-${APP_NAME2}-logs
          __path__: /var/app/logs/app2
```
## 3 - Logger Whitelisting
In order for everything to work, we should whitelist the server in the Logger Instance

1. Get the Public IP:

```
curl ifconfig.io
```

2. Add it in the `.env` of the logger instance: (More info on this repository: https://github.com/generalmagicio/gm-logger)
```
LOKI_WHITELIST="192.168.0.0/16 172.16.0.0/12 10.0.0.0/8 ${YOUR_PIBLIC_IP}"
GRAFANA_WHITELIST="192.168.0.0/16 172.16.0.0/12 10.0.0.0/8 ${YOUR_PIBLIC_IP}"
```