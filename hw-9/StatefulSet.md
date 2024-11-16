# Kubernetes: StatefulSet

---

`StatefulSet` — это объект Kubernetes, предназначенный для управления состоянием приложений, которые требуют сохранения данных и уникальных идентификаторов. В отличие от `Deployment`, который создает идентичные экземпляры (подов), `StatefulSet` управляет подами с уникальными идентификаторами и предоставляет гарантию на порядок создания и удаления подов. 

### Основные особенности StatefulSet

1. **Сохранение состояния (State)**: Поддерживает долгоживущие данные, которые необходимы при перезапуске подов.
2. **Уникальные идентификаторы**: Каждый под в StatefulSet имеет постоянное имя и идентификатор, которые сохраняются даже после перезапуска пода.
3. **Упорядоченные операции**: Создание, обновление и удаление подов выполняются в определенном порядке.
4. **Использование PersistentVolumeClaim (PVC)**: Для каждого пода StatefulSet автоматически создает `PersistentVolumeClaim` для хранения данных. Это позволяет каждому поду получать доступ к своему уникальному хранилищу, которое остается неизменным даже при перезапуске подов.

### Для чего нужен StatefulSet?

StatefulSet подходит для работы с приложениями, которые требуют стабильного хранилища и уникальных идентификаторов для каждого пода. Примеры таких приложений:
- СУБД, такие как MySQL, PostgreSQL.
- Системы кеширования, такие как Redis, Cassandra.
- Распределенные системы и кластеры, такие как Kafka, Zookeeper, ElasticSearch.

### Основные параметры StatefulSet

- **replicas**: Количество подов, которые нужно создать.
- **selector**: Условия для выбора подов, управляемых StatefulSet.
- **serviceName**: Указывает на `Headless Service`, который обслуживает поды StatefulSet. Такой сервис нужен для обеспечения стабильных DNS-имен для каждого пода.
- **volumeClaimTemplates**: Шаблоны для создания PersistentVolumeClaim для каждого пода. PVC создается автоматически и закрепляется за подом.
- **updateStrategy**: Определяет стратегию обновления подов (например, `RollingUpdate` для поочередного обновления).

### Пример манифеста StatefulSet

Ниже приведен пример манифеста для StatefulSet, который разворачивает кластер Redis с тремя подами, каждый из которых имеет собственное постоянное хранилище:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: "redis"
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.0
        ports:
        - containerPort: 6379
          name: redis
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 5Gi
```

### Объяснение манифеста

1. **serviceName**: `Headless Service`, именуемый `redis`, обеспечивает каждому поду стабильный DNS-адрес (например, `redis-0`, `redis-1`, и т.д.).
2. **replicas**: Задает количество подов в StatefulSet (в данном случае 3).
3. **template**: Шаблон пода, который включает в себя контейнер Redis.
4. **volumeClaimTemplates**: Шаблон для `PersistentVolumeClaim`, который создает для каждого пода уникальный диск с объемом `5Gi`.

### Особенности запуска StatefulSet

1. **Порядок создания подов**: Поды создаются последовательно — сначала `redis-0`, затем `redis-1`, и так далее. 
2. **Порядок обновления**: При обновлении StatefulSet поды также обновляются по очереди, начиная с последнего.
3. **Поддержка отказоустойчивости**: Каждый под в StatefulSet получает свой уникальный PVC, что гарантирует сохранность данных, если под перезапускается или мигрирует на другой узел.

### Пример создания Headless Service для StatefulSet

Для работы StatefulSet требуется Headless Service, чтобы назначать уникальные DNS-имена подам. Вот пример манифеста для Headless Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
  - port: 6379
    name: redis
```

Этот Headless Service (с `clusterIP: None`) будет использоваться для доступа к каждому поду StatefulSet, предоставляя стабильные DNS-имена в формате `<имя-пода>.<имя-сервиса>`. Например, `redis-0.redis`, `redis-1.redis`, и так далее.

### Применение и удаление StatefulSet

- Для создания StatefulSet используйте команду:
  ```bash
  kubectl apply -f statefulset.yaml
  ```
- Для удаления StatefulSet:
  ```bash
  kubectl delete -f statefulset.yaml
  ```
  
При удалении StatefulSet по умолчанию удаляет только поды, но оставляет связанные с ними PersistentVolumeClaim (PVC). Это сделано для сохранения данных.