### Learning Prometheus <br>

Prometheus docker 

>docker run -p 9191:9090 -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
                               
Setting up exporters

>sudo nano /opt/prometheus/prometheus.yml    
                               
```
global:
 scrape_interval: 15s

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

>./promtool query instant http://127.0.0.1:9090 up 
