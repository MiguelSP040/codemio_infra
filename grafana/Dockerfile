FROM grafana/grafana:latest

COPY datasources.yml /etc/grafana/provisioning/datasources/datasources.yml
COPY dashboards.yml /etc/grafana/provisioning/dashboards/dashboards.yml

EXPOSE 3000

USER grafana
