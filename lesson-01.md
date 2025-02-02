# Helm Kubernetes Package Manager

A Tutorial on Helm by @amandewatnitrr.

![](./imgs/ff1xjsnvdlnfinirlwb-0p42tlo.png)

### Why we need Helm ??

- Let's say we are wokring on a microservice based application, that needs to be deployed on a Kubernetes cluster. The application might use a database as well, we need to come up with all the YAML files required to deploy the application on the cluster. This includes the deployment, service, ingress, configmap, secret, etc. files.

- The Problem comes here is we need to hardcode a lot of values in the YAML files, like the image name, tag, environment variables, etc. As all these files are static.

  This is not a good practice as we need to change these values for different environments like dev, staging, production, etc.

- If you look at the yaml configurations in most of the elements are same, and cannot recieve the parameters dynamically.

- When all the resources are created, when we do a kube installation, that means our application is live on the cluster are all the resources are live on the cluster.

  But, it is hard to maintain consistency if a developer or a DevOps Engineer directly goes and tweaks the resources on the Kubernetes cluster instead of updating these files, checking them into Github or pushing them to Github and then updating the Kubernetes cluster.

- If we use commands like kubectl directly on the cluster, they always have consistency issues. That is something helm will address.

- Revision History, as our applications are installed and upgraded, kubernetes doesnot maintain any version history for us. Also, if we make changes to even one of the Kubernetes Components, the whole redeployment happens again, and let's say we want to rollback to a previous version of it, it is hard to track back to unless a proper backup is maintained for this.

  For this purpose, Helm also maintains a revision history for us.

## What is Helm ??

![](./imgs/670d1ae23c7f883635b3ba90_mogenius_hero_helm-for-kubernetes.jpg)

- Helm works with Charts. Charts are like packages in Helm. They are a collection of all the resources required to run an application on a Kubernetes cluster.

- These charts will have all the template files and the configuration required to create kubernetes resources.

- We can pull the chart using a single command to pull the chart you want. Let's say to install an Apache onto k8s cluster, we use the following command:

  ```bash
  helm install apache bitnami/apache --namespace=dev
  ```
  
  where that chart lives, it pulls the chart, takes all the templates and the values in their chart, and it will create the resource file to kubernetes, which will eventually create the kubernetes resources under the namespace you provide, all with a single command.

  - Simplifying deployments: Helm charts can simplify the deployment process and make it easier to manage applications.
  - Reducing complexity: Helm can help reduce the complexity of creating different environments for development, testing, and production.
  - Improving productivity: Helm can help developers be more productive by allowing them to focus on writing code instead of deployment.
  - Streamlining CI/CD pipelines: Helm can help streamline CI/CD pipelines.
  - Automating application management: Helm can help automate the deployment and management of applications, which can improve system reliability and stability.
  - Centralizing application sharing: Helm repositories can help developers share their applications with users and help users find the applications they need.
  - Other advantages of Helm include: Maintaining a database of all release versions, Customizing application configurations during deployment, Creating custom charts, Accessing many preconfigured packages, and Organizing application components into charts.

- The Charts that we talked about are stored in Chart repositoies. Using the helm commands, we can pull and install the charts from these repositories.

## Working with Helm Chart Repositories

- To add a chart repository, we use the following command:

  ```bash
  helm repo add bitnami https://charts.bitnami.com/bitnami
  ```

  This command will add the bitnami repository to the helm.

- To list all the repositories added to the helm, we use the following command:

  ```bash
  helm repo list
  # helm repo ls
  ```

- To install a chart from the repository, we use the following command:

  ```bash
  helm install apache bitnami/apache --namespace=dev
  ```

- `helm repo list` command will list all the repositories added to the helm.
- `helm repo add bitnami https://charts.bitnami.com/bitnami` command will add the bitnami repository to the helm.
- `helm repo remove bitnami` command will remove the bitnami repository from the helm.
- `helm search repo mysql` command will search for the mysql chart in the helm repositories.
- `helm search repo database` command will search for the database chart in the helm repositories.
- `helm search repo database --versions` command will search for the database chart in the helm repositories along with the versions.

![](./imgs/demo.gif)

## The Magic of Helm

- To experience what we can do helm and how it simplifies the deployment process, let's take an example of deploying a MySQL database on a Kubernetes cluster from bitnami repository.

  ![](./imgs/demo1.gif)

- In order to install the MySQL database on the Kubernetes cluster, we use the following command:

  ```bash
  helm install mysql bitnami/mysql
  ```
  
  ![](./imgs/demo2.gif)

- Once the installation is doen and completed, copy the logs from the terminal and paste it on a notepad or a text editor.

  Use the command and instructions as shown in the logs to connect to the MySQL database.

- And, thus you can easily install a MySQL database on a Kubernetes cluster using Helm. If you want to confirm, go to the terminal where you used `minikube ssh` and run the following command:

  ```bash
  docker images
  ```

  You will see the MySQL pod running on the Kubernetes cluster.

>[!NOTE]
>We can not have the same chatname in the same namespace. If we want to install the same chart again, we need to provide a different name to the chart. <br><br>For that you need to create a new namespace using the command <b>`kubectl create namespace new_manespace_name`</b> and then install the chart using the command <b>`helm install mysql1 bitnami/mysql --namespace=dev1`</b>.

## List and Uninstall the Helm Charts

