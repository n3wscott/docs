How to use a Broker and a Trigger to invoke a function.

## Deployment Steps

### Prerequisites

1. Setup [Knative Serving](../../../serving).
1. Setup [Knative Eventing](../../../eventing).

### Enable Default Broker

Eventing will watch the namespace to create a default broker. To get a default
broker, simply add the label `knative-eventing-injection=enabled` to the default
namespace:

```bash
$ kubectl label namespace default knative-eventing-injection=enabled
```

You can check that the correct labels have been added with,

```bash
$  kubectl get namespace default --show-labels
NAME      STATUS    AGE       LABELS
default   Active    48d       istio-injection=enabled,knative-eventing-injection=enabled
```

There should now be a default broker in the default namespace,

```bash
$ kubectl get broker
NAME      READY     REASON    HOSTNAME
default   True                default-broker.default.svc.cluster.local
```

### Create a function

In order to verify `CronJobSource` is working, we will create a simple Knative
Service that dumps incoming messages to its log.

```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: message-dumper
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            image: gcr.io/knative/github.com/knative/eventing/cmd/pong
```

Use following command to create the service from `service.yaml`:

```shell
kubectl apply --filename service.yaml
```


### Verify

We will verify that the message was sent to the Knative eventing system by
looking at message dumper logs.

```shell
kubectl logs -l serving.knative.dev/service=message-dumper -c user-container --since=10m
```

You should see log lines showing the request headers and body from the source:

```
2019/03/14 14:28:06 Message Dumper received a message: POST / HTTP/1.1
Host: message-dumper.default.svc.cluster.local
Transfer-Encoding: chunked
Accept-Encoding: gzip
Ce-Cloudeventsversion: 0.1
Ce-Eventid: 9790bf44-914a-4e66-af59-b43c06ccb73b
Ce-Eventtime: 2019-03-14T14:28:00.005163309Z
Ce-Eventtype: dev.knative.cronjob.event
Ce-Source: CronJob
...

{"message":"Hello world!"}
```

You can also use [`kail`](https://github.com/boz/kail) instead of `kubectl logs`
to tail the logs of the subscriber.

```shell
kail -l serving.knative.dev/service=message-dumper -c user-container --since=10m
```
