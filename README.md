````markdown
# 🐘 Kafka on Kubernetes with Helm and GitHub Actions (KRaft Mode)

This project automates the deployment of an Apache Kafka cluster (in **KRaft mode**, i.e., ZooKeeper-less) to a Kubernetes cluster using **Helm** and **GitHub Actions**. It provisions a `StatefulSet`, exposes it via a `NodePort` service, and verifies Kafka functionality through automated topic/message tests.

---

## 📦 Features

- 📜 Kafka cluster in **KRaft mode** (ZooKeeper-free)
- 🪛 Configurable with `values.yaml` (replicas, ports, env vars)
- 🪄 Helm-based Kubernetes deployment (StatefulSet + PVC + Service)
- ⚙️ GitHub Actions pipeline to:
  - Set up Minikube
  - Deploy Kafka via Helm
  - Create Kafka topic
  - Produce & consume Kafka messages

---

## 📁 Project Structure

```plaintext
.
├── helm/
│   ├── templates/
│   │   ├── statefulset.yaml     # Kafka StatefulSet definition
│   │   └── service.yaml         # Kafka Service definition
│   └── values.yaml              # Helm values (replicas, image, env, ports)
├── .github/
│   └── workflows/
│       └── ci-cd.yaml           # GitHub Actions CI/CD pipeline
````

---

## 🚀 Getting Started

### 1. Prerequisites

* GitHub repository
* Kubernetes cluster (or Minikube locally)
* [Helm](https://helm.sh/docs/intro/install/) installed
* GitHub Secrets:

  * `KUBECONFIG_CONTENT`: your base64-encoded `kubeconfig` file

---

### 2. Deploy Kafka via Helm

```bash
helm install kafka ./helm -n kafka --create-namespace
```

To upgrade:

```bash
helm upgrade kafka ./helm -n kafka
```

---

### 3. Verify Deployment

```bash
kubectl get pods -n kafka
kubectl get svc -n kafka
```

---

## ⚙️ Configuration (`values.yaml`)

```yaml
statefulset:
  name: kafka
  namespace: kafka
  replicas: 3
  appname: kafka-app
  serviceName: kafka-svc
  image: doughgle/kafka-kraft
  ports:
    - containerPort: 9092
    - containerPort: 9093
  env:
    - name: REPLICAS
      value: '3'
    - name: SERVICE
      value: kafka-svc
    - name: NAMESPACE
      value: kafka
    - name: SHARE_DIR
      value: /mnt/kafka
    - name: CLUSTER_ID
      value: bXktY2x1c3Rlci0xMjM0NQ==
    - name: DEFAULT_REPLICATION_FACTOR
      value: '3'
    - name: DEFAULT_MIN_INSYNC_REPLICAS
      value: '2'

service:
  name: kafka-svc
  namespace: kafka
  replicas: 3
  appname: kafka-app
  type: NodePort
  ports:
    - name: "9092"
      port: 9092
      targetPort: 9092
      nodePort: 30092
```

---

## 🔁 CI/CD Pipeline Overview

The CI/CD pipeline is defined in `.github/workflows/ci-cd.yaml`.

### 📌 Trigger

* On push to `master` branch

### 🧪 Steps

1. ✅ Check out the repository
2. 🐳 Set up Minikube (in GitHub Actions runner)
3. 🔧 Install and configure `kubectl`
4. 🔐 Load `KUBECONFIG_CONTENT` from secret
5. 🚀 Install Kafka via Helm
6. 📦 Create `test-topic`
7. 📨 Send a message (`Hello, Kafka!`)
8. 📬 Consume the message from another pod

### 📄 Key Snippet

```yaml
- name: Create test-topic on pod
  run: |
    kubectl exec kafka-0 -n kafka -- /bin/bash -c "kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1"

- name: Send message to Kafka topic
  run: |
    kubectl exec kafka-0 -n kafka -- /bin/bash -c "echo 'Hello, Kafka!' | kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092"

- name: Consume message from Kafka topic
  run: |
    kubectl exec kafka-1 -n kafka -- /bin/bash -c "kafka-console-consumer.sh --topic test-topic --bootstrap-server localhost:9092 --from-beginning --max-messages 1"
```

---

## 🧹 Cleanup

To uninstall Kafka:

```bash
helm uninstall kafka -n kafka
kubectl delete namespace kafka
```

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 🙋‍♂️ Questions or Contributions?

Feel free to open an issue or submit a pull request if you'd like to improve this setup.

```

---

Would you like me to export this as a downloadable `README.md` file so you can commit it directly to your repo?
```
