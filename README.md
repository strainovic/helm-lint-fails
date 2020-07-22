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
...
          env:
            - name: "TEST"
              value: "nginx"
...
```
