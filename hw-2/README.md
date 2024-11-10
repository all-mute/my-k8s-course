# Kubernetes: Namespaces

---

В Kubernetes *Namespaces* (пространства имен) — это способ разделения кластера на логические, изолированные сегменты. Namespaces создают виртуальные кластеры внутри одного физического кластера Kubernetes. Они позволяют разделить ресурсы и управлять доступом, что особенно полезно в крупных и мультиарендных средах, где несколько команд или проектов работают в одном кластере.

### Зачем нужны Namespaces?

1. **Изоляция ресурсов**:
   В крупных кластерах, где разные команды могут разрабатывать свои приложения, Namespaces позволяют каждой команде работать со своими ресурсами (подами, сервисами, секретами и т.д.), не мешая друг другу. Каждая команда может работать в отдельном Namespace, что предотвращает конфликты имен (например, два пода с одинаковым именем в разных Namespaces не конфликтуют между собой).

2. **Управление доступом**:
   Kubernetes поддерживает *Role-Based Access Control* (RBAC), что позволяет ограничить доступ к ресурсам в кластере. Например, можно предоставить одной команде доступ только к определенному Namespace, не давая доступа к другим. Это особенно полезно, когда одна команда или отдел не должен иметь доступ к данным или компонентам другой.

3. **Разделение окружений**:
   Namespaces также часто используют для разделения окружений, таких как `development`, `staging`, `production`. Например, вы можете развернуть свои приложения в разных Namespaces, чтобы тестировать их изолированно, избегая влияния на производственные среды.

4. **Управление ресурсами**:
   Kubernetes позволяет установить квоты на использование ресурсов в каждом Namespace (CPU, память, хранилище и т.д.). Это помогает контролировать потребление ресурсов в кластере, чтобы одна команда не могла перегрузить кластер, забрав все ресурсы.

### Как работают Namespaces?

Namespaces в Kubernetes не создают настоящих физических изоляций, как виртуальные машины или контейнеры. Они скорее являются виртуальными разделами на уровне логики кластера. При этом некоторые ресурсы (например, *Nodes* и *Persistent Volumes*) работают на уровне всего кластера и не привязываются к конкретным Namespaces. 

Когда вы создаете объекты в Kubernetes (например, поды, сервисы или конфигурации), они принадлежат определенному Namespace. Если Namespace не указан, то объект по умолчанию размещается в Namespace `default`.

### Встроенные Namespaces

В Kubernetes есть несколько предустановленных Namespaces:
1. **default** — стандартный Namespace, в который попадают все ресурсы, если не указать иной.
2. **kube-system** — Namespace для ресурсов, связанных с самим Kubernetes, таких как компоненты кластера (например, `kube-dns`).
3. **kube-public** — Namespace для общедоступных ресурсов. Обычно это ресурсы, к которым можно получить доступ без специальных разрешений.
4. **kube-node-lease** — Namespace, где Kubernetes хранит лизинговую информацию для узлов, чтобы отслеживать их доступность.

### Пример использования Namespaces

Создание Namespace:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-team
```

Применение:
```bash
kubectl apply -f namespace.yaml
```

После этого вы можете создавать ресурсы внутри Namespace `dev-team`. Например:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  namespace: dev-team  # Указываем Namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:latest
```

### Как взаимодействовать с Namespaces в командной строке?

Для выполнения операций с ресурсами в определенном Namespace в `kubectl` добавляется параметр `--namespace`. Например:
```bash
kubectl get pods --namespace=dev-team
```

Также можно установить Namespace по умолчанию:
```bash
kubectl config set-context --current --namespace=dev-team
```

### Заключение

Namespaces — это эффективный способ управления многопользовательскими, мультиарендными и разнородными кластерами Kubernetes. Они позволяют разделять и ограничивать ресурсы, делая работу с кластером более гибкой, безопасной и упорядоченной.

---

# Kubernetes: Role-Based Access Control (RBAC)

---

В Kubernetes доступ пользователей к ресурсам можно настроить с помощью механизма *Role-Based Access Control* (RBAC). RBAC позволяет определять роли и правила для пользователей и сервисов, указывая, к каким ресурсам в кластере или Namespace они имеют доступ и какие действия могут выполнять. Ниже опишу основные компоненты RBAC и шаги по настройке.

