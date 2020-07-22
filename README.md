# TEST

We're trying to read value from aliased subchart:

`test-umbrella/Chart.yaml`

```yaml
dependencies:
  - name: "test-subchart"
    repository: "file://../test-subchart"
    version: "0.1.0"
    alias: "alias-subchart"
```

`test-umbrella/templates/deployment.yaml`

```yaml
env:
  - name: "TEST"
    value: {{ index .Values "alias-subchart" "image" "repository" | quote }}
```

## Lint fails

```bash
cd test-umbrella
helm lint .
==> Linting .
[INFO] Chart.yaml: icon is recommended
[ERROR] templates/: template: test-umbrella/templates/deployment.yaml:38:24: executing "test-umbrella/templates/deployment.yaml" at <index .Values "alias-subchart" "image" "repository">: error calling index: index of nil pointer

Error: 1 chart(s) linted, 1 chart(s) failed
```

## Template passes

```bash
helm template test .
```

```yaml
---
# Source: test-umbrella/charts/alias-subchart/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: test-alias-subchart
  labels:
    helm.sh/chart: alias-subchart-0.1.0
    app.kubernetes.io/name: alias-subchart
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
---
# Source: test-umbrella/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: test-test-umbrella
  labels:
    helm.sh/chart: test-umbrella-0.1.0
    app.kubernetes.io/name: test-umbrella
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
---
# Source: test-umbrella/charts/alias-subchart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: test-alias-subchart
  labels:
    helm.sh/chart: alias-subchart-0.1.0
    app.kubernetes.io/name: alias-subchart
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: alias-subchart
    app.kubernetes.io/instance: test
---
# Source: test-umbrella/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: test-test-umbrella
  labels:
    helm.sh/chart: test-umbrella-0.1.0
    app.kubernetes.io/name: test-umbrella
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: test-umbrella
    app.kubernetes.io/instance: test
---
# Source: test-umbrella/charts/alias-subchart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-alias-subchart
  labels:
    helm.sh/chart: alias-subchart-0.1.0
    app.kubernetes.io/name: alias-subchart
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: alias-subchart
      app.kubernetes.io/instance: test
  template:
    metadata:
      labels:
        app.kubernetes.io/name: alias-subchart
        app.kubernetes.io/instance: test
    spec:
      serviceAccountName: test-alias-subchart
      securityContext:
        {}
      containers:
        - name: alias-subchart
          securityContext:
            {}
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}
---
# Source: test-umbrella/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-test-umbrella
  labels:
    helm.sh/chart: test-umbrella-0.1.0
    app.kubernetes.io/name: test-umbrella
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: test-umbrella
      app.kubernetes.io/instance: test
  template:
    metadata:
      labels:
        app.kubernetes.io/name: test-umbrella
        app.kubernetes.io/instance: test
    spec:
      serviceAccountName: test-test-umbrella
      securityContext:
        {}
      containers:
        - name: test-umbrella
          securityContext:
            {}
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          env:
            - name: "TEST"
              value: "nginx"
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}
---
# Source: test-umbrella/charts/alias-subchart/templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "test-alias-subchart-test-connection"
  labels:
    helm.sh/chart: alias-subchart-0.1.0
    app.kubernetes.io/name: alias-subchart
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
  annotations:
    "helm.sh/hook": test-success
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['test-alias-subchart:80']
  restartPolicy: Never
---
# Source: test-umbrella/templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "test-test-umbrella-test-connection"
  labels:
    helm.sh/chart: test-umbrella-0.1.0
    app.kubernetes.io/name: test-umbrella
    app.kubernetes.io/instance: test
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
  annotations:
    "helm.sh/hook": test-success
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['test-test-umbrella:80']
  restartPolicy: Never
```
