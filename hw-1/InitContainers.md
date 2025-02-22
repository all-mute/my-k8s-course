# Kubernetes: *InitContainers*

`InitContainers` — это специальные контейнеры в Kubernetes, которые запускаются перед основными контейнерами (`app containers`) в поде. `InitContainers` часто используются для подготовки среды или выполнения предварительных задач, необходимых для работы основного приложения. Они помогают гарантировать, что приложение будет запущено в правильных условиях.

### Основные особенности InitContainers:
1. **Запускаются последовательно**: Все `InitContainers` в поде запускаются последовательно. Каждый контейнер должен завершиться успешно, прежде чем начнется выполнение следующего.
2. **Выполняются перед основными контейнерами**: Основные контейнеры пода не начнут выполняться до тех пор, пока все `InitContainers` не завершатся.
3. **Могут использовать свои образы и настройки**: `InitContainers` могут использовать другой Docker-образ, чем основной контейнер, и иметь собственные параметры, такие как сетевые настройки, команды, переменные окружения и пр.
4. **Повторные попытки при ошибках**: Если `InitContainer` завершился с ошибкой, Kubernetes будет пытаться его перезапустить, пока он не завершится успешно.

### Примеры использования InitContainers:
- **Подготовка данных**: Загрузка конфигурационных файлов или других данных перед запуском приложения.
- **Проверка условий**: Проверка зависимостей или условий (например, доступность базы данных) перед запуском основного контейнера.
- **Права и настройки окружения**: Установка разрешений или монтирование файлов перед началом работы основного контейнера.

### Пример использования InitContainers
Предположим, у нас есть веб-сервер, которому нужен статический файл для старта. Мы можем использовать `InitContainer`, чтобы загрузить этот файл из удаленного источника перед запуском основного контейнера.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  initContainers:
  - name: init-download
    image: busybox
    command: ['sh', '-c', 'wget -O /data/index.html http://example.com/index.html']
    volumeMounts:
    - mountPath: /data
      name: data-volume

  containers:
  - name: web-server
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: data-volume

  volumes:
  - name: data-volume
    emptyDir: {}
```
