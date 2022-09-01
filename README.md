## deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
    maintainer: Daniil # просто как пример еще одного label
spec:
  replicas: 4 # количество реплик
  selector:
    matchLabels: # какими pod управляет этот Deployment
      app: nginx
  template:
    metadata:
      labels:
        app: nginx # label по которому Kubernetes будем потом выбирать на какую node разместить pod
    spec:
      affinity:
        preferredDuringSchedulingIgnoredDuringExecution: # желательное правило рпаспределения pod оболочек
        - podAffinityTerm: # анти аффинити значит что распологать на node на которых нет лейбла app=nginx
            labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "topology.kubernetes.io/zone" # для расположения на node в разных зонах доступности
      containers:
      - name: nginx
        image: nginx:1.23.1-alpine # образ nginx используя более безопасную и маленькую в размере версию alpine linux
        ports:
        - name: http
          containerPort: 80
        resources:
          requests:
            memory: "150Mi"
            cpu: "150m"
          limits:
            memory: "150Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 3
          periodSeconds: 3
```
