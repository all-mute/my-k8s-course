# Kubernetes: Local Persistent Volumes

Для создания томов с динамическим выделением пространства в Kubernetes на локальном хосте можно использовать **Local Persistent Volumes** в сочетании с **StorageClass** и **PersistentVolumeClaim** (PVC). Это позволяет автоматически создавать и монтировать локальные тома к подам, используя локальное хранилище хоста.

### 1. Настройка локального хранилища с динамическим выделением

На локальном хосте вам потребуется предварительно настроить директорию, которая будет использоваться Kubernetes для размещения локальных томов.

### 2. Создание StorageClass для локальных томов

1. **Создайте StorageClass**, который будет указывать на локальное хранилище и поддерживать динамическое выделение.

2. **Создайте директорию на хосте** (например, `/mnt/data`), которая будет использоваться для хранения данных.

Пример StorageClass с локальным хранилищем:

```yaml
# storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

- **provisioner: kubernetes.io/no-provisioner**: Указывает, что Kubernetes не будет автоматически выделять диски, но будет использовать локальные Persistent Volumes.
- **volumeBindingMode: WaitForFirstConsumer**: Том будет выделен только тогда, когда под запросит его. Это позволяет избежать автоматического выделения ресурса.

### 3. Создание PersistentVolume с использованием локального хранилища

Для каждого тома вам нужно будет создать PersistentVolume (PV), указывающий на определенную директорию на хосте.

```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - <имя_вашего_узла>
```

Замените `<имя_вашего_узла>` на имя ноды Kubernetes (его можно узнать с помощью `kubectl get nodes`).

### 4. Создание PersistentVolumeClaim для динамического запроса тома

PersistentVolumeClaim (PVC) запрашивает определенный объем и StorageClass. Когда PVC создается, Kubernetes будет пытаться найти или выделить подходящий том.

```yaml
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 5Gi
```

### Применение файлов манифеста

После создания файлов примените их:

```bash
kubectl apply -f storage-class.yaml
kubectl apply -f persistent-volume.yaml
kubectl apply -f persistent-volume-claim.yaml
```

### Использование тома в поде

Когда `PersistentVolumeClaim` будет готов, вы сможете использовать его в поде:

```yaml
# pod-with-local-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-local-volume
spec:
  containers:
    - name: app-container
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: local-storage
  volumes:
    - name: local-storage
      persistentVolumeClaim:
        claimName: local-pvc
```

Теперь Kubernetes будет динамически монтировать локальные тома, используя StorageClass и PersistentVolumeClaim, которые вы создали.