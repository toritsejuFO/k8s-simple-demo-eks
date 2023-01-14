# k8s-simple-demo-eks

Documentation can be found [here (project22)](https://github.com/toritsejuFO/darey.io-projects/tree/main/project-22) and [here (project23)](https://github.com/toritsejuFO/darey.io-projects/tree/main/project-23)


### The nginx-sidecar implementation (using minikube)

For the `nginx-sidecar` pod file, I demonstrate how to mount volumes from my local machine onto the node container and the sidecar pattern.

#### Caveat
Volumes **can not be mounted directly** from local machine to container. To achieve this, we first have to mount our local machine volume onto the minikube cluster like so

`minikube mount /path/on/local/machine:/path/on/vm`

In my case, my local machine volume is named `local-vol` on my Desktop, and the path on the minikube vm (docker, I used docker as a driver) as specified in the `hostPath.path` in the `nginx-sidecar` file is `/vm-vol`. Hence, to mount the local volume onto the minikube vm, I use

`minikube mount ~/local-vol:/vm-vol`

When my pod starts, the `vm-vol` path on my minikube vm will be mounted into the containers, as the `app-container` container writes to it's `/var/log` dir, it gets propagated to the `/vm-vol` dir on the minikube cluster, and that gets propagated to my local machine's `local-vol` dir.

### Sidecar Pattern

We have two containers, `app-container` container, and `log-exporter-sidecar` container. The main container is only concerned with generating logs and nothing else, but what good are the logs if we can't view them, that's where the pattern comes in. The helper container has **nginx** running, and can serve us our log file via http.

The pattern is demonstrated thus;

1. We have our main `app-container` container, writing to a log file, which is propagated to the `vm-vol` volume
2. The nginx `log-exporter-sidecar` container also mounts, the `vm-vol` volume, hence it has access to what is being written there by the main container.
3. A port-forward is performed, which exposes the nginx service to our local machine
4. Using curl we can then access the app.log file content being served by nginx (browser attempts a download)

### Reference article
https://betterprogramming.pub/understanding-kubernetes-multi-container-pod-patterns-577f74690aee
