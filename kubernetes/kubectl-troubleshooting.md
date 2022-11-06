# kubectl Cheat Sheet > Troubleshooting

[go to kubectl Cheat Sheet](cheat-sheet-kubectl.md)

## Get a Shell to a Running Container

Source: [kubernetes.io/docs](https://kubernetes.io/docs/tasks/debug/debug-application/get-shell-running-container/)

Create a Pod
```
kubectl apply -f https://k8s.io/examples/application/shell-demo.yaml
```

Verify that the container is running:
```
kubectl get pod shell-demo
```

Get a shell to the running container:
```
kubectl exec --stdin --tty shell-demo -- /bin/bash
```

Add some tools for investigating DNS and the network.
```
apt-get update ; apt-get install curl dnsutils -y
```

Note: The double dash (--) separates the arguments you want to pass to the command from the kubectl arguments.

## Logging

### Pod and container logs

This example uses a manifest for a Pod with a container that writes text to the standard output stream, once per second.
```
kubectl apply -f https://k8s.io/examples/debug/counter-pod.yaml
```

```
kubectl logs counter
```

You can use this to retrieve logs from a previous instantiation of a container.
```
kubectl logs counter --previous
```

[more additional information on kubernetes docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs)

![logging node level](images/logging-node-level.png)

### System component logs

On Linux nodes that use systemd, the kubelet and container runtime write to journald by default.
```
journalctl -u kubelet
```
If systemd is not present, the kubelet and container runtime write to .log files in the /var/log directory.

### Cluster-level logging architectures

While Kubernetes does not provide a native solution for cluster-level logging, there are several common approaches you can consider. Here are some options:

- Use a node-level logging agent that runs on every node.
- Include a dedicated sidecar container for logging in an application pod.
- Push logs directly to a backend from within an application.

__Using a node logging agent__
![logging with node agent](images/logging-with-node-agent.png)

__Using a sidecar container with the logging agent__
You can use a sidecar container in one of the following ways:

- The sidecar container streams application logs to its own stdout.
- The sidecar container runs a logging agent, which is configured to pick up logs from an application container.

![logging with streaming sidecar](images/logging-with-streaming-sidecar.png)

It is not recommended to write log entries with different formats to the same log stream, even if you managed to redirect both components to the stdout stream of the container. Instead, you can create two sidecar containers. Each sidecar container could tail a particular log file from a shared volume and then redirect the logs to its own stdout stream.

[two-files-counter-pod-streaming-sidecar.yaml](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/admin/logging/two-files-counter-pod-streaming-sidecar.yaml)

```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done      
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-1
    image: busybox:1.28
    args: [/bin/sh, -c, 'tail -n+1 -F /var/log/1.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-log-2
    image: busybox:1.28
    args: [/bin/sh, -c, 'tail -n+1 -F /var/log/2.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
```

Now when you run this pod, you can access each log stream separately by running the following commands:
```
kubectl logs counter count-log-1
```

or
```
kubectl logs counter count-log-2
```

### Sidecar container with a logging agent

![logging with sidecar agent](images/logging-with-sidecar-agent.png)

Here are two example manifests that you can use to implement a sidecar container with a logging agent. The first manifest contains a ConfigMap to configure fluentd.

[fluentd-sidecar-config.yaml](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/admin/logging/fluentd-sidecar-config.yaml)
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluentd.conf: |
    <source>
      type tail
      format none
      path /var/log/1.log
      pos_file /var/log/1.log.pos
      tag count.format1
    </source>

    <source>
      type tail
      format none
      path /var/log/2.log
      pos_file /var/log/2.log.pos
      tag count.format2
    </source>

    <match **>
      type google_cloud
    </match>    
```

[two-files-counter-pod-agent-sidecar.yaml](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/admin/logging/two-files-counter-pod-agent-sidecar.yaml)
```
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done      
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-agent
    image: registry.k8s.io/fluentd-gcp:1.30
    env:
    - name: FLUENTD_ARGS
      value: -c /etc/fluentd-config/fluentd.conf
    volumeMounts:
    - name: varlog
      mountPath: /var/log
    - name: config-volume
      mountPath: /etc/fluentd-config
  volumes:
  - name: varlog
    emptyDir: {}
  - name: config-volume
    configMap:
      name: fluentd-config
```

### Exposing logs directly from the application

![logging from application](images/logging-from-application.png)


### Metrics Server

The metrics-server fetches resource metrics from the kubelets and exposes them in the Kubernetes API server through the Metrics API for use by the HPA and VPA.

__Install Metrics Server__

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

__Check Metrics Server__

```
kubectl get deployment metrics-server -n kube-system
```

__Get metrics__

```
kubectl top node
```

Hint: Error cause

Source: [Kubernetes metrics-server Error – Readiness probe failed: HTTP probe failed with statuscode](https://www.scmgalaxy.com/tutorials/kubernetes-metrics-server-error-readiness-probe-failed-http-probe-failed-with-statuscode/)
```
kubectl logs deployment/metrics-server -n kube-system
```

```
E0713 16:52:04.774647       1 scraper.go:139] "Failed to scrape node" err="Get \"https://172.31.12.77:10250/stats/summary?only_cpu_and_memory=true\": x509: cannot validate certificate for 172.31.12.77 because it doesn't contain any IP SANs" node="ip-172-31-12-77"
```

Steps:

- Download https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
- Modify and add "- --kubelet-insecure-tls" in deployment.spec.template.spec.containers.args
```
...
...
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
```
- kubectl apply -f components.yaml
- kubectl top nodes
- kubectl top pod kube-proxy-xyz -n=kube-system
