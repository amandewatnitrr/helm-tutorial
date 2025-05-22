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

    </details>

  Run the command:

  ```bash
  helm template test-chart
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

## Template Information

- When Helm renders a template, it passes all the information for the template as a single object represented by `.` (dot).

>[!IMPORTANT]
> The `.` (dot) is the root object that contains all the information about the template.
> It repesents all the information that a template can use, it has sub object likes values.
> <br/><br/>
> The sub-object values contains all the information from the `values.yaml` file.
> Values is a sub-object of the root object `.` (dot).

- We can also add our own custom values to the `values.yaml` file and use them in the template.

- For this example, as well let's take the `deployment.yaml` file.

    <details>
    <summary>deployment.yaml</summary><br>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    {{.Values.my.custom.data }}
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

    </details>

    Now, run the following command:

    ```bash
    helm template test-chart
    ```

    The output of the command `helm template test-chart` would be somethings like this:

    ```yaml
    ---
    # Source: test-chart/templates/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
      test
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

- The `.` (dot) object also has several other sub-objects if you want to access the chart information in one of your templates, we can do that using the Chart object.

  The code will look something like this:

  ```yaml
  {{.Chart.Version}}
  {{.Chart.Name}}
  {{.Chart.AppVersion}}
  {{.Chart.Annotations}}
  ```

  <details>
  <summary>deployment.yaml</summary><br>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
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

    </details>

    Now, run the following command:

    ```bash
    helm template test-chart
    ```

    The output of the command `helm template test-chart` would be somethings like this:

    ```yaml
    ---
    # Source: test-chart/templates/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    ```

- Another important object that we can use is the Release object.

  The release object contains information about the release, such as the release name, namespace, and the release timestamp.

  The code will look something like this:

  ```yaml
  {{.Release.Name}}
  {{.Release.Namespace}}
  {{.Release.IsUpgrade}}
  {{.Release.IsInstall}}
  {{.Release.Revision}}
  {{.Release.Service}}
  {{.Release.Chart}}
  {{.Release.Config}}
  {{.Release.Manifest}}
  ```

  <details>
  <summary>deployment.yaml</summary><br>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
    {{.Release.Name}}
    {{.Release.IsUpgrade}}
    {{.Release.IsInstall}}
    {{.Release.Service}}
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

- Now, run the following command:

  ```bash
  helm template test-chart
  ```

  The output of the command `helm template test-chart` would be somethings like this:

    ```yaml
    ---
    selector:
    app.kubernetes.io/name: test-chart
    app.kubernetes.io/instance: release-name
    ---
    # Source: test-chart/templates/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    metadata:
    name: release-name-test-chart
    labels:
        helm.sh/chart: test-chart-0.1.0
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
    ...
    ```

- We also have another important one `Template`, which contains information about the template itself.

  The code will look something like this:

  ```yaml
  {{.Template.Name}}
  {{.Template.BasePath}}
  {{.Template.Path}}
  ```

  <details>
  <summary>deployment.yaml</summary><br>

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
        {{.Values.my.custom.data }}
        {{.Chart.Version}}
        {{.Chart.Name}}
        {{.Chart.AppVersion}}
        {{.Chart.Annotations}}
        {{.Release.Name}}
        {{.Release.IsUpgrade}}
        {{.Release.IsInstall}}
        {{.Release.Service}}
        {{.Template.Name}}
        {{.Template.BasePath}}
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

    </details>

    Now, run the following command:

    ```bash
    helm template test-chart
    ```

    The output of the command `helm template test-chart` would be somethings like this:

    ```yaml
      selector:
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
    ---
    # Source: test-chart/templates/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    test-chart/templates/deployment.yaml
    test-chart/templates
    metadata:
    name: release-name-test-chart
    labels:
        helm.sh/chart: test-chart-0.1.0
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
        app.kubernetes.io/version: "1.16.0"
        app.kubernetes.io/managed-by: Helm
    ...
    ```

## Pipe `|`

