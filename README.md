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

1. [Helm](http://github.com/kubernetes/helm)

# Setup

1. go to prometheus-k8s directory
2. on one terminal, run

    ```
    tiller
    ```
3. copy down tiller listening port number
4. open another terminal, run

    ```
    helm install . --host 0.0.0.0:<tiller port number>
    ```
    e.g.
    ```
    helm install . --host 0.0.0.0:44134
    ```
5. after deployed, run

    ```
    charts/grafana/create-datasource.sh
    ```
6. on terminal, run

    ```
    minikube dashboard
    minikube service prometheus
    minikube service grafana
    ```
7. go to browser, and find the internal ip address of prometheus
8. assign grafana instance to prometheus internal ip address with 9090 port (e.g. 172.17.0.8:9090)

# Migrate into AWS

1. For service, NodePort should change to LoadBalancer.
2. It will use 'port' instead of 'NodePort'.

# Federate with another region or dev / production

Please see the config on federate-configmap.yaml.sample. Pay special attention on job_name: 'federate' settings.

1. 'match[]' config must be '{__name__=~".+"}'
1. Should skip tls verify.
1. Kill prometheus pod to get config work immediately (curl -XPOST http://<prometheus>:9090/-/reload not work properly)

# Docker Registry

Currently I include a docker registry in k8s for ease of testing / development. Have tried mounting into some persist volume but failed. Mounting into the host VM would be far faster.

Extra steps:

1. Add "--insecure-registry=<minikube ip>:5000" into DOCKER_OPTS
1. Start minikube with "--insecure-registry <minikube ip>:5000". 

    ```
    minikube start --insecure-registry 192.168.99.100:5000
    ```

In general, minikube ip should be fixed on 192.168.99.100. If you use the registry inside minikube, please bear in mind to update your deployment yaml file before and after your experiments.

# Prometheus scrape custom service

You need to put something like this in your service:

    ```
    apiVersion: v1
    kind: Service
    metadata:
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '4001'
      ...
    spec:
      ...
    ```

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
