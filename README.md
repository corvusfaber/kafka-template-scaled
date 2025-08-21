````markdown
# 🐘 Kafka Producer/Consumer on Kubernetes with Helm + KEDA

This project provides a **Helm chart** for deploying Apache Kafka workloads (producers, consumers, and supporting jobs) to Kubernetes.  
It supports **topic bootstrapping**, **autoscaling consumers with KEDA**, and configurable deployments via `values.yaml`.

---

## 📦 Features

- 📜 Deploys Kafka **producers** and **consumers** with Helm
- 🗂️ Optional `Job` for **automatic topic creation**
- ⚡ **KEDA integration** for autoscaling consumers based on Kafka lag
- 🪛 Configurable with `values.yaml` (replicas, images, topics, env vars, ports)
- 🪄 Kubernetes-native objects:
  - `StatefulSet` for Kafka core
  - `Deployment` for producers/consumers
  - `Service` for networking
  - `ScaledObject` for autoscaling

---

## 📁 Project Structure

```plaintext
.
├── helm/
│   ├── templates/
│   │   ├── statefulset.yaml      # Kafka StatefulSet definition
│   │   ├── service.yaml          # Kafka Service definition
│   │   ├── producer.yaml         # Kafka Producer Deployment
│   │   ├── consumer.yaml         # Kafka Consumer Deployment
│   │   ├── kafka-topic-job.yaml  # Job to create topics
│   │   └── scaledobject.yaml     # KEDA autoscaling config
│   └── values.yaml               # Helm values (replicas, image, env, ports)
````

---

## 🚀 Getting Started

### 1. Prerequisites

* Kubernetes cluster (local with Minikube, or cloud)
* [Helm](https://helm.sh/docs/intro/install/) installed
* (Optional) [KEDA](https://keda.sh) installed for autoscaling

---

### 2. Deploy via Helm

```bash
helm install kafka-app ./helm -n kafka --create-namespace
```

To upgrade:

```bash
helm upgrade kafka-app ./helm -n kafka
```

---

### 3. Verify Deployment

```bash
kubectl get pods -n kafka
kubectl get svc -n kafka
```

---

## ⚙️ Configuration (`values.yaml`)

### StatefulSet

```yaml
statefulset:
  name: kafka
  namespace: kafka
  replicas: 3
  appname: kafka-app
  serviceName: kafka-svc
  image: bitnami/kafka:3.7.0
  ports:
    - containerPort: 9092
  env:
    - name: DEFAULT_REPLICATION_FACTOR
      value: "1"
    - name: DEFAULT_MIN_INSYNC_REPLICAS
      value: "1"
```

### Producer

```yaml
producer:
  name: kafka-producer
  namespace: kafka
  replicas: 3
  image: bitnami/kafka:3.7.0
  topic: new-demo-topic
  intervalSeconds: 5
```

### Consumer

```yaml
consumer:
  name: kafka-consumer
  namespace: kafka
  replicas: 3
  image: bitnami/kafka:3.7.0
  topic: new-demo-topic
```

### Topic Job

```yaml
topicJob:
  enabled: true
  topicName: new-demo-topic
  partitions: 3
  replicationFactor: 1
```

### KEDA (optional)

```yaml
keda:
  enabled: true
  ScaledObject: kafka-consumer
  minReplicaCount: 1
  maxReplicaCount: 5
  topic: new-demo-topic
  bootstrapServers: kafka-svc:9092
  consumerGroup: consumer-group-1
  lagThreshold: "10"
```

---

## 🔁 Example Workflow (with KEDA)

1. 🐳 Deploy Kafka StatefulSet + Service
2. 🏗️ Run `Job` to create topic (`new-demo-topic`)
3. 📤 Deploy **producers** to send messages every few seconds
4. 📥 Deploy **consumers** to read from topic
5. ⚡ Enable **KEDA ScaledObject** → consumers autoscale based on lag

---

## 🧹 Cleanup

To uninstall:

```bash
helm uninstall kafka-app -n kafka
kubectl delete namespace kafka
```

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 🙋 Contributions

PRs and issues are welcome! 🚀

```
