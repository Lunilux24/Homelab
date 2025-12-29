# Monitoring Stack

I wanted to get better visibility into my home lab environment, so I set up a monitoring stack using Glances, Prometheus, and Grafana. This stack collects metrics from various services and provides a user-friendly dashboard for visualization.

Setting up the stack gave me a lot of issues because of Glances' unwillingness to work out the box with Prometheus in a Docker environment. After a lot of trial and error, I managed to get everything working smoothly. My main issue was getting the Glances Prometheus exporter to expose metrics correctly when running inside a Docker container. I had to adjust the network settings and ensure that the correct ports were exposed. Additionally, I had to configure Prometheus to scrape metrics from the Glances exporter properly. Once I got past those hurdles, the stack has been running reliably, providing valuable insights into my home lab's performance and resource usage, although I have yet to customize the Grafana dashboards to my liking.

Here is the `docker-compose.yml` file I used to set up the monitoring stack:

```yaml
services:
  glances:
    image: nicolargo/glances:latest-full
    container_name: glances
    restart: unless-stopped
    pid: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./glances.conf:/etc/glances/glances.conf:ro
    environment:
      - GLANCES_OPT=--export prometheus --conf /etc/glances/glances.conf
    ports:
      - "9091:9091"
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    ports:
      - "9190:9090"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3002:3000"
    volumes:
      - ./grafana/data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - glances
    networks:
      - monitoring

networks:
  monitoring:
    driver: bridge
```

The main lines to pay attention to are the `GLANCES_OPT` environment variable in the Glances service, which enables the Prometheus exporter, and the volume mounts that ensure configuration files are correctly loaded. Make sure to adjust the paths and settings according to your specific environment and requirements. It was also important to configure the monitoring network to allow communication between the services. This is so that Prometheus can scrape metrics from Glances and Grafana can visualize them. It is imperative that all services are on the same Docker network for this to work seamlessly.

After setting up the stack, I can now monitor CPU, memory, disk usage, and network statistics of my home lab environment in real-time. The Grafana dashboards provide a clear and intuitive way to visualize the data collected by Prometheus from Glances. Overall, this monitoring stack has been a valuable addition to my home lab setup, helping me keep track of system performance and identify potential issues before they become critical.