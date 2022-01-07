### Learning Prometheus <br>

Prometheus docker 

>docker run -p 9191:9090 -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
                               
Setting up exporters

>sudo nano /opt/prometheus/prometheus.yml    
                               
```
global:
 scrape_interval: 15s
 
rule_files:
- 'alerts.yml'

alerting:
 alertmanagers:
 - static_configs: 
   - targets:
     - host.docker.internal:9093   

scrape_configs:
- job_name: 'prometheus'
  static_configs:
  - targets: ['localhost:9090']

- job_name: 'node'
  scrape_interval: 5s
  static_configs:
  - targets: ['host.docker.internal:9100']

- job_name: 'redis'
  scrape_interval: 5s
  static_configs:
  - targets: ['host.docker.internal:9121']
  
```

>./promtool query instant http://127.0.0.1:9191 up 

<img width="600" src="https://user-images.githubusercontent.com/10347255/148077707-79fe16f8-72a0-48f0-b178-75ccbf7571b0.png">

### Alerts manger

alerts.yml
```
groups:
- name: instances
  interval: 10s
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
      labelscount: '{{ len $labels }}'
      annotations:
        summary: 'Instance {{ $labels.instance }} is down!'
        description: '{{ $labels.instance }} of job {{ $labels.job}} was down for 1minute'
```

>docker run -d --name alertmanager -v /opt/prometheus/alertmanager.yml:/etc/alertmanager/alertmanager.yml  prom/alertmanager

alertmanager.yml                                                                                    
```
route:
 group_by: ["alertname"]
 group_wait: 30s
 group_interval: 5m
 repeat_interval: 1h
 receiver: "web.hook"
 routes:
 - receiver: "pagerduty-critical"
   group_wait: 10s
   matchers:
   - severity="critical"
 - receiver: "slack-warning"
   group_wait: 60s
   matchers:
   - severity="warning"


receivers:
- name: "web.hook"
  webhook_configs:
  - url: "http://127.0.0.1:5001"
- name: "pagerduty-critical"
  pagerduty_configs:
  - routing_key: "test"
    service_key: "md5-hash"
- name: "slack-warning"
  slack_configs:
  - api_url: "https://hooks.slack.com/services/TOKEN"
    channel: "#warnings"

inhibit_rules:
 - source_matchers:
   - severity="critical"
   target_matchers:
   - severity="warning"
   equal: ["instance"]
```

>docker exec -it alertmanager sh

>amtool check-config /etc/alertmanager/alertmanager.yml 

>amtool config routes --config.file /etc/alertmanager/alertmanager.yml 

## On Docker Swarm

>docker stop $(docker ps -q)

>docker container prune

>docker swarm init

>docker service create <...>
