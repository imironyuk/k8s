## mindbox.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels: # использую рекомендованные https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/
    app.kubernetes.io/name: nginx
    app.kubernetes.io/version: 1.23.1-alpine
    app.kubernetes.io/component: website
spec:
  replicas: 4 # желаемое количество реплик приложения
  selector:
    matchLabels: # какими pod управляет этот Deployment (у которых есть лейбл что ниже)
      app.kubernetes.io/name: nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx # label по которому Kubernetes будем потом выбирать на какую node разместить pod
    spec:
      affinity: # переводится как 'близость', то есть ниже приведена anti близость, то есть не близость
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution: # желательное правило рпаспределения pod оболочек
          - weight: 100 # от 0 до 100, это имеет смысл, когда есть несколько правил, и нужно выбрать, какое из них важнее
            podAffinityTerm: # анти аффинити значит что распологать на node на которых нет лейбла app.kubernetes.io/name=nginx
              labelSelector:
                matchExpressions:
                - key: app.kubernetes.io/name
                  operator: In
                  values:
                  - nginx
              topologyKey: "topology.kubernetes.io/zone" # чтобы планировщик пытался расположить реплики приложения в разных зонах
      terminationGracePeriodSeconds: 60 # время которое отводится на остановку Pod перед его завершением
      containers:
      - name: nginx
        image: nginx:1.23.1-alpine # образ nginx используя более безопасную и маленькую в размере версию alpine linux
        ports:
        - name: http
          containerPort: 80
        resources: 
          requests: # запрашиваемые ресурсы контейнера, по сути необходимый минимум ресурсов свободных на node
            memory: "150Mi"
            cpu: "150m"
          limits: # пределы ресурсов которые может использовать контейнер
            memory: "150Mi" # попробует больше памяти то будет убить OOMkilled
            cpu: "500m" # больше ресурсов процессора -> попадет в очередь, не будет убит
        livenessProbe: # постоянный тест работоспособности контейнера чтобы если он выйдет из строя перезапустить его
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        readinessProbe: # не отправлять трафик на pod пока не отработает эта проверка готовности
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

