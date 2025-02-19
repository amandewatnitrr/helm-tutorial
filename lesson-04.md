# Templates Deep Dive

![](./imgs/Make-Kubernetes-Work-for-you-with-KubeOps.svg)

- Helm uses the Go Programming Language's `text/template` and `html/template` packages/ Text templating engine to render templates.

- Anything within {{ and }} is a template action, which is part of the Go templating syntax.

## Template Actions

- One of the most basic and most used Helm Templating Syntax is actions.

- Action start and end with `{{` and `}}` respectively.

- Within these actions, we can use several other elements from the helm templating syntax, defining variables using conditional logic, if using loops invoking functions, and more.

- Whatever we have inside the `{{` and `}}` is rendered by the templating engine dynamically.

>[!NOTE]
>The `-` hyphen symbol is used in the begining of template to remove any trailing or leadin whitespaces.

- Let's start by playing around with `deployment.yaml` file.

  <details>
  <summary>deployment.yaml</summary><br>

  ```yaml
    apiVersion: apps/v1
    kind: Deployment
    {{"Helm Templating is " -}}, {{- "Cool"}}
    metadata:
    name: {{ include "test-chart.fullname" . }}
    labels:
        {{- include "test-chart.labels" . | nindent 4 }}
    spec:
    {{- if not .Values.autoscaling.enabled }}
    replicas: {{ .Values.replicaCount }}
    {{- end }}
    selector:
        matchLabels:
        {{- include "test-chart.selectorLabels" . | nindent 6 }}
    template:
        metadata:
        {{- with .Values.podAnnotations }}
        annotations:
            {{- toYaml . | nindent 8 }}
        {{- end }}
        labels:
            {{- include "test-chart.labels" . | nindent 8 }}
            {{- with .Values.podLabels }}
            {{- toYaml . | nindent 8 }}
            {{- end }}
        spec:
        {{- with .Values.imagePullSecrets }}
        imagePullSecrets:
            {{- toYaml . | nindent 8 }}
        {{- end }}
        serviceAccountName: {{ include "test-chart.serviceAccountName" . }}
        {{- with .Values.podSecurityContext }}
        securityContext:
            {{- toYaml . | nindent 8 }}
        {{- end }}
        containers:
            - name: {{ .Chart.Name }}
            {{- with .Values.securityContext }}
            securityContext:
                {{- toYaml . | nindent 12 }}
            {{- end }}
            image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
            imagePullPolicy: {{ .Values.image.pullPolicy }}
            ports:
                - name: http
                containerPort: {{ .Values.service.port }}
                protocol: TCP
            {{- with .Values.livenessProbe }}
            livenessProbe:
                {{- toYaml . | nindent 12 }}
            {{- end }}
            {{- with .Values.readinessProbe }}
            readinessProbe:
                {{- toYaml . | nindent 12 }}
            {{- end }}
            {{- with .Values.resources }}
            resources:
                {{- toYaml . | nindent 12 }}
            {{- end }}
            {{- with .Values.volumeMounts }}
            volumeMounts:
                {{- toYaml . | nindent 12 }}
            {{- end }}
        {{- with .Values.volumes }}
        volumes:
            {{- toYaml . | nindent 8 }}
        {{- end }}
        {{- with .Values.nodeSelector }}
        nodeSelector:
            {{- toYaml . | nindent 8 }}
        {{- end }}
        {{- with .Values.affinity }}
        affinity:
            {{- toYaml . | nindent 8 }}
        {{- end }}
        {{- with .Values.tolerations }}
        tolerations:
            {{- toYaml . | nindent 8 }}
        {{- end }}
    ```

  Run the command:

  ```bash
  $ helm template test-chart
  ```

  Now in the output, look for deployments section and there you would find something like this:

    ```yaml
    # Source: test-chart/templates/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    Helm Templating is ,Cool
    metadata:
    name: release-name-test-chart
    labels:
        helm.sh/chart: test-chart-0.1.0
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
        app.kubernetes.io/version: "1.16.0"
        app.kubernetes.io/managed-by: Helm
    spec:
    replicas: 1
    ...
    ```

  And, here we can clearly see that the `{{"Helm Templating is " -}}, {{- "Cool"}}` is rendered as `Helm Templating is Cool`.

  All the unnecessary whitespaces are removed by using `-` hyphen symbol.
