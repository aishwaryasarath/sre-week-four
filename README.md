# Release engineering at UpCommerce

## Deploy the stable version

Start minikube and create the namespace
```
minikube start
kubectl create ns sre
```

Create the deployment & service 
```
helm install upcommerce ./upcommerce -n sre
```
<img width="709" alt="image" src="https://github.com/aishwaryasarath/sre-week-four/assets/49971693/dfa4a3a8-19bf-4665-bf2d-ee612a769a9c">

Confirm the deployment
```
kubectl get deployment -n sre
```
or check the pod
```
kubectl get po -n sre
```
<img width="610" alt="image" src="https://github.com/aishwaryasarath/sre-week-four/assets/49971693/94513d5f-5d3d-486b-a2e3-689702f5c3f7">


## Create canary version

`canary-deployment.yml` 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-canary-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-canary-app
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-canary-app
    spec:
      containers:
        - name: canary
          image: {{ .Values.canary.image }}
          ports:
            - containerPort: 5003
          imagePullPolicy: {{ .Values.imagePullPolicy }}
          resources:
            limits:
              cpu: {{ .Values.cpuLimit }}
              memory: {{ .Values.memoryLimit }}
```

`canary-service.yml` 
```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-canary-service
spec:
  selector:
    app: {{ .Release.Name }}-canary-app
  ports:
    - protocol: TCP
      port: 5003
      targetPort: 5003

```

Update values.yaml to append canary image from [dockerhub](https://hub.docker.com/r/uonyeka/canary/tags)
```
canary:
  image: uonyeka/canary:linux-amd64
```


## Deploy the canary 
```
helm upgrade upcommerce ./upcommerce -n sre
```
<img width="734" alt="image" src="https://github.com/aishwaryasarath/sre-week-four/assets/49971693/0a2b7639-bf6e-45a8-a938-006d93ab62b0">


## In the event that the canary fails, shut it down and rollback to the previous stable condition of UpCommerce's deployment before the canary release.

Check deployment
```
kubectl get deployment -n sre
```
or list the pod
<img width="576" alt="image" src="https://github.com/aishwaryasarath/sre-week-four/assets/49971693/4a74e11b-5661-4235-bff7-49e688001e5b">


List helm releases
```
helm list -a
```
<img width="1020" alt="image" src="https://github.com/aishwaryasarath/sre-week-four/assets/49971693/bb45c343-318b-47e8-9f43-64d1e29a3794">


### Rollback to stable release
```
helm rollback upcommerce 2 -n sre
```
<img width="667" alt="image" src="https://github.com/aishwaryasarath/sre-week-four/assets/49971693/e406dabc-b0af-49cb-b9d2-123a99526711">

Confirm rollback
```
helm list -a
```
<img width="1066" alt="image" src="https://github.com/aishwaryasarath/sre-week-four/assets/49971693/e09f6286-85db-461e-b2aa-ae5f63d45318">
