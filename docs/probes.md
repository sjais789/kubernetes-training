# kubernetes-training

## `liveness probe`

Kubernetes can probe a container using one of the three mechanisms:

- An HTTP GET probe performs an HTTP GET request on the container’s IP address, a port and path you specify. If the probe receives a response, and the response code doesn’t represent an error (in other words, if the HTTP response code is 2xx or 3xx), the probe is considered successful. If the server returns an error response code or if it doesn’t respond at all, the probe is considered a failure and the container will be restarted as a result.


- A TCP Socket probe tries to open a TCP connection to the specified port of the container. If the connection is established successfully, the probe is successful. Otherwise, the container is restarted.

- An Exec probe executes an arbitrary command inside the container and checks the command’s exit status code. If the status code is 0, the probe is successful. All other codes are considered failures.

:warning: Always remember to set an initial delay to account for your app’s startup time. :warning:

:star: If you'd like to start sending traffic to a Pod only when a probe succeeds, specify a readiness probe. In this case, the readiness probe might be the same as the liveness probe, but the existence of the readiness probe in the spec means that the Pod will start without receiving any traffic and only start receiving traffic after the probe starts succeeding. If your container needs to work on loading large data, configuration files, or migrations during startup, specify a readiness probe.

:star: If you want your container to be able to take itself down for maintenance, you can specify a readiness probe that checks an endpoint specific to readiness that is different from the liveness probe.

### example

Start a pod with a HTTP Get liveness probe:

```bash
❯ kubectl apply -f k8s/liveness-probe/dummy_node_app.yaml

# you might see the following error
error: error validating "k8s/liveness-probe/dummy_node_app_with_probes.yaml": error validating data: ValidationError(Pod.spec.containers[0]): unknown field "startupProbe" in io.k8s.api.core.v1.Container; if you choose to ignore these errors, turn validation off with --validate=false
# on my GKE 1.15.12 startupProbe was not enabled
# https://v1-15.docs.kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#container-v1-core
# You can add the --validate=false to start the pod and omit the startupProbe
```

Wait for a minute and then describe the pod:

```bash
❯ kubectl describe pods dummy-node-app-with-liveness-probe
#
# At the end you will se the events logs
#
Events:
  Type     Reason     Age                  From                                                  Message
  ----     ------     ----                 ----                                                  -------
  Normal   Scheduled  2m27s                default-scheduler                                     Successfully assigned chapter2/dummy-node-app-with-liveness-probe to gke-k8s-training-default-pool-3088a1f5-rd72
  Warning  Unhealthy  59s (x3 over 79s)    kubelet, gke-k8s-training-default-pool-3088a1f5-rd72  Liveness probe failed: HTTP probe failed with statuscode: 500
  Normal   Killing    59s                  kubelet, gke-k8s-training-default-pool-3088a1f5-rd72  Container dummy-liveness-probe failed liveness probe, will be restarted
  Normal   Pulling    29s (x2 over 2m25s)  kubelet, gke-k8s-training-default-pool-3088a1f5-rd72  Pulling image "tommystark/dummy-node-app"
  Normal   Pulled     27s (x2 over 2m23s)  kubelet, gke-k8s-training-default-pool-3088a1f5-rd72  Successfully pulled image "tommystark/dummy-node-app"
  Normal   Created    27s (x2 over 2m23s)  kubelet, gke-k8s-training-default-pool-3088a1f5-rd72  Created container dummy-liveness-probe
  Normal   Started    27s (x2 over 2m22s)  kubelet, gke-k8s-training-default-pool-3088a1f5-rd72  Started container dummy-liveness-probe
```


This will go undefinitely and you can check that your pod is periodically restarted as soon as it is considered unhealty.

```bash
❯ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
dummy-node-app-with-liveness-probe   1/1     Running   5          11m
```