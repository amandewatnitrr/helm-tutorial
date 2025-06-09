# Advanced Charts

## Adding Dependencies

### Understanding Chart Dependencies

- Helm charts can declare dependencies on other charts, enabling you to:

  - Reuse existing Charts
  - Compose complex applications from components
  - Manage external services required by your application

- Dependencies are specified in the `Chart.yaml` file under the dependencies section. Let's extend our example:

  ```yaml
  apiVersion: v2
  name: test-chart
  description: A Helm chart for Kubernetes
  
  # A chart can be either an 'application' or a 'library' chart.
  #
  # Application charts are a collection of templates that can be packaged into versioned archives
  # to be deployed.
  #
  # Library charts provide useful utilities or functions for the chart developer. They're included as
  # a dependency of application charts to inject those utilities and functions into the rendering
  # pipeline. Library charts do not define any templates and therefore cannot be deployed.
  type: application
  
  # This is the chart version. This version number should be incremented each time you make changes
  # to the chart and its templates, including the app version.
  # Versions are expected to follow Semantic Versioning (https://semver.org/)
  version: 0.1.0
  
  # This is the version number of the application being deployed. This version number should be
  # incremented each time you make changes to the application. Versions are not expected to
  # follow Semantic Versioning. They should reflect the version the application is using.
  # It is recommended to use it with quotes.
  appVersion: "1.16.0"
  
  dependencies:
  - name: prometheus-node-exporter
    version: 4.6.0  # PSP-free version
    repository: https://prometheus-community.github.io/helm-charts
    condition: metrics.enabled
  ```
  
- Dependency Fields Explained:

  - `name`: Required. The chart name 
  - `version`: Required. Chart version (SemVer)
  - `repository`: Chart repository URL or alias 
  - `condition`: (Optional) Boolean value path to enable/disable 
  - `tags`: (Optional) Group of tags to enable/disable charts 
  - `alias`: (Optional) Alternative name for dependency

### Dependency Management Workflow

- Add required repositories
- 
  ```bash
  helm repo add bitnami https://charts.bitnami.com/bitnami
  ```
  
- Update dependencies

  ```bash
  helm dependency update test-chart
  ```
  
  And, we see the following output:

  ```bash
  > helm dependency update test-chart
  Hang tight while we grab the latest from your chart repositories...
  ...Successfully got an update from the "kubernetes-dashboard" chart repository
  ...Successfully got an update from the "kong" chart repository
  ...Successfully got an update from the "jetstack" chart repository
  ...Successfully got an update from the "mysql-operator" chart repository
  ...Successfully got an update from the "bitnami" chart repository
  Update Complete. ⎈Happy Helming!⎈
  Saving 2 charts
  Downloading redis from repo https://charts.bitnami.com/bitnami
  Pulled: registry-1.docker.io/bitnamicharts/redis:21.1.4
  Digest: sha256:9e8416184c8b070c39c7d54918b8194dcf3cc90d2becbaafdd352c67f3027a5e
  Downloading cert-manager from repo https://charts.jetstack.io
  Deleting outdated charts
  ```

- Verify the dependencies

  ```bash
  helm dependency list test-chart
  ```
  
  And, we get the following output:

  ```bash
  > helm dependency list test-chart
  NAME                            VERSION REPOSITORY                                              STATUS
  prometheus-node-exporter        4.6.0   https://prometheus-community.github.io/helm-charts      ok
  ```
  
- Use the helm install command, to install the chart:

  ```bash
  > helm install test test-chart
  NAME: test
  LAST DEPLOYED: Fri May 23 08:55:20 2025
  NAMESPACE: default
  STATUS: deployed
  REVISION: 1
  NOTES:
  1. Get the application URL by running these commands:
    export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=test-chart,app.kubernetes.io/instance=test" -o jsonpath="{.items[0].metadata.name}")
    export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
    echo "Visit http://127.0.0.1:8080 to use your application"
    kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
  ```
  
