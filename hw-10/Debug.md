# Kubernetes: Debug

Дебаг (отладка) приложений в Kubernetes — это процесс анализа и устранения проблем, возникающих в рабочих приложениях, подах, сервисах или других компонентах кластера. Kubernetes предоставляет инструменты и методы для эффективного выявления и решения проблем на уровне инфраструктуры, контейнеров и сетевого взаимодействия.

---

### Для чего нужен дебаг в Kubernetes?

1. **Выявление неисправностей**:
   - Проблемы с подами (ошибки запуска, сбои контейнеров).
   - Ошибки сетевого взаимодействия (недоступные сервисы, неверные маршруты).
   - Неправильная конфигурация (секреты, монтирование томов).
   
2. **Диагностика производительности**:
   - Высокое потребление ресурсов (CPU, памяти).
   - Проблемы с задержкой или пропускной способностью сети.

3. **Отладка отказоустойчивости**:
   - Проблемы с масштабированием или балансировкой нагрузки.
   - Неудачи при развертывании обновлений.

---

### Основные инструменты и команды для отладки

1. **`kubectl describe`**:
   - Предоставляет подробную информацию о ресурсе.
   - Пример:
     ```bash
     kubectl describe pod <pod-name>
     ```
   - Используется для просмотра событий, связанных с подом, например, отказов запуска.

2. **`kubectl logs`**:
   - Просмотр логов контейнера.
   - Пример:
     ```bash
     kubectl logs <pod-name>
     ```
   - Для многоконтейнерного пода:
     ```bash
     kubectl logs <pod-name> -c <container-name>
     ```

3. **`kubectl exec`**:
   - Выполнение команды внутри работающего контейнера.
   - Пример:
     ```bash
     kubectl exec -it <pod-name> -- /bin/bash
     ```

4. **`kubectl get events`**:
   - Отображение событий в пространстве имен.
   - Пример:
     ```bash
     kubectl get events --sort-by='.metadata.creationTimestamp'
     ```

5. **`kubectl port-forward`**:
   - Перенаправление порта для доступа к приложению, запущенному в поде.
   - Пример:
     ```bash
     kubectl port-forward pod/<pod-name> 8080:80
     ```

6. **`kubectl debug`** (начиная с Kubernetes 1.20):
   - Создание временного пода для отладки проблем.
   - Пример:
     ```bash
     kubectl debug pod/<pod-name> --image=busybox
     ```

7. **`kubectl cp`**:
   - Копирование файлов между локальной машиной и контейнером.
   - Пример:
     ```bash
     kubectl cp <pod-name>:<path-in-container> <local-path>
     ```

---

### Частые сценарии отладки

1. **Проблемы с подами**:
   - **CrashLoopBackOff**:
     - Посмотреть логи: `kubectl logs <pod-name>`.
     - Проверить события: `kubectl describe pod <pod-name>`.
   - **ContainerCreating**:
     - Убедиться, что образ доступен: `kubectl describe pod <pod-name>` для ошибок image pull.
     - Проверить состояние PersistentVolume: `kubectl get pvc`.

2. **Сетевые проблемы**:
   - Проверить доступность сервиса:
     ```bash
     kubectl get svc
     kubectl describe svc <service-name>
     ```
   - Использовать `kubectl port-forward` для проверки локального доступа.
   - Проверить DNS внутри пода:
     ```bash
     kubectl exec -it <pod-name> -- nslookup <service-name>
     ```

3. **Проблемы с ресурсами**:
   - Проверить состояние ресурсов:
     ```bash
     kubectl top pod
     kubectl top node
     ```
   - Убедиться, что поды правильно распределены по узлам:
     ```bash
     kubectl get pods -o wide
     ```

---

### Best Practices для отладки приложений

1. **Подготовка и мониторинг**:
   - Используйте инструменты мониторинга, такие как Prometheus, Grafana, ELK/EFK.
   - Включите централизованный сбор логов.

2. **Организация логирования**:
   - Используйте стандартный вывод (`stdout`/`stderr`) для логов контейнеров.
   - Настройте ротацию логов для предотвращения переполнения.

3. **Резервное копирование конфигурации**:
   - Всегда сохраняйте и документируйте настройки манифестов Kubernetes.

4. **Снижение времени на диагностику**:
   - Добавляйте метки (`labels`) и аннотации (`annotations`) к объектам для упрощения поиска.
   - Настройте readiness и liveness-пробы.

5. **Работа с сетью**:
   - Используйте сетевые политики (Network Policies) для упрощения отладки доступов.
   - Применяйте утилиты вроде `curl`, `ping` внутри пода для проверки сетевого соединения.

6. **Используйте инструмент `kubectl debug`**:
   - Для отладки сложных случаев создавайте временные поды с инструментами диагностики.

7. **Минимизация нагрузки**:
   - Используйте лимиты ресурсов (`requests`/`limits`) для предотвращения потребления всех ресурсов.

8. **Деление на этапы**:
   - Диагностируйте сначала состояние API-сервера и мастера.
   - Проверяйте сетевую подсистему (CNI).
   - Анализируйте поды и контейнеры.

---

### Пример использования `kubectl debug`

Если под не может стартовать из-за ошибки в конфигурации, вы можете запустить отладочный контейнер с инструментами:

```bash
kubectl debug pod/<pod-name> --image=busybox --share-processes --copy-to=debugger
kubectl exec -it debugger -- /bin/sh
```

Это создаст отладочный под с образом `busybox`, где вы сможете проанализировать состояние пода.

---

### Заключение

Дебаг приложений в Kubernetes — это неотъемлемая часть работы с кластером. Используя предоставленные инструменты и подходы, вы можете эффективно решать проблемы, выявлять узкие места и повышать надежность ваших приложений. Следуя best practices, вы обеспечите быструю диагностику и предотвращение проблем в будущем.