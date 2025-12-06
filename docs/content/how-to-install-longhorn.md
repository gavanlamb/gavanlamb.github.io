+++
title = 'How to Install Longhorn on Kubernetes'
date = 2024-12-06T12:24:00Z
draft = false
description = 'A comprehensive guide on installing Longhorn, a distributed block storage system for Kubernetes'
tags = ['kubernetes', 'longhorn', 'storage', 'devops']
categories = ['tutorials', 'kubernetes']
+++

# How to Install Longhorn on Kubernetes

Longhorn is a lightweight, reliable, and easy-to-use distributed block storage system for Kubernetes. This guide will walk you through the installation process.

## Prerequisites

Before installing Longhorn, ensure your Kubernetes cluster meets the following requirements:

### Software Requirements
- Kubernetes v1.21 or higher
- `kubectl` configured to communicate with your cluster
- `open-iscsi` installed on all nodes
- `nfs-common` or `nfs-utils` installed on all nodes (for NFS support)

### Node Requirements
- At least 3 worker nodes for production deployments
- Each node should have:
  - At least 1 CPU core
  - At least 1GB of RAM
  - A filesystem that supports file extents (ext4, XFS)

### Installing Prerequisites on Nodes

For Ubuntu/Debian:
```bash
sudo apt-get update
sudo apt-get install -y open-iscsi nfs-common
sudo systemctl enable --now iscsid
```

For RHEL/CentOS:
```bash
sudo yum install -y iscsi-initiator-utils nfs-utils
sudo systemctl enable --now iscsid
```

## Installation Methods

There are three primary methods to install Longhorn:

### Method 1: Using kubectl (Recommended)

This is the simplest method for most users.

1. **Install Longhorn**:
```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/longhorn.yaml
```

2. **Verify the installation**:
```bash
kubectl get pods -n longhorn-system
```

Wait until all pods are in the `Running` state.

3. **Check deployment**:
```bash
kubectl -n longhorn-system get pod
```

### Method 2: Using Helm

Helm provides more flexibility for customization during installation.

1. **Add the Longhorn Helm repository**:
```bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
```

2. **Install Longhorn**:
```bash
helm install longhorn longhorn/longhorn \
  --namespace longhorn-system \
  --create-namespace \
  --version 1.5.3
```

3. **Customize installation (optional)**:
Create a `values.yaml` file with your custom configuration:
```yaml
defaultSettings:
  backupTarget: s3://backups@us-east-1/
  backupTargetCredentialSecret: aws-secret
persistence:
  defaultClass: true
  defaultClassReplicaCount: 3
```

Then install with:
```bash
helm install longhorn longhorn/longhorn \
  --namespace longhorn-system \
  --create-namespace \
  --values values.yaml
```

### Method 3: Using Rancher

If you're using Rancher for cluster management:

1. Navigate to your cluster in the Rancher UI
2. Go to **Apps & Marketplace**
3. Search for "Longhorn"
4. Click **Install**
5. Configure settings as needed
6. Click **Install** to deploy

## Accessing the Longhorn UI

After installation, you can access the Longhorn UI:

1. **Port-forward the service**:
```bash
kubectl port-forward -n longhorn-system svc/longhorn-frontend 8080:80
```

2. **Access the UI**:
Open your browser and navigate to: `http://localhost:8080`

### Exposing Longhorn UI (Production)

For production environments, you can expose the UI using an Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: longhorn-system
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
spec:
  rules:
  - host: longhorn.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: longhorn-frontend
            port:
              number: 80
```

## Verification Steps

1. **Check all pods are running**:
```bash
kubectl get pods -n longhorn-system
```

All pods should be in the `Running` state.

2. **Verify StorageClass creation**:
```bash
kubectl get storageclass
```

You should see `longhorn` in the list.

3. **Test with a sample workload**:
Create a test PVC:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 1Gi
```

Apply and verify:
```bash
kubectl apply -f test-pvc.yaml
kubectl get pvc longhorn-test-pvc
```

The PVC should be in the `Bound` state.

## Setting Longhorn as Default StorageClass

To make Longhorn the default StorageClass:

```bash
kubectl patch storageclass longhorn -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

If you have an existing default StorageClass, remove it first:
```bash
kubectl patch storageclass <old-default-class> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

## Configuration Options

### Replica Count
Set the default number of replicas for volumes:
```bash
kubectl -n longhorn-system edit settings.longhorn.io default-replica-count
```

### Storage Path
By default, Longhorn stores data in `/var/lib/longhorn/`. To change this:
```bash
kubectl -n longhorn-system edit settings.longhorn.io default-data-path
```

### Backup Configuration
Configure S3-compatible backup targets:
```bash
kubectl -n longhorn-system edit settings.longhorn.io backup-target
```

## Upgrading Longhorn

To upgrade Longhorn to a newer version:

**Using kubectl**:
```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v<NEW_VERSION>/deploy/longhorn.yaml
```

**Using Helm**:
```bash
helm repo update
helm upgrade longhorn longhorn/longhorn --namespace longhorn-system --version <NEW_VERSION>
```

## Uninstalling Longhorn

If you need to uninstall Longhorn:

**Using kubectl**:
```bash
kubectl delete -f https://raw.githubusercontent.com/longhorn/longhorn/v1.5.3/deploy/longhorn.yaml
```

**Using Helm**:
```bash
helm uninstall longhorn -n longhorn-system
kubectl delete namespace longhorn-system
```

**Important**: Make sure to backup any important data before uninstalling!

## Troubleshooting

### Pods stuck in Pending state
Check node prerequisites:
```bash
kubectl describe pod <pod-name> -n longhorn-system
```

### Volume attachment issues
Check iSCSI service:
```bash
sudo systemctl status iscsid
```

### Performance issues
Review the Longhorn documentation on [performance tuning](https://longhorn.io/docs/latest/best-practices/).

## Additional Resources

- [Official Longhorn Documentation](https://longhorn.io/docs/)
- [Longhorn GitHub Repository](https://github.com/longhorn/longhorn)
- [Longhorn Community Slack](https://slack.cncf.io/)
- [Best Practices Guide](https://longhorn.io/docs/latest/best-practices/)

## Conclusion

Longhorn provides a robust, cloud-native storage solution for Kubernetes. With its easy installation process and rich feature set, it's an excellent choice for persistent storage needs in your cluster.

Remember to:
- Regularly backup your data
- Monitor your storage usage through the Longhorn UI
- Keep Longhorn updated to the latest stable version
- Review and apply security best practices

Happy storage managing! 🚀