- If you go under the metadata section under labels we will see pipe `|` symbol. Not, only that the pipe allows us to chain  multiple expressions, commands or  function calls.

  The output of function on the left side of the pipe will be passed as an input to the function on the right side of the pipe.

  ```yaml
  {{ .Values.my.custom.data | default "testdefault" | upper | quote }}
  ```

  <details>
  <summary>deployment.yaml</summary><br>
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
    {{.Release.Name}}
    {{.Release.IsUpgrade}}
    {{.Release.IsInstall}}
    {{.Release.Service}}
    {{.Template.Name}}
    {{.Template.BasePath}}
    {{ .Values.my.custom.data | default "testdefault" | upper | quote }}
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

    </details>

    Now, run the following command:

    ```bash
    $ helm template test-chart
    ```

    Once done, the output of the command `helm template test-chart` would be somethings like this:

    ```yaml
        app.kubernetes.io/name: test-chart
    app.kubernetes.io/instance: release-name
    ---
    # Source: test-chart/templates/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    test-chart/templates/deployment.yaml
    test-chart/templates
    "TEST"
    metadata:
    name: release-name-test-chart
    labels:
        helm.sh/chart: test-chart-0.1.0
        app.kubernetes.io/name: test-chart
    ...
    ```

## Function

- Helm gives us various template functions that can be used to transform data within our template. We have already seen some of these functions like `default` , `upper`, `quote`, `nindent` etc...
- Functions are used to manipulate the data within the template. We can pass data/arguments to the function, and based on that it will render the yaml files.
- Similarly, with `toYaml` function when we get the data from values files or chart files, wtc.. we get it as objects. This 2 YAML will convert the current object into YAML, so that it can be pushed into the output as YAML.

## Conditional Logic in Helm

- So, to understand this let's try it out with an example:
- Add the following to your `values.yam` file:

  ```yaml
  my:
    custom:
      data: "test"
      flag: true
  ```
  
- Once added, let's make changes to `deployment.yaml` file now:

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
    {{.Release.Name}}
    {{.Release.IsUpgrade}}
    {{.Release.IsInstall}}
    {{.Release.Service}}
    {{.Template.Name}}
    {{.Template.BasePath}}
    {{ .Values.my.custom.data | default "testdefault" | upper | quote }}
    
  {{- if .Values.my.custom.flag }}
  {{ "Output of if condition" | nindent 2 }}
  {{- end }}
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
  
- Once done, let's see how the changes comes up.

  ![](./imgs/demo16.gif)

  Here's the output:
  
  ```yaml
  ---
  # Source: test-chart/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  automountServiceAccountToken: true
  ---
  # Source: test-chart/templates/service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
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
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
  ---
  # Source: test-chart/templates/deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    test-chart/templates/deployment.yaml
    test-chart/templates
    "TEST"
  
    Output of if condition
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
    selector:
      matchLabels:
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
    template:
      metadata:
        labels:
          helm.sh/chart: test-chart-0.1.0
          app.kubernetes.io/name: test-chart
          app.kubernetes.io/instance: release-name
          app.kubernetes.io/version: "1.16.0"
          app.kubernetes.io/managed-by: Helm
      spec:
        serviceAccountName: release-name-test-chart
        containers:
          - name: test-chart
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
  ---
  # Source: test-chart/templates/tests/test-connection.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: "release-name-test-chart-test-connection"
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
    annotations:
      "helm.sh/hook": test
  spec:
    containers:
      - name: wget
        image: busybox
        command: ['wget']
        args: ['release-name-test-chart:80']
    restartPolicy: Never
  ```
  
- If you set the same flag to false, this line won't show up in the output, as now the condition is false.
- We can also have a `else` condition in the template, which will be executed if the `if` condition is false.

  ```yaml
  {{- if .Values.my.custom.flag }}
  {{ "Output of if condition" | nindent 2 }}
  {{- else }}
  {{ "Output of else condition" | nindent 2 }}
  {{- end }}
  ```
  
- So, now when you will re-execute it, with the `flag` value being false, the output of the else condition will show up.

## Using `with`

- `with` is used to change the context of the template. It allows us to change the context of the template to a different object.
- This is useful when we want to access the properties of an object without having to use the `.` (dot) notation every time.
- Let's try to understand this with an example. Let's start by making changes in `values.yaml`

  ```yaml
  # Update our custom data in values.yaml
  my:
    custom:
      data: "test"
      flag: true
      region:
        - US
        - Pacific
        - EMEA
        - China
  ```

