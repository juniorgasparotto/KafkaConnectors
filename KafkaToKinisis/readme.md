# Introdução

Conecta o Kafka ao AWS Kinisis

# Conector

O conector usado será: https://github.com/awslabs/kinesis-kafka-connector

Siga os passos do conector para gerar o JAR caso queira modifica-lo.

# Passos para gerar a imagem docker

```bash
# Build image - DEV
docker build . -t kafka-connect:latest --build-arg env=dev

# Build image - Prod
docker build . -t kafka-connect:latest --build-arg env=prod

# Run with docker-compose
docker-compose up -d

# Destroy container
docker kill kafka-connect

# view logs container
docker logs kafkatokinisis_kafka-connect_1 --follow
docker logs kafkatokinisis_kafka_1 --follow

# Open kafka container
docker exec -it kafkatokinisis_kafka_1 bash
/opt/bitnami/kafka/bin/kafka-topics.sh --list --zookeeper zookeeper:2181
/opt/bitnami/kafka/bin/kafka-console-producer.sh --broker-list kafka:9092 --topic test
```

# Deploy no Kubernates

```bash
# Build image - Prod
docker build . -t kafka-connect:latest --build-arg env=prod

# Tagging
docker tag kafka-connect:latest [ID].dkr.ecr.us-east-1.amazonaws.com/kafka-connect:latest

# Connect ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [ID].dkr.ecr.us-east-1.amazonaws.com

# Push image
docker push [ID].dkr.ecr.us-east-1.amazonaws.com/kafka-connect:latest

# Apply YAMLs
export AWS_PROFILE=DEV
aws eks --region us-east-1 update-kubeconfig --name dev-front

kubectl apply -f ./kubernetes/namespace.yml
kubectl apply -f ./kubernetes/configmap.yml
kubectl apply -f ./kubernetes/secrets.yml
kubectl apply -f ./kubernetes/deployment.yml

kubectl config set-context --current --namespace=ns-kafka-connect
kubectl get pods
kubectl logs "" -c kafka-connect --follow
kubectl delete pod ""
kubectl exec -it "" bash

/opt/kafka/bin/kafka-console-producer.sh --broker-list kafka-url:9092 --topic test
```