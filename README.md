# Helm Chart Cheat Sheet

This cheat sheet covers everything you need to know to get started with Helm charts.

## Basic Helm Commands

| Command | Description |
|---------|-------------|
| `helm create <chart-name>` | Create a new Helm chart |
| `helm install <chart-name> <release-name>` | Install a Helm chart |
| `helm upgrade <release-name> <chart-name>` | Upgrade a Helm release |
| `helm rollback <release-name> <revision-number>` | Rollback a Helm release to a previous revision |
| `helm uninstall <release-name>` | Uninstall a Helm release |
| `helm list` | List all Helm releases |
| `helm status <release-name>` | Show the status of a Helm release |

## Helm Chart Structure

A Helm chart is a collection of files and directories that define a Kubernetes application. The basic structure of a Helm chart includes:

```PowerShell
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

- `Chart.yaml`: A YAML file that contains metadata about the chart, such as its name, version, and description.
- `values.yaml`: A YAML file that contains default values for the chart's templates.
- `templates/`: A directory that contains the chart's templates. Templates are used to generate Kubernetes manifests.

## Advanced Helm Commands

| Command | Description |
|---------|-------------|
| `helm dependency build <chart-name>` | Build the chart's dependencies |
| `helm dependency list <chart-name>` | List the chart's dependencies |
| `helm lint <chart-name>` | Check a chart's syntax for errors |
| `helm package <chart-name>` | Package a chart into a versioned archive |
| `helm repo add <repository-name> <repository-url>` | Add a Helm chart repository |
| `helm repo list` | List all added chart repositories |
| `helm repo remove <repository-name>` | Remove a chart repository |
| `helm repo update` | Update the local cache of Helm chart repositories |
| `helm search <chart-name>` | Search for a chart in the local or remote repository |
| `helm sign <chart-name>` | Sign a chart with PGP private key |
| `helm verify <chart-name>` | Verify the signature of a signed chart |

## Using Values Files

Values files are used to configure the chart's templates. Here are some common operations:

| Command | Description |
|---------|-------------|
| `helm install <chart-name> --values <values-file>` | Install a chart with a specific values file |
| `helm install <chart-name> --set <key=value>` | Override a value in a values file |
| `helm upgrade <release-name> <chart-name> --values <values-file>` | Upgrade a release with a specific values file |
| `helm upgrade <release-name> <chart-name> --set <key=value>` | Override a value in an existing release |

## Loops and Conditionals in Helm Charts

Helm templates support a simple templating language that allows loops, conditionals, and other operations.

### Loops
Loops can be used to generate multiple copies of a template with different values. The range function is used to iterate over a list or map. Here's an example of a loop that generates a Kubernetes Deployment resource for each item in a list:

```yaml
# deployment.yaml

{{- range .Values.services }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .name }}-deployment
spec:
  replicas: {{ .replicaCount }}
  selector:
    matchLabels:
      app: {{ .name }}
  template:
    metadata:
      labels:
        app: {{ .name }}
    spec:
      containers:
      - name: {{ .name }}
        image: {{ .image.repository }}:{{ .image.tag }}
        imagePullPolicy: {{ .image.pullPolicy }}
{{- end }}
```
In this example, the range function iterates over the services list, which is defined in the values file. It generates a Deployment resource for each item in the list, using the values defined for that item.

### Conditionals
Conditionals can be used to include or exclude parts of a template based on the value of a condition. The if function is used to test the condition. Here's an example of a conditional that includes a Kubernetes Service resource if the enableService value is true:

```yaml
# service.yaml

{{- if .Values.enableService }}
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  ports:
    - port: {{ .Values.servicePort }}
      targetPort: {{ .Values.containerPort }}
  selector:
    app: my-app
{{- end }}
```
In this example, the if function tests the value of enableService, which is defined in the values file. If it's true, the Service resource is included in the template.

### values.yaml and chart.yaml files Syntax

The values file is a YAML file that contains key-value pairs. You can override any value in the values file with the `--set` or `--values` flag.

- values.yaml
```yaml
replicaCount: 3
image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent

```

In this example, replicaCount is a value that can be overridden at installation time, while image is a nested value that can be overridden at a more granular level (e.g., image.repository).

- Chart.yaml
```yaml
# chart.yaml

apiVersion: v2
name: mychart
description: A Helm chart for my application
version: 0.1.0
```

### Scope
Helm provides the ability to control the scope of a variable using the global and local keywords. Here is an example:


```yaml
# values.yaml
global:
  globalVar: "I'm a global variable"

config:
  configVar: "I'm a local variable"
```

In this example, globalVar is a global variable that can be accessed from any template or chart. configVar is a local variable that is only accessible within the scope of the config block.

### Dependency
```yaml
# Chart.yaml
dependencies:
  - name: mysql
    version: 1.2.3
    repository: https://example.com/charts
```
In this example, the chart has a dependency on the mysql chart at version 1.2.3 from the repository at https://example.com/charts.

### Hooks
Helm provides the ability to define hooks that are run at specific points during the installation and upgrading process. Here is an example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-hook
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "10"
  labels:
    app.kubernetes.io/name: {{ include "my-chart.fullname" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: my-hook
          image: alpine:latest
          command: ["/bin/sh"]
          args: ["-c", "echo 'Running pre-install/pre-upgrade hook'"]
  backoffLimit: 0

```


In this example, the hook is a Job that runs before the installation process. The job runs a container that outputs a message to the console.
Where, **helm.sh/hook:** This annotation specifies the type of hook that the resource represents. In this case, it's a pre-upgrade hook, meaning that it will be executed before the chart is upgraded.

### Range with blocks

Helm provides the ability to define blocks that are only rendered if the condition is true. Here is an example:

```yaml
# templates/deployment.yaml
{{- if and (eq .Values.image.tag "latest") (eq .Values.image.pullPolicy "Always") }}
spec:
  template:
    spec:
      imagePullSecrets:
        - name: {{ .Values.image.pullSecrets }}
{{- end }}
```
In this example, the image pull secrets are only rendered if the image tag is latest and the pull policy is Always.

### Named templates
Helm provides the ability to define named templates that can be called from within other templates. Here is an example:

```yaml
---
# templates/_helpers.tpl
{{- define "labels" }}
  app.kubernetes.io/name: {{ .Release.Name }}
  app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-demo
  labels:
    {{ include "labels" . | indent 2 }}
spec:
  selector:
    matchLabels:
      {{ include "labels" . | indent 4 }}
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        {{ include "labels" . | indent 4 }}
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

