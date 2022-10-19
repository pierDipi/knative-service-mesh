# Service Mesh Federation

## Prerequisites

- Install Red Hat OpenShift Serverless
- Install Red Hat OpenShift Service Mesh
- Install Red Hat OpenShift distributed tracing platform
- Install Red Hat Kiali
- alias `kubectl` or `oc` to `k` (Optional)

## Configuration

```shell
# Create Kafka cluster
k apply -f config/kafka/100-amq-streams-kafka.yaml

# Install Knative Eventing
k apply -f config/knative-eventing/100-knative-eventing.yaml
k apply -f config/knative-eventing/101-knative-kafka.yaml

# Create Mesh in sandbox-sm with sandbox and sandbox-sm namespaces federated with knative-eventing-sm
k apply -f config/sandbox-sm/100-namespace.yaml
k apply -f config/sandbox-sm/200-sm-control-plane.yaml
k apply -f config/sandbox-sm/300-sm-member-roll.yaml

# Allow API server to access Knative Eventing webhooks
k apply -f config/knative-eventing/200-allow-api-server-access-webhook.yaml

# Create Mesh in knative-eventing-sm (knative-eventing and knative-eventing-sm namespaces) federated with sandbox-sm
k apply -f config/knative-eventing-sm/100-namespace.yaml
k apply -f config/knative-eventing-sm/200-sm-control-plane.yaml
k apply -f config/knative-eventing-sm/300-sm-member-roll.yaml
# Create ServiceEntry for Kafka cluster
#   Before executing this command you need to add all Kafka pods IPs to the `addresses` field,
#   see https://istio.io/v1.13/blog/2018/egress-tcp/ for more details.
#   TODO does this limitation still apply?
k apply -f config/knative-eventing-sm/400-kafka-service-entry.yaml

# Inject sidecars to Knative Kafka pods

k patch -n knative-eventing deployment kafka-controller --patch-file config/patch/kafka-broker-patch.yaml

k patch -n knative-eventing deployment kafka-broker-receiver --patch-file config/patch/kafka-broker-patch.yaml
k patch -n knative-eventing deployment kafka-broker-dispatcher --patch-file config/patch/kafka-broker-patch.yaml

k patch -n knative-eventing deployment kafka-broker-receiver --patch-file config/patch/kafka-broker-patch.yaml
k patch -n knative-eventing deployment kafka-channel-dispatcher --patch-file config/patch/kafka-broker-patch.yaml

k patch -n knative-eventing deployment kafka-sink-receiver --patch-file config/patch/kafka-broker-patch.yaml
k patch -n knative-eventing deployment kafka-source-dispatcher --patch-file config/patch/kafka-broker-patch.yaml

# Install application and an Knative Kafka Broker
k apply -f config/sandbox/100-namespace.yaml
k apply -f config/sandbox/200-kafka-broker-example.yaml

# Get Knative Kafka Brokers
k get -n sandbox brokers # use describe for more details

```