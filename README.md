# Sample Microservice Application

Stan's Robot Shop is a sample microservice application you can use as a sandbox to test and learn containerised application orchestration and monitoring techniques. It is not intended to be a comprehensive reference example of how to write a microservices application, although you will better understand some of those concepts by playing with Stan's Robot Shop. To be clear, the error handling is patchy and there is not any security built into the application.

You can get more detailed information from my [blog post](https://www.instana.com/blog/stans-robot-shop-sample-microservice-application/) about this sample microservice application.

This sample microservice application has been built using these technologies:
- NodeJS ([Express](http://expressjs.com/))
- Java ([Spring Boot](https://spring.io/))
- Python ([Flask](http://flask.pocoo.org))
- Golang
- PHP (Apache)
- MongoDB
- Redis
- MySQL ([Maxmind](http://www.maxmind.com) data)
- RabbitMQ
- Nginx
- AngularJS (1.x)

The various services in the sample application already include all required Instana components installed and configured. The Instana components provide automatic instrumentation for complete end to end [tracing](https://docs.instana.io/core_concepts/tracing/), as well as complete visibility into time series metrics for all the technologies.

To see the application performance results in the Instana dashboard, you will first need an Instana account. Don't worry a [trial account](https://instana.com/trial?utm_source=github&utm_medium=robot_shop) is free.

## Build from Source
To optionally build from source (you will need a newish version of Docker to do this) use Docker Compose. Optionally edit the `.env` file to specify an alternative image registry and version tag; see the official [documentation](https://docs.docker.com/compose/env-file/) for more information.

To download the tracing module for Nginx, it needs a valid Instana agent key. Set this in the environment before starting the build.

```shell
$ export INSTANA_AGENT_KEY="<your agent key>"
```

Now build all the images.

```shell
$ docker-compose build
```

If you modified the `.env` file and changed the image registry, you need to push the images to that registry

```shell
$ docker-compose push
```

## Run Locally
You can run it locally for testing.

If you did not build from source, don't worry all the images are on Docker Hub. Just pull down those images first using:

```shell
$ docker-compose pull
```

Fire up Stan's Robot Shop with:

```shell
$ docker-compose up
```

If you want to fire up some load as well:

```shell
$ docker-compose -f docker-compose.yaml -f docker-compose-load.yaml up
```

If you are running it locally on a Linux host you can also run the Instana [agent](https://docs.instana.io/quick_start/agent_setup/container/docker/) locally, unfortunately the agent is currently not supported on Mac.

There is also only limited support on ARM architectures at the moment.

## Marathon / DCOS

The manifests for robotshop are in the *DCOS/* directory. These manifests were built using a fresh install of DCOS 1.11.0. They should work on a vanilla HA or single instance install.

You may install Instana via the DCOS package manager, instructions are here: https://github.com/dcos/examples/tree/master/instana-agent/1.9

## Kubernetes
You can run Kubernetes locally using [minikube](https://github.com/kubernetes/minikube) or on one of the many cloud providers.

The Docker container images are all available on [Docker Hub](https://hub.docker.com/u/robotshop/).

Install Stan's Robot Shop to your Kubernetes cluster using the [Helm](K8s/helm/README.md) chart.

To deploy the Instana agent to Kubernetes, just use the [helm](https://github.com/instana/helm-charts) chart.

## Accessing the Store
If you are running the store locally via *docker-compose up* then, the store front is available on localhost port 8080 [http://localhost:8080](http://localhost:8080/)

If you are running the store on Kubernetes via minikube then, find the IP address of Minikube and the Node Port of the web service.

```shell
$ minikube ip
$ kubectl get svc web
```

If you are using a cloud Kubernetes / Openshift / Mesosphere then it will be available on the load balancer of that system.

## Load Generation
A separate load generation utility is provided in the `load-gen` directory. This is not automatically run when the application is started. The load generator is built with Python and [Locust](https://locust.io). The `build.sh` script builds the Docker image, optionally taking *push* as the first argument to also push the image to the registry. The registry and tag settings are loaded from the `.env` file in the parent directory. The script `load-gen.sh` runs the image, it takes a number of command line arguments. You could run the container inside an orchestration system (K8s) as well if you want to, an example descriptor is provided in K8s directory. For End-user Monitoring ,load is not automatically generated but by navigating through the Robotshop from the browser .For more details see the [README](load-gen/README.md) in the load-gen directory.  

## Website Monitoring / End-User Monitoring

### Docker Compose

To enable Website Monioring / End-User Monitoring (EUM) see the official [documentation](https://docs.instana.io/website_monitoring/) for how to create a configuration. There is no need to inject the JavaScript fragment into the page, this will be handled automatically. Just make a note of the unique key and set the environment variable `INSTANA_EUM_KEY` and `INSTANA_EUM_REPORTING_URL` for the web image within `docker-compose.yaml`.

### Kubernetes

The Helm chart for installing Stan's Robot Shop supports setting the key and endpoint url required for website monitoring, see the [README](K8s/helm/README.md).

## Prometheus

The cart and payment services both have Prometheus metric endpoints. These are accessible on `/metrics`. The cart service provides:

* Counter of the number of items added to the cart

The payment services provides:

* Counter of the number of items perchased
* Histogram of the total number of items in each cart
* Histogram of the total value of each cart

To test the metrics use:

```shell
$ curl http://<host>:8080/api/cart/metrics
$ curl http://<host>:8080/api/payment/metrics
```


---

# DevOps Capstone: Deployment, Logging and Operations

This repository was extended as a hands-on DevOps capstone using Kubernetes, Helm, and a local Minikube cluster.

## Project Work Completed

- Deployed Robot Shop using the Helm chart in `K8s/helm`.
- Updated application deployment configurations and removed Instana agent host settings from the targeted service templates.
- Added Fluent Bit and Loki configuration for Kubernetes container log collection and querying.
- Created operational documentation covering incident response, an incident postmortem, and an availability SLO policy.
- Performed an application availability drill by scaling the web deployment down to zero replicas and restoring it to one replica.
- Committed and pushed the project changes to the `master` branch.

## Logging Architecture

The logging pipeline collects Kubernetes container logs using Fluent Bit and sends them to Loki for querying.

Configuration files:

- `fluent-bit-values.yaml` — Fluent Bit collection, Kubernetes metadata filtering, and Loki output configuration.
- `loki-writable-values.yaml` — Loki configuration for the local lab environment.

### Logging Verification

The Loki readiness endpoint returned `ready`. The Loki labels API returned successfully, and a range query for the `robot-shop` namespace returned real application log entries.

Example local verification, with the Loki port-forward running:

```powershell
Invoke-RestMethod http://127.0.0.1:3101/ready

$query = [uri]::EscapeDataString('{namespace="robot-shop"}')
$result = Invoke-RestMethod -Uri "http://127.0.0.1:3101/loki/api/v1/query_range?query=$query&limit=5"
$result.data.result | ConvertTo-Json -Depth 8
```

The port-forward maps local port `3101` to Loki service port `3100`.

## Operational Documentation

See `docs/operations/` for:

- `incident-runbook.md` — incident detection, triage, mitigation, recovery, and closure.
- `postmortem.md` — the web availability drill, recovery actions, and lessons learned.
- `slo-policy.md` — the proposed availability target, error budget, and alerting policy.

The availability target is a proposed lab policy, not a measured production SLO.

## Current Limitations

This is a local learning environment, not a production-ready deployment.

- The Kubernetes API has experienced intermittent timeouts.
- The `ratings` pod was not Ready during the last successful pod listing.
- Prometheus and Grafana were installed, but some monitoring components were restarting and the stack was not consistently healthy.
- A successful end-to-end OpenTelemetry trace has not been verified.
- Slack alert delivery has not been verified.

These items remain outstanding and should be resolved before treating the full observability setup as complete.

## Reproducibility and Safety

Use the Helm chart and configuration files in this repository as the starting point for reproducing the lab. Confirm that the Minikube cluster and required namespaces are healthy before applying changes.

The included Loki configuration uses local lab storage and is not intended for durable production log retention. Do not commit credentials, API tokens, or Slack webhook URLs.

