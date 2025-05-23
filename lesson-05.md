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