### Основные компоненты RBAC

1. **Role**:
   - Определяет набор разрешений для конкретного Namespace.
   - Включает список ресурсов, действий (чтение, запись и т.д.) и Namespace, в котором они применяются.
   - Применяется только к одному Namespace.

2. **ClusterRole**:
   - Похожа на Role, но действует на уровне всего кластера.
   - Может использоваться в любом Namespace или для глобальных ресурсов кластера (например, nodes).
   - ClusterRole можно назначить как для конкретного Namespace, так и для всех.

3. **RoleBinding**:
   - Привязывает Role к пользователю, группе или сервисному аккаунту в определенном Namespace.
   - RoleBinding разрешает использовать разрешения, определенные Role, для конкретного субъекта в Namespace.

4. **ClusterRoleBinding**:
   - Привязывает ClusterRole к пользователю, группе или сервисному аккаунту на уровне всего кластера или в нескольких Namespaces.
   - Это позволяет дать пользователю доступ к ресурсам вне одного конкретного Namespace.

### Настройка доступа с использованием RBAC

Предположим, нам нужно создать Namespace, в котором пользователь будет иметь определенные права. Далее приведен пример создания и применения ролей и привязок в Namespace.

#### 1. Создание Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-team
```

Применение:
```bash
kubectl apply -f namespace.yaml
```

#### 2. Определение Role для Namespace

Создадим роль, которая разрешает пользователю видеть поды и деплойменты, а также создавать и удалять поды.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team  # Указываем Namespace
  name: dev-role
rules:
  - apiGroups: [""]  # "" означает основную группу API
    resources: ["pods"]
    verbs: ["get", "list", "create", "delete"]  # Действия, разрешенные для подов
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list"]  # Действия, разрешенные для деплойментов
```

Применение:
```bash
kubectl apply -f role.yaml
```

#### 3. Создание RoleBinding для пользователя

Чтобы привязать эту роль к пользователю, создадим RoleBinding, указав имя пользователя или сервисного аккаунта. 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-role-binding
  namespace: dev-team
subjects:
  - kind: User                 # или 'ServiceAccount', если вы хотите использовать сервисный аккаунт
    name: developer-user       # Имя пользователя, которому предоставляем доступ
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role                   # Привязываем к Role
  name: dev-role               # Название Role, которую создавали ранее
  apiGroup: rbac.authorization.k8s.io
```

Применение:
```bash
kubectl apply -f rolebinding.yaml
```

### Ограничение ресурсов в Namespace

Чтобы ограничить ресурсы (например, CPU, память), можно настроить *ResourceQuota*. 

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-team-quota
  namespace: dev-team
spec:
  hard:
    pods: "10"                     # Максимум 10 подов
    requests.cpu: "4"              # Лимит CPU на запросы в 4 ядра
    requests.memory: "8Gi"         # Лимит на запрос памяти в 8 ГБ
    limits.cpu: "8"                # Максимально доступный CPU
    limits.memory: "16Gi"          # Максимально доступная память
```

Применение:
```bash
kubectl apply -f resourcequota.yaml
```

### Пример использования ClusterRole и ClusterRoleBinding

Если нужно предоставить доступ ко всему кластеру, например, для мониторинга ресурсов, можно создать ClusterRole и ClusterRoleBinding.

#### 1. Создание ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: view-nodes
rules:
  - apiGroups: [""] 
    resources: ["nodes"]
    verbs: ["get", "list"]
```

#### 2. Создание ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: view-nodes-binding
subjects:
  - kind: User
    name: monitor-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view-nodes
  apiGroup: rbac.authorization.k8s.io
```

### Проверка доступов пользователя

Чтобы проверить, имеет ли пользователь доступ к определенным ресурсам, можно воспользоваться командой:
```bash
kubectl auth can-i <verb> <resource> --namespace=<namespace> --as=<username>
```

Пример:
```bash
kubectl auth can-i create pods --namespace=dev-team --as=developer-user
```

### Заключение

Namespaces и RBAC позволяют создавать безопасные и гибкие конфигурации, обеспечивая доступ к ресурсам на уровне как всего кластера, так и отдельных пространств имен.