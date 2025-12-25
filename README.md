# Loki Logs and Prometheus Metrics

This repo contains all the code needed to follow along with our **[YouTube Tutorial](https://youtu.be/0POI5E7Uzjo)** or **[Written Article](https://rslim087a.github.io/rayanslim/lesson.html?course=loki-prometheus-grafana-course&lesson=introduction)**.

## Prerequisites

To follow along with this tutorial, you'll need:

- kubectl installed and configured ([https://youtu.be/IBkU4dghY0Y](https://youtu.be/IBkU4dghY0Y))
- Helm installed: [https://rslim087a.github.io/rayanslim/lesson.html?course=prometheus-grafana-monitoring-course&lesson=helm-installation](https://rslim087a.github.io/rayanslim/lesson.html?course=prometheus-grafana-monitoring-course&lesson=helm-installation)

**Become a Cloud and DevOps Engineer:** [https://rslim087a.github.io/rayanslim](https://rslim087a.github.io/rayanslim)

## Handy URLs

- Loki Helm Repository: https://grafana.github.io/helm-charts/
- Loki values.yaml: https://github.com/grafana/loki/blob/main/production/helm/loki/values.yaml
- Prometheus Helm Repository: https://prometheus-community.github.io/helm-charts
- Prometheus values.yaml: https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml

## Installing Loki

Installing loki if custom-values.yaml file isn't present 

***Some parameters may be missing; best to refer to the video tutorial for complete setup instructions).***

```bash
helm install loki grafana/loki --version 6.29.0 --namespace monitoring \
  --set deploymentMode=SingleBinary \
  --set loki.auth_enabled=false \
  --set singleBinary.replicas=1 \
  --set write.replicas=0 \
  --set read.replicas=0 \
  --set backend.replicas=0 \
  --set loki.commonConfig.replication_factor=1 \
  --set loki.storage.type=filesystem \
  --set loki.storage.filesystem.directory=/var/loki/chunks \
  --set loki.useTestSchema=true \
  --set chunksCache.enabled=false \
  --set resultsCache.enabled=false \
  --set test.enabled=false
```
### Testing Loki

port-forward

```bash
kubectl port-forward --namespace monitoring svc/loki-gateway 3100:80
```

send test logs

```bash
curl -H "Content-Type: application/json" -H "X-Scope-OrgID: fake" -XPOST -s "http://localhost:3100/loki/api/v1/push" \
  --data-raw '{"streams": [{"stream": {"job": "test"}, "values": [["'$(date +%s)'000000000", "test log entry"]]}]}'
```

query for the same logs

```bash
curl -H "X-Scope-OrgID: fake" "http://localhost:3100/loki/api/v1/query_range" --data-urlencode 'query={job="test"}'
```

## Installing kube-prometheus-stack

Installing kube-prometheus-stack if custom-values.yaml file isn't present. This adds loki as an additional data source.

***Some parameters may be missing; best to refer to the video tutorial for complete setup instructions).***

```bash
helm install prometheus prometheus-community/kube-prometheus-stack --version 45.7.1 \
  --namespace monitoring \
  --set "grafana.additionalDataSources[0].name=Loki" \
  --set "grafana.additionalDataSources[0].type=loki" \
  --set "grafana.additionalDataSources[0].url=http://loki-gateway.monitoring.svc.cluster.local" \
  --set "grafana.additionalDataSources[0].access=proxy"
```

### Testing Prometheus

port-forward

```bash
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
```

### Testing Grafana

Access grafana password

```bash
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

port-forward

```bash
kubectl port-forward -n monitoring svc/grafana 3000:80
```

## Become a Cloud and DevOps Engineer

Learn every tool that matters: [https://rslim087a.github.io/rayanslim](https://rslim087a.github.io/rayanslim)