- Use the following command to fetch the POD Name:

  ```bash
  > POD_NAME=$(kubectl get pods -l app.kubernetes.io/name=prometheus-node-exporter -o jsonpath='{.items[0].metadata.name}')
  
  # Forward port
  > kubectl port-forward pod/$POD_NAME 9100:9100
  Forwarding from 127.0.0.1:9100 -> 9100
  Forwarding from [::1]:9100 -> 9100
  Handling connection for 9100
  ```
  
  If, you visit this page or curl it you see the following:

  ```bash
  > curl localhost:9100/metrics
  # HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
  # TYPE go_gc_duration_seconds summary
  go_gc_duration_seconds{quantile="0"} 8.0874e-05
  go_gc_duration_seconds{quantile="0.25"} 9.1458e-05
  go_gc_duration_seconds{quantile="0.5"} 0.000660583
  go_gc_duration_seconds{quantile="0.75"} 0.000747958
  go_gc_duration_seconds{quantile="1"} 0.000747958
  go_gc_duration_seconds_sum 0.001580873
  go_gc_duration_seconds_count 4
  # HELP go_goroutines Number of goroutines that currently exist.
  # TYPE go_goroutines gauge
  go_goroutines 7
  .....................
  ```
  
  Something of this sort should be visible. This is how you can add a dependency within a chart.

- Using this `_helpers.tpl` file where we have specified a specific version for the prometheus chart, we can assign a range of version for the use.

  For example in our scenario we can use:

  ```yaml
  dependencies:
  - name: prometheus-node-exporter
    version: ">=4.2.0" 
    {{/* All versions greater than 4.2.0 */}}
    repository: https://prometheus-community.github.io/helm-charts
    condition: metrics.enabled
  ```
  
- In a similar fashion we can combine these conditions, so we can modify it as:

  ```yaml
  dependencies:
  - name: prometheus-node-exporter
    version: ">=4.2.0 and =<4.6.0" 
    {{/* All versions greater than 4.2.0 */}}
    repository: https://prometheus-community.github.io/helm-charts
    condition: metrics.enabled
  ```

- You might be thinking what is the advantage of using these ranges instead of exact version. If we are sure that our application, the chart we are trying to deploy, which is using this dependency, can work with the latest version of the dependency.

- Let's consider the scenario that postgres is going to release a new version of itself, and we are pretty much sure that it will work just fine with the new upcoming release. So, instead of specifying one specific version we can set a range for this.

### Use Dependencies Conditionally

- Since, we define a chart dependency and, we have the dependent chart under the charts folder, it will be used everytime an installation or upgrade of our chart is done, and that chart will than be installed onto the Kubernetes Cluster.

- But, if we want that to happen on a condition which we can pass from the `values.yaml` file. It's super simple to do so.

- Just go the `values.yaml` chart, and create a variable similar to this:

  ```yaml
  # Metrics exporter (simple addon)
  metrics:
    enabled: true
  ```
  
- Once, done just go to `Chart.yaml`, and put the `conditions` parameter as we have done earlier as well with the associated dependency.

  ```yaml
  apiVersion: v2
  name: test-chart
  description: A Helm chart for Kubernetes
  
  # A chart can be either an 'application' or a 'library' chart.
  #
  # Application charts are a collection of templates that can be packaged into versioned archives
  # to be deployed.
  #
  # Library charts provide useful utilities or functions for the chart developer. They're included as
  # a dependency of application charts to inject those utilities and functions into the rendering
  # pipeline. Library charts do not define any templates and therefore cannot be deployed.
  type: application
  
  # This is the chart version. This version number should be incremented each time you make changes
  # to the chart and its templates, including the app version.
  # Versions are expected to follow Semantic Versioning (https://semver.org/)
  version: 0.1.0
  
  # This is the version number of the application being deployed. This version number should be
  # incremented each time you make changes to the application. Versions are not expected to
  # follow Semantic Versioning. They should reflect the version the application is using.
  # It is recommended to use it with quotes.
  appVersion: "1.16.0"
  
  dependencies:
    - name: prometheus-node-exporter
      version: 4.6.0  # PSP-free version
      repository: https://prometheus-community.github.io/helm-charts
      condition: metrics.enabled
  ```

- Similarly, we can also do the same using tags as well:

  ```yaml
  dependencies:
      - name: prometheus-node-exporter
        version: 4.6.0  # PSP-free version
        repository: https://prometheus-community.github.io/helm-charts
        tags:
          - enabled
  ```
  
### Reading Value from Child Charts

- There are 2 ways to import values from child chart to a parent chart, but we will be using the same element in the dependency.

