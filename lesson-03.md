# Helm Charts

![](./imgs/maxresdefault.jpg)

## Creating Helm Charts

> [!IMPORTANT]
>
> - In order to create a Helm Chart use the command:
>
>    ```bash
>    helm create <chart-name>
>    ```
>
>    <details>
>    <summary>Click to see the example</summary>
>
>    ```bash
>    $ helm create test-chart
>    Creating test-chart
>    ```
>
>    </details>

- If you `cd` into the `test-chart` directory, you will see the following structure:

    ```bash
    $ tree test-chart
    test-chart
    ├── Chart.yaml
    ├── charts
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── hpa.yaml
    │   ├── ingress.yaml
    │   ├── service.yaml
    │   ├── serviceaccount.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml

    4 directories, 10 files
    ```

- The `chart.yaml` file is where the metadata of the chart is stored.
- The `charts` folder is where the dependencies of the chart are stored, and is initially empty.
- The templates folder is where we have all the templates for the Kubernetes resources that will be created.
- `values.yaml` is where the default values for the chart are stored.

- So, let's start by installing the chart we just created:

-   ```bash
    $ helm install test test-chart 

    NAME: test
    LAST DEPLOYED: Sun Feb 16 22:15:28 2025
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

- So, now the helm deployment is successfull, by default it's an nginx deployment. Once, done you need to fetch the `POD_NAME` and `CONTAINER_PORT` and then you can access the application using the `port-forward` command.

  ![](./imgs/Screenshot%202025-02-16%20at%2010.16.24 PM.png)

## `chart.yaml`

- The `chart.yaml` file is where the metadata of the chart is stored. It contains the following fields:

    <details>
    <summary>Click to see "charts.yaml" </summary>

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
    ```

    </details>

- There are 3 mandatory elements within this chart.yaml:
  - `apiVersion`: The version of the Helm chart API.
  - `name`: The name of the chart.
  - `version`: The version of the chart.

- The rest of the fields are optional, but it is recommended to fill them out. 

- The `apiVersion` determines the rest of the document, i.e. the structure that needs to be followed by the rest of the document.

  The elements that can be used, the mandatory elements and the optional elements are determined by the `apiVersion` we use here.

  - The Current version of the Helm chart API is `v2`.

- The next is the `name` of the chart. The description can be any textul information about the chart or what it does.

- The `type` of the chart can be either `application` or `library`.

  - The `application` type is a collection of templates that can be packaged into versioned archives to be deployed.

  - The `library` type provides useful utilities or functions for the chart developer.

  - They're included as a dependency of application charts to inject those utilities and functions into the rendering pipeline. Library charts do not define any templates and therefore cannot be deployed.

- The `version` is the version of the chart. This version number should be incremented each time you make changes to the chart and its templates, including the app version.

- The `appVersion` is the version number of the application being deployed. This version number should be incremented each time you make changes to the application. Versions are not expected to follow Semantic Versioning. They should reflect the version the application is using.

Other than this there are some other additional fields that can be added to the `chart.yaml` file:

- `icon`: The URL to an SVG or PNG image to be used as an icon for this chart.

  ```yaml
  icon: https://example.com/icon.png
  ```

- `keywords`: A list of keywords about the chart.

  ```yaml
  keywords:
    - nginx
    - web
    - server
  ```