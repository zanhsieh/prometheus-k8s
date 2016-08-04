# Kubernetes configs to setup prometheus

Initially aimed at minikube, but should be adaptable
for real clusters.

- Adds node_exporter DaemonSet to each node
- Creates a persistant volume and claim for prometheus data
- Creates a prometheus instance
- Creates an alertmanager instance.
- Creates a pushgateway instance.
- Creates a statsd exporter instance.
- Creates a blackbox exporter instance.
- Prometheus will monitor all elements of the kube cluster (including
  services)
- A grafana deployment (no state is kept yet, and no default
  data source setup)
- A templated grafana dashboard (not loaded by default yet)


# Dependencies

1. http://github.com/kubernetes/helm

# Setup

1. go to prometheus-k8s directory
2. on one terminal, run

```bash
tiller
```

3. copy down tiller port number
4. open another terminal, run

```bash
helm install . --host 0.0.0.0:<tiller port number>
```

e.g.

```bash
helm install . --host 0.0.0.0:44134
```

5. after deployed, run

```bash
charts/grafana/create-datasource.sh
```

6. on terminal, run

```bash
minikube dashboard
minikube service prometheus
minikube service grafana
```

7. go to browser, and find the internal ip address of prometheus
8. assign grafana instance to prometheus internal ip address with 9090 port (e.g. 172.17.0.8:9090)

# TODO

- How to specify the volumes?
- Prometheus rules in a config map
- Reload prometheus config on rule update
- HA Prom (should just mean changing the rep count)
- HA Alertmanager
- Alertmanager config

# License

These configs are released under the Apache 2.0 license. All images
downloaded are subject to their individual licenses.