- For this we will use `import-values` to import the values that are exposed by the child chart in the parent chart. As, we said there are 2 ways to do so:

  - Explicit export and import
    - The Child Chart can explicitly expose out some values so that the parent charts can use them.
    - In this case, the child chart should have a property called `export`. It's a root level element. It can be added anywhere in the `values.yaml` file, which will look something like this:

     ```yaml
     export:
        service:
           portL: 8080
     ```
    
    - Now, any chart using the above assumed child chart as dependency, than we can directly use them under the `improt-values` element. And, we can import it as follows:

     ```yaml
     dependencies:
         - name: prometheus-node-exporter
           version: 4.6.0  # PSP-free version
           repository: https://prometheus-community.github.io/helm-charts
           tags:
             - enabled
           import-values:
             - service
     ```
    
  - Importing from child even when there is no explicit export used
    - So, in this scenario, we have no `export` parameter in the child anymore. But, we still have `import-values` parameter still there, however it's usage here is changes.
    - Now, as we don't have anything to export, the parent chart has to pull these changes using the `import-values` parameter.
    - Let's try to explain using an example.

      ```yaml
      dependencies:
         - name: prometheus-node-exporter
           version: 4.6.0  # PSP-free version
           repository: https://prometheus-community.github.io/helm-charts
           tags:
             - enabled
           import-values:
             - child: xyz_parent_element.param_to_import
               parent: new_name_param 
      ```
      
    - Here, the `new_name_param` will ahve all the values in the `param_to_import` that was imported from the child chart.

## Hooks

- If we want to take some special action during the heelm release process, then we create a hook.

- For example, storing some data in the databse, backing up the database etc.. during the release process.

- A hook is just like any other template file under the Charts `template` folder. Generally named as `hoodpod.yaml`. It looks just like any other template file, and we can create any type of kubernetes resource, within this template file as well. Difference is our hook will be marked with some annotations like this:

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: {{ include "test-chart.fullname" . }}-pre-install-hook
    annotations:
      "helm.sh/hook": pre-install
      "helm.sh/hook-weight": "-1"
  spec:
    containers:
      - name: pre-install-hook
        image: busybox
        imagePullPolicy: IfNotPresent
        command: ["sh", "-c", "echo 'This is a pre-install hook'"]
    restartPolicy: OnFailure
  ```
    
- As, soon as helm sees these hooks in the annotations, it will treat them as hooks and will execute them at the time of release process.

- They will not be used when the installation happens, but you can configure when these hooks should be created. Like, in the scenario here, as you can see, we have pre-install hook, which means this will be executed before the installation of the chart. 

- We can create different types of resources not just pods, we can create jobs, configmaps, secrets etc. as well.

- We can also decide when the hook should be executed, by assigning a proper execution priority to it using `hook-weight` annotation for the purpose.

  - If we have multiple hooks, we can provide a number here.
  - It can be a negative number, zero, or a positive number.
  - The hooks will be ordered in the ascending order of the weight we provide here.

- now, when you run the `helm install` command, you will see this pod, and to see if it executed the command check the logs.

  ```shell
  $ kubectl logs pod/test-test-chart-pre-install-hook
  This is a pre-install hook
  ```
  
## Testing your charts

- For testing your charts go to the `test-chart` folder that we created, than to templates, and there we will find the tests folder.

- So, when we created our chart command helm creates a test yaml file for you named `test-connection.yaml`.

- This is what `test-connection.yaml` looks like:

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: "{{ include "test-chart.fullname" . }}-test-connection"
    labels:
      {{- include "test-chart.labels" . | nindent 4 }}
    annotations:
      "helm.sh/hook": test
  spec:
    containers:
      - name: wget
        image: busybox
        command: ['wget']
        args: ['{{ include "test-chart.fullname" . }}:{{ .Values.service.port }}']
    restartPolicy: Never
  ```

- It is a resource of type Pod, but if you look at the annotations it's a hook as mentioned in the line `"helm.sh/hook": test`. As, we know the helm hooks are executed depending upon the phase we configure.

- So, the hook here will be enabled and used only when we run the helm test command. If the return value of the command and the Pod is 0. it means the execution is successfull.

  If a non-zero value is returned it means that the test has failed.

- Here's a example below on how to execute this:

  ```bash
  $ helm test test
  NAME: test
  LAST DEPLOYED: Sat Jun  7 11:34:59 2025                                                            NAMESPACE: default
  STATUS: deployed
  REVISION: 1
  TEST SUITE:     test-test-chart-test-connection
  Last Started:   Sat Jun  7 11:47:11 2025
  Last Completed: Sat Jun  7 11:47:16 2025
  Phase:          Succeeded
  NOTES:
  1. Get the application URL by running these commands:
    export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=test-chart,app.kubernetes.io/instance=test" -o jsonpath="{.items[0].metadata.name}")
    export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
    echo "Visit http://127.0.0.1:8080 to use your application"
    kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
  ```