- Now, let's make changes to `deployment.yaml` file now:


  ```yaml
  apiVersion: apps/v1
  kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
    {{.Release.Name}}
    {{.Release.IsUpgrade}}
    {{.Release.IsInstall}}
    {{.Release.Service}}
    {{.Template.Name}}
    {{.Template.BasePath}}
    {{ .Values.my.custom.data | default "testdefault" | upper | quote }}

    {{- if .Values.my.custom.flag }}
    {{ "Output of if condition" | nindent 2 }}
    {{- end }}

  metadata:
    name: {{ include "test-chart.fullname" . }}
    region:
            {{- with .Values.my.custom.region }}
                    # If variable over here is empty it won't be part of the output
                    {{- toYaml . | nindent 4 }}
                    {{ end }}
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

- Let's try to understand what it means actually:

  ```yaml
  region:
    {{- with .Values.my.custom.region }}
    # If variable over here is empty it won't be part of the output
    {{- toYaml . | nindent 4 }}
    {{ end }}
  ```
  
- So, `with` is not same as `if`. `with` only works when the variable has some value. We can than use the `toYaml` function that will take the data from `values.yaml` as an object. So the `.` here, doesn't point to the root anymore, the scope will be the current element.
- If, you want to use `.Values` and naviagate within the `if` block, you can use a special symbol called `$`.

> [!IMPORTANT]
> In `with`, `$.` again points to the root. And, the `.` only points to the current element.

- Now, once we have made the changes use the command `helm template test-chart`, and view the output:

  ```yaml
  ---
  # Source: test-chart/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  automountServiceAccountToken: true
  ---
  # Source: test-chart/templates/service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
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
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
  ---
  # Source: test-chart/templates/deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    test-chart/templates/deployment.yaml
    test-chart/templates
    "TEST"
  
    Output of if condition
  
  metadata:
    name: release-name-test-chart
    region:
        # If variable over here is empty it won't be part of the output
      - US
      - Pacific
      - EMEA
      - China
  
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  spec:
    replicas: 1
    selector:
      matchLabels:
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
    template:
      metadata:
        labels:
          helm.sh/chart: test-chart-0.1.0
          app.kubernetes.io/name: test-chart
          app.kubernetes.io/instance: release-name
          app.kubernetes.io/version: "1.16.0"
          app.kubernetes.io/managed-by: Helm
      spec:
        serviceAccountName: release-name-test-chart
        containers:
          - name: test-chart
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
  ---
  # Source: test-chart/templates/tests/test-connection.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: "release-name-test-chart-test-connection"
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
    annotations:
      "helm.sh/hook": test
  spec:
    containers:
      - name: wget
        image: busybox
        command: ['wget']
        args: ['release-name-test-chart:80']
    restartPolicy: Never
  ```
  
- As, you can see herein the output, now we see the regions that we mentioned in the `values.yaml` file. But, if we were to leave the values empty, it won't render them.

- We can also add an else condition here, similar to if-else, in the following manner:

  ```yaml
  region:
      {{- with .Values.my.custom.region }}
      # If variable over here is empty it won't be part of the output
      {{- toYaml . | nindent 4 }}
      {{- else }}
      {{ "- Pacific"}}
      {{ end }}
  ```

## Defining Variables

- Defining a variable in helm is pretty easy, let's look at an example on how to define a variable.

  ```yaml
  {{ $myFLAG := "value" }}  
  ```
  
- Once, we assign value to a variable, it becomes a variable of that type.

>[!NOTE]
> If you try to assign different values to this variable at later point in the helm chart template, those will just be ignored. They will not be considered and, only the initial value will be used.

- Let's test this out with an example. For this we will be only making changes to  `deployment.yaml` file only.


  `deployment.yaml`
  
  
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
    {{.Release.Name}}
    {{.Release.IsUpgrade}}
    {{.Release.IsInstall}}
    {{.Release.Service}}
    {{.Template.Name}}
    {{.Template.BasePath}}
    {{ .Values.my.custom.data | default "testdefault" | upper | quote }}

    {{ $myFLAG := .Values.my.custom.flag }}

    {{- if $myFLAG }}
    {{ "Output of if condition" | nindent 2 }}
    {{- end }}

  metadata:
    name: {{ include "test-chart.fullname" . }}
    region:
            {{- with .Values.my.custom.region }}
                    # If variable over here is empty it won't be part of the output
                    {{- toYaml . | nindent 4 }}
                    {{- else }}
                    {{ "- Pacific"}}
                    {{ end }}
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

- Here, you can clearly see that, how we have utilised variables.

  ```yaml
  {{ $myFLAG := .Values.my.custom.flag }}
  
      {{- if $myFLAG }}
      {{ "Output of if condition" | nindent 2 }}
      {{- end }}
  ```

- If you run the command `helm template test-chart`, you will see the following output:

  ```yaml
      ~/G/helm-tutorial/e/test-chart  on   main !2  helm template test-chart
  ---
  # Source: test-chart/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  automountServiceAccountToken: true
  ---
  # Source: test-chart/templates/service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
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
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
  ---
  # Source: test-chart/templates/deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    test-chart/templates/deployment.yaml
    test-chart/templates
    "TEST"
  
  
  
    Output of if condition
  
  metadata:
    name: release-name-test-chart
    region:
        # If variable over here is empty it won't be part of the output
      - US
      - Pacific
      - EMEA
      - China
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  spec:
    replicas: 1
    selector:
      matchLabels:
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
    template:
      metadata:
        labels:
          helm.sh/chart: test-chart-0.1.0
          app.kubernetes.io/name: test-chart
          app.kubernetes.io/instance: release-name
          app.kubernetes.io/version: "1.16.0"
          app.kubernetes.io/managed-by: Helm
      spec:
        serviceAccountName: release-name-test-chart
        containers:
          - name: test-chart
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
  ---
  # Source: test-chart/templates/tests/test-connection.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: "release-name-test-chart-test-connection"
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
    annotations:
      "helm.sh/hook": test
  spec:
    containers:
      - name: wget
        image: busybox
        command: ['wget']
        args: ['release-name-test-chart:80']
    restartPolicy: Never
  ```

## Using Loops

- Let's try to see looping in Helm with an example:

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
    {{.Release.Name}}
    {{.Release.IsUpgrade}}
    {{.Release.IsInstall}}
    {{.Release.Service}}
    {{.Template.Name}}
    {{.Template.BasePath}}
    {{ .Values.my.custom.data | default "testdefault" | upper | quote }}
    
    {{ $myFLAG := .Values.my.custom.flag }}
    
    {{- if $myFLAG }}
    {{ "Output of if condition" | nindent 2 }}
    {{- end }}
  
  metadata:
    name: {{ include "test-chart.fullname" . }}
    region:
        {{- with .Values.my.custom.region }}
        # If variable over here is empty it won't be part of the output
        {{- toYaml . | nindent 4 }}
        {{- else }}
        {{ "- Pacific"}}
        {{ end }}
    deployRegionPossible:
      {{- range .Values.my.custom.region }}
      - {{.}}
      {{- end}}
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
  
- As, here we can clearly see that we have used the `range` keyword to loop through the variables.

  ```yaml
  deployRegionPossible:
    {{- range .Values.my.custom.region }}
    - {{.}}
    {{- end}}
  ```
  
- When you run the `helm template test-chart`, you get the following output:

  ```yaml
     ~/G/helm-tutorial/e/test-chart  on   main !2  helm template test-chart
  ---
  # Source: test-chart/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  automountServiceAccountToken: true
  ---
  # Source: test-chart/templates/service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
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
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
  ---
  # Source: test-chart/templates/deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    test-chart/templates/deployment.yaml
    test-chart/templates
    "TEST"
  
  
  
    Output of if condition
  
  metadata:
    name: release-name-test-chart
    region:
        # If variable over here is empty it won't be part of the output
      - US
      - Pacific
      - EMEA
      - China
    deployRegionPossible:
      - US
      - Pacific
      - EMEA
      - China
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  spec:
    replicas: 1
    selector:
      matchLabels:
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
    template:
      metadata:
        labels:
          helm.sh/chart: test-chart-0.1.0
          app.kubernetes.io/name: test-chart
          app.kubernetes.io/instance: release-name
          app.kubernetes.io/version: "1.16.0"
          app.kubernetes.io/managed-by: Helm
      spec:
        serviceAccountName: release-name-test-chart
        containers:
          - name: test-chart
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
  ---
  # Source: test-chart/templates/tests/test-connection.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: "release-name-test-chart-test-connection"
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
    annotations:
      "helm.sh/hook": test
  spec:
    containers:
      - name: wget
        image: busybox
        command: ['wget']
        args: ['release-name-test-chart:80']
    restartPolicy: Never
  ```
  
## Loop Dictionary

- `range` can also be used to look through a dictionary. Let's try to understand through an example:

  `Code changes`
  
  ```yaml
  image:
        {{- range $key,$value := .Values.image }}
        - {{$key}}:{{$value | quote}}
        {{- end }}
  ```

  `deeployment.yaml`
  
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
    {{.Release.Name}}
    {{.Release.IsUpgrade}}
    {{.Release.IsInstall}}
    {{.Release.Service}}
    {{.Template.Name}}
    {{.Template.BasePath}}
    {{ .Values.my.custom.data | default "testdefault" | upper | quote }}
    
    {{ $myFLAG := .Values.my.custom.flag }}
    
    {{- if $myFLAG }}
    {{ "Output of if condition" | nindent 2 }}
    {{- end }}
  
  metadata:
    name: {{ include "test-chart.fullname" . }}
    region:
        {{- with .Values.my.custom.region }}
        # If variable over here is empty it won't be part of the output
        {{- toYaml . | nindent 4 }}
        {{- else }}
        {{ "- Pacific"}}
        {{ end }}
    deployRegionPossible:
      {{- range .Values.my.custom.region }}
      - {{.}}
      {{- end}}
    image:
      {{- range $key,$value := .Values.image }}
      - {{$key}}:{{$value | quote}}
      {{- end }}
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
  
- Now, run the command `helm template test-chart`, and you will see the following output:

  ```yaml
  ---
  # Source: test-chart/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  automountServiceAccountToken: true
  ---
  # Source: test-chart/templates/service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
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
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
  ---
  # Source: test-chart/templates/deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    test-chart/templates/deployment.yaml
    test-chart/templates
    "TEST"
  
  
  
    Output of if condition
  
  metadata:
    name: release-name-test-chart
    region:
        # If variable over here is empty it won't be part of the output
      - US
      - Pacific
      - EMEA
      - China
    deployRegionPossible:
      - US
      - Pacific
      - EMEA
      - China
    image:
      - pullPolicy:"IfNotPresent"
      - repository:"nginx"
      - tag:""
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  spec:
    replicas: 1
    selector:
      matchLabels:
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
    template:
      metadata:
        labels:
          helm.sh/chart: test-chart-0.1.0
          app.kubernetes.io/name: test-chart
          app.kubernetes.io/instance: release-name
          app.kubernetes.io/version: "1.16.0"
          app.kubernetes.io/managed-by: Helm
      spec:
        serviceAccountName: release-name-test-chart
        containers:
          - name: test-chart
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
  ---
  # Source: test-chart/templates/tests/test-connection.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: "release-name-test-chart-test-connection"
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
    annotations:
      "helm.sh/hook": test
  spec:
    containers:
      - name: wget
        image: busybox
        command: ['wget']
        args: ['release-name-test-chart:80']
    restartPolicy: Never
  ```
  
>[!NOTE]
> We can `dry-run` the template command using `--dry-run` attribute along side `helm template` command.

>[!TIP]
> If you want to get the manifest that was sent to Kubernetes use the command `helm get manifest <chart-name>`.

  `manifest` for `test`

  ```yaml
  helm get manifest test
  ---
  # Source: test-chart/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: test-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: test
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  automountServiceAccountToken: true
  ---
  # Source: test-chart/templates/service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: test-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
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
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: test
  ---
  # Source: test-chart/templates/deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: test-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: test
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  spec:
    replicas: 1
    selector:
      matchLabels:
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: test
    template:
      metadata:
        labels:
          helm.sh/chart: test-chart-0.1.0
          app.kubernetes.io/name: test-chart
          app.kubernetes.io/instance: test
          app.kubernetes.io/version: "1.16.0"
          app.kubernetes.io/managed-by: Helm
      spec:
        serviceAccountName: test-test-chart
        containers:
          - name: test-chart
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
  ```

## Revisiting `_helpers.tpl` file

- `.tpl` stands for template because this file holds several templates that can used across other template files. 
- The very first that is defined over here is the `test-chart.name`.
- Every template starts with a comment.
- A template is more or less like a function in programming language.
- The first one begins with the `define` keyword within the action block followed by the name of the template.

  In our case if we see, it's:

  ```yaml
  {{- define "test-chart.name" -}}
  ```
  
- `test-chart` in our case is a uniqyue namespace. Followed by the actual name of the template. We can look at this as a function name in programming language.

- The question, that you might think of is, why do we need a namespace, cause for the charts we have been creating or application charts to deploy our applications to Kubernetes. 

- But, helm also supports library charts. These charts are not specific to any application. We can reuse those charts across the charts in our applications. In that case, we don't want multiple people naming these templates, giving there templates the same name, and than we having problems to use those templates in our application.

- For example, if a guy has template called `loadOnDeployment`. This is a template somebody has created, and they have given it out as a library chart. Now, we start using this template across our template files. And, another library has also used the same template name. And, we are using both the libraries now in our template filesm, we don't want to have issues using the same name, and that's why we have the concept of namespaces. 

>[!TIP]
> Namespaces in helm as for the given context above are like namespaces in `.NET` and packages in Java and modules in python, that uniquely identify a particular function, so that we can reuse the same functuion name across modules, across packages, projects and so on...

- Within the blocks of the templates we define the entire logic. In this case, it is simply defaulting the chart name. The `test-chart.name` will return the chart name to the `.Chart.Name`.

- If, there is no value in the `values.yaml` file for the field `nameOverride`. If we provide it that will be used otherwisse the chart name will be returned and rendered in the template. The comments are not rendered in the template.

>[!IMP]
> To do a comment in Helm use the following pattern:
> `{{/*.....*/}}`

>[!NOTE]
> The Chart Name we discussed just about in the above section is specifically truncated to 63 characters because some kubernetes name fields are limited to this. 

## Creating and Using Custom Template

- In this section, we will learn how to define our own templates, and use them.

- To, do that go to the `_helpers.tpl` file copy any of the existing templates, and paste it on the top as follows: 

  ```tpl
  {{/*
  My Custom Template
  */}}
  {{- define "test-chart.myTemplate" -}}
  
  {{- end }}
  
  {{/*
  Expand the name of the chart.
  */}}
  {{- define "test-chart.name" -}}
  {{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
  {{- end }}
  
  {{/*
  Create a default fully qualified app name.
  We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
  If release name contains chart name it will be used as a full name.
  */}}
  {{- define "test-chart.fullname" -}}
  {{- if .Values.fullnameOverride }}
  {{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
  {{- else }}
  {{- $name := default .Chart.Name .Values.nameOverride }}
  {{- if contains $name .Release.Name }}
  {{- .Release.Name | trunc 63 | trimSuffix "-" }}
  {{- else }}
  {{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
  {{- end }}
  {{- end }}
  {{- end }}
  
  {{/*
  Create chart name and version as used by the chart label.
  */}}
  {{- define "test-chart.chart" -}}
  {{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
  {{- end }}
  
  {{/*
  Common labels
  */}}
  {{- define "test-chart.labels" -}}
  helm.sh/chart: {{ include "test-chart.chart" . }}
  {{ include "test-chart.selectorLabels" . }}
  {{- if .Chart.AppVersion }}
  app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
  {{- end }}
  app.kubernetes.io/managed-by: {{ .Release.Service }}
  {{- end }}
  
  {{/*
  Selector labels
  */}}
  {{- define "test-chart.selectorLabels" -}}
  app.kubernetes.io/name: {{ include "test-chart.name" . }}
  app.kubernetes.io/instance: {{ .Release.Name }}
  {{- end }}
  
  {{/*
  Create the name of the service account to use
  */}}
  {{- define "test-chart.serviceAccountName" -}}
  {{- if .Values.serviceAccount.create }}
  {{- default (include "test-chart.fullname" .) .Values.serviceAccount.name }}
  {{- else }}
  {{- default "default" .Values.serviceAccount.name }}
  {{- end }}
  {{- end }}
  ```

- Edit the `_helpers.tpl` file something similar to this:

  ```tpl
  {{/*
  My Custom Template
  */}}
  {{- define "test-chart.myTemplate" -}}
  {{- .Values.myValue }}
  {{- end }}
  
  {{/*
  Expand the name of the chart.
  */}}
  {{- define "test-chart.name" -}}
  {{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
  {{- end }}
  
  {{/*
  Create a default fully qualified app name.
  We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
  If release name contains chart name it will be used as a full name.
  */}}
  {{- define "test-chart.fullname" -}}
  {{- if .Values.fullnameOverride }}
  {{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
  {{- else }}
  {{- $name := default .Chart.Name .Values.nameOverride }}
  {{- if contains $name .Release.Name }}
  {{- .Release.Name | trunc 63 | trimSuffix "-" }}
  {{- else }}
  {{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
  {{- end }}
  {{- end }}
  {{- end }}
  
  {{/*
  Create chart name and version as used by the chart label.
  */}}
  {{- define "test-chart.chart" -}}
  {{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
  {{- end }}
  
  {{/*
  Common labels
  */}}
  {{- define "test-chart.labels" -}}
  helm.sh/chart: {{ include "test-chart.chart" . }}
  {{ include "test-chart.selectorLabels" . }}
  {{- if .Chart.AppVersion }}
  app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
  {{- end }}
  app.kubernetes.io/managed-by: {{ .Release.Service }}
  {{- end }}
  
  {{/*
  Selector labels
  */}}
  {{- define "test-chart.selectorLabels" -}}
  app.kubernetes.io/name: {{ include "test-chart.name" . }}
  app.kubernetes.io/instance: {{ .Release.Name }}
  {{- end }}
  
  {{/*
  Create the name of the service account to use
  */}}
  {{- define "test-chart.serviceAccountName" -}}
  {{- if .Values.serviceAccount.create }}
  {{- default (include "test-chart.fullname" .) .Values.serviceAccount.name }}
  {{- else }}
  {{- default "default" .Values.serviceAccount.name }}
  {{- end }}
  {{- end }}
  ```
  
- Update the Value in `values.yaml` file:

  ```yaml
  # Default values for test-chart.
  # This is a YAML-formatted file.
  # Declare variables to be passed into your templates.
  
  my:
    custom:
      data: "test"
      flag: true
      region:
        - US
        - Pacific
        - EMEA
        - China
  
  # This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
  replicaCount: 1
  
  # This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/
  image:
    repository: nginx
    # This sets the pull policy for images.
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
    tag: ""
  
  # This is for the secrets for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  imagePullSecrets: []
  # This is to override the chart name.
  nameOverride: ""
  fullnameOverride: ""
  
  # This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/
  serviceAccount:
    # Specifies whether a service account should be created
    create: true
    # Automatically mount a ServiceAccount's API credentials?
    automount: true
    # Annotations to add to the service account
    annotations: {}
    # The name of the service account to use.
    # If not set and create is true, a name is generated using the fullname template
    name: ""
  
  # This is for setting Kubernetes Annotations to a Pod.
  # For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
  podAnnotations: {}
  # This is for setting Kubernetes Labels to a Pod.
  # For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
  podLabels: {}
  
  podSecurityContext: {}
    # fsGroup: 2000
  
  securityContext: {}
    # capabilities:
    #   drop:
    #   - ALL
    # readOnlyRootFilesystem: true
    # runAsNonRoot: true
    # runAsUser: 1000
  
  # This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/
  service:
    # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
    type: ClusterIP
    # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports
    port: 80
  
  # This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/
  ingress:
    enabled: false
    className: ""
    annotations: {}
      # kubernetes.io/ingress.class: nginx
      # kubernetes.io/tls-acme: "true"
    hosts:
      - host: chart-example.local
        paths:
          - path: /
            pathType: ImplementationSpecific
    tls: []
    #  - secretName: chart-example-tls
    #    hosts:
    #      - chart-example.local
  
  resources: {}
    # We usually recommend not to specify default resources and to leave this as a conscious
    # choice for the user. This also increases chances charts run on environments with little
    # resources, such as Minikube. If you do want to specify resources, uncomment the following
    # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
    # limits:
    #   cpu: 100m
    #   memory: 128Mi
    # requests:
    #   cpu: 100m
    #   memory: 128Mi
  
  # This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
  livenessProbe:
    httpGet:
      path: /
      port: http
  readinessProbe:
    httpGet:
      path: /
      port: http
  
  # This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/
  autoscaling:
    enabled: false
    minReplicas: 1
    maxReplicas: 100
    targetCPUUtilizationPercentage: 80
    # targetMemoryUtilizationPercentage: 80
  
  # Additional volumes on the output Deployment definition.
  volumes: []
  # - name: foo
  #   secret:
  #     secretName: mysecret
  #     optional: false
  
  # Additional volumeMounts on the output Deployment definition.
  volumeMounts: []
  # - name: foo
  #   mountPath: "/etc/foo"
  #   readOnly: true
  
  nodeSelector: {}
  
  tolerations: []
  
  affinity: {}
  
  
  myValue: test
  ```
  
- Utilise the templates value in `deployment.yaml`.

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
    {{.Values.my.custom.data }}
    {{.Chart.Version}}
    {{.Chart.Name}}
    {{.Chart.AppVersion}}
    {{.Chart.Annotations}}
    {{.Release.Name}}
    {{.Release.IsUpgrade}}
    {{.Release.IsInstall}}
    {{.Release.Service}}
    {{.Template.Name}}
    {{.Template.BasePath}}
    {{ .Values.my.custom.data | default "testdefault" | upper | quote }}
    
    {{ $myFLAG := .Values.my.custom.flag }}
    
    {{- if $myFLAG }}
    {{ "Output of if condition" | nindent 2 }}
    {{- end }}
  
  metadata:
    name: {{ include "test-chart.fullname" . }}
    customTemplateValue: {{template "test-chart.myTemplate" . }}
    region:
        {{- with .Values.my.custom.region }}
        # If variable over here is empty it won't be part of the output
        {{- toYaml . | nindent 4 }}
        {{- else }}
        {{ "- Pacific"}}
        {{ end }}
    deployRegionPossible:
      {{- range .Values.my.custom.region }}
      - {{.}}
      {{- end}}
    image:
      {{- range $key,$value := .Values.image }}
      - {{$key}}:{{$value | quote}}
      {{- end }}
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
  
- Run the command `helm template test-chart`. You will see the following output:

  ```bash
     ~/G/helm-tutorial/e/test-chart  on   main !3  helm template test-chart
  ---
  # Source: test-chart/templates/serviceaccount.yaml
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  automountServiceAccountToken: true
  ---
  # Source: test-chart/templates/service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: release-name-test-chart
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
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
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
  ---
  # Source: test-chart/templates/deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
    test
    0.1.0
    test-chart
    1.16.0
    map[]
    release-name
    false
    true
    Helm
    test-chart/templates/deployment.yaml
    test-chart/templates
    "TEST"
  
  
  
    Output of if condition
  
  metadata:
    name: release-name-test-chart
    customTemplateValue: test
    region:
        # If variable over here is empty it won't be part of the output
      - US
      - Pacific
      - EMEA
      - China
    deployRegionPossible:
      - US
      - Pacific
      - EMEA
      - China
    image:
      - pullPolicy:"IfNotPresent"
      - repository:"nginx"
      - tag:""
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
  spec:
    replicas: 1
    selector:
      matchLabels:
        app.kubernetes.io/name: test-chart
        app.kubernetes.io/instance: release-name
    template:
      metadata:
        labels:
          helm.sh/chart: test-chart-0.1.0
          app.kubernetes.io/name: test-chart
          app.kubernetes.io/instance: release-name
          app.kubernetes.io/version: "1.16.0"
          app.kubernetes.io/managed-by: Helm
      spec:
        serviceAccountName: release-name-test-chart
        containers:
          - name: test-chart
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
  ---
  # Source: test-chart/templates/tests/test-connection.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: "release-name-test-chart-test-connection"
    labels:
      helm.sh/chart: test-chart-0.1.0
      app.kubernetes.io/name: test-chart
      app.kubernetes.io/instance: release-name
      app.kubernetes.io/version: "1.16.0"
      app.kubernetes.io/managed-by: Helm
    annotations:
      "helm.sh/hook": test
  spec:
    containers:
      - name: wget
        image: busybox
        command: ['wget']
        args: ['release-name-test-chart:80']
    restartPolicy: Never
  ```
  
- Look for the deployment section, and specifically look for variable `customTemplateValue`.