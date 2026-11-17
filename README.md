# Lab 7 - Introduction to Kubernetes Basics
**Name:** Angus MacGillivary<br>
**Course:** CST8915<br>
**Section:** 11 <br> 
**Lab:** 7<br>
**Date:** November 18, 2026<br>

## Video Demo
[Link](https://youtu.be/vroDiXB-Kyc)

## RabbitMQ Configuration Analysis

This deployment of the RabbitMQ service does not include any persistent storage methods, so all messages in the queue must be kept in memory.  This means that message queues are exclusive to a specific run of the service, and will be permanently lost if the container or pod is deleted or restarted.

According to the [documentation](https://learn.microsoft.com/en-us/azure/aks/concepts-storage#persistent-volumes) for AKS, persistent volumes can be declared in the cluster using Azure Files, Azure Container Storage or Azure Disk. The pods can then be bound to the volumes using a [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) in the manifest. In the case of algonquin-pet-store-all-in-one.yaml, we can make the following changes:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rabbitmq-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

---
# 1. RabbitMQ Deployment - runs the message broker used by Order Service.
# Broker data is stored on the rabbitmq-data PersistentVolumeClaim defined above.
# The API group and version identify the Kubernetes API used for this resource.
apiVersion: apps/v1
kind: Deployment
...
        containers:
        ...
            volumeMounts:
                - name: rabbitmq-data
                mountPath: /var/lib/rabbitmq
        volumes:
            - name: rabbitmq-data
            persistentVolumeClaim:
                claimName: rabbitmq-data
```

Alternatively, we can replace the RabbitMQ backing service with Azure's managed messaging service, [Service Bus](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview). Azure Service Bus supports easy scaling and disaster recovery using triple-redundant storage.  It is also totally decoupled from other services and only links up with them through a connection string.