- To list all the charts installed on the Kubernetes cluster, we use the following command:

  ```bash
  helm list
  ```

- If you want to list charts from a specific namespace, you can use the following command:

  ```bash
  helm list --namespace=<namespace>
  ```

  ![](./imgs/demo3.gif)

- To uninstall a chart from the Kubernetes cluster, we use the following command:

  ```bash
  helm uninstall <chart-name>
  ```

  ![](./imgs/demo4.gif)

## Providing Custom Values to the Helm Charts

![](./imgs/demo5.gif)

- So, in the previous example, we installed the MySQL database on the Kubernetes cluster using the bitnami repository. But, what if we want to provide some custom values to the MySQL database like password, you can do it using `--set` flag, as follows:

  ```bash
  helm install mysql bitnami/mysql --set auth.rootPassword = test1234
  ```

  ![](./imgs/demo-set-flag-example.gif)

- We can also use `--values` flag to provide the custom values to the helm charts. For that, we need to create a file with the custom values and provide the file path to the `--values` flag.

  ```bash
  helm install mysql bitnami/mysql --values values.yaml
  ```

  The `values.yaml` file in our case will look like:

  ```yaml
  auth:
    rootPassword: "test1234"
  ```

  Once, you have created the `values.yaml` file, you can use the following command:

  ```bash
  helm install mysql bitnami/mysql --values /path/to/values.yaml
  ```

  ![](./imgs/deom-f-flag-example.gif)

  In our above example gif, we created a file by name `providing-custom-values.yaml` and the content of the file is:

  ```yaml
  replicaCount: 3
  global:
    security:
      allowInsecureImages: true
  service:
    type: NodePort
    nodePort: 30080
  env:
    - name: NGINX_PORT
      value: "8080"
  persistence:
    enabled: true
    storageClass: standard
    size: 10Gi
  ```

  And, this will set the values as given in the `values.yaml` file. And than used the command:

  ```bash
   helm install my-release oci://registry-1.docker.io/bitnamicharts/nginx -f ./experiments/providing-custom-values.yaml
  ```

## Helm Upgrade

> [!NOTE]
> `helm repo update` <br>
> These command will fetch the latest charts by going to the repository.
> ![](./imgs/Screenshot%202025-01-23%20at%202.47.59 AM.png)

> [!NOTE]
> `helm upgrade <chart-name> <chart-repo/chart-name> --values values.yaml` <br>
> This command will upgrade the chart with the new values provided in the `values.yaml` file.

  For example, let's say we want to update the replicaCount to 4 in the `values.yaml` file, make changes to the `values.yaml` file as follows:

  ```yaml
  replicaCount: 4
  global:
    security:
      allowInsecureImages: true
  service:
    type: NodePort
    nodePort: 30080
  env:
    - name: NGINX_PORT
      value: "8080"
  persistence:
    enabled: true
    storageClass: standard
    size: 10Gi
  ```

  Than, use the following command to upgrade the chart:

  ```bash
  helm upgrade my-release oci://registry-1.docker.io/bitnamicharts/nginx --values ./experiments/providing-custom-values.yaml
  ```

  ![](./imgs/demo6.gif)

> [!IMPORTANT]
> Helm is intelligent enough to only push those changes to the kubernetes cluster that are required if there are no updates to the chart version that we are using, it won't send any updates to the kubernetes cluster.
> It will generate the Kubernetes Templates and compare them with the existing templates on the cluster and only push the changes that are required.

## Release Records

- Run the command `helm list` to list all the releases that are installed on the Kubernetes cluster.

  ```bash
  > helm ls

  NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
  my-release      default         2               2025-02-02 04:32:34.639945 +0530 IST    deployed        nginx-18.3.5    1.27.3
  ```

- Now, run the command:

  ```bash
  kubectl get secrets
  ```

  And, you will see some secrets created by the helm, like this:

  ![](./imgs/Screenshot%202025-02-02%20at%205.32.06 PM.png)

  There's one secret for NGINX Installation, and other 2 secrets for same installation, for each version of the installation. So, when we first installed nginx, `sh.helm.v1.my-release.v1` was created, and when we upgraded the nginx, `sh.helm.v1.my-release.v2` was created.

- This secret record has the entire information about he installation, this is how helm maintains the revision history.

- We might discuss about this in depth in the coming lessons.

- Also, once we do `helm uninstall <chart-name>`, it will remove all the resources from the kubernetes cluster.

  Not only installation will be gone, but also the version information will be gone.

  > [!TIP]
  > If you want to retain this information for rollback etc, which we will learn in other section later on, you can use the `--keep-history` flag with the `helm uninstall` command.<br/>
  > In versions before Helm 3, `--keep-history` was the default behavior, but in Helm 3, it is not the default behavior.

## Assignment

- Pull `bitnami/tomcat` chart from the bitnami repository and install it on the Kubernetes cluster.
  
- Create a `values.yaml` file with the following content:

  ```yaml
  tomcatPassword: BtjztgfSBO
  service:
    type: NodePort # It's a LoadBalancer by default
    nodePort: 30007
  ```

- Use the `helm upgrade` command to update the chart with the new values provided in the `values.yaml` file.

- Once, done show the changes by running the `kubectl get services <service-name> -o yaml` command.

- Use `helm delete <chart-name>` to delete the chart from the Kubernetes cluster.