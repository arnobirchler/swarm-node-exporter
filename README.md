# Swarm node-exporter

This is an extension of the original node exporter for being used with Swarm Clusters so that as special metric with Swarm Node ID and hostname is generated for later join operations.

Inspired on https://github.com/stefanprodan/swarmprom

## Usage
`docker-compose.yml`

```yml
version: '3.7'

services:
  node-exporter:
    image: arnobirchler/swarm-node-exporter:latest
    command:
      - '--path.sysfs=/host/sys'
      - '--path.procfs=/host/proc'
      - '--collector.textfile.directory=/etc/node-exporter/'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'
      - '--no-collector.ipvs'
    environment:
      - NODE_ID={{.Node.ID}}

    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
      - /etc/hostname:/etc/nodename
    deploy:
      mode: global
      resources:
        limits:
          memory: 128M
        reservations:
          memory: 64M
    networks:
      - monitoring_network
```

## Prometheus settings 

Extract of `prometheus.yml`

```yml
#...

scrape_configs:

#...

# Node exporter
  - job_name: 'node-exporter'
    dns_sd_configs:
    - names:
      - 'tasks.node-exporter'
      type: 'A'
      port: 9100
#...
```