# Helm Advanced

## Helm release workflow

- Now, we will walk through the behind the scenes of Helm when we execute `helm install` command.

- This is the same process, whether you are pulling a helm chart from a remote repository or using your own chart that you have on your machine, which we will be learning in sections later on. The process is the same.

- When we execute `helm install` command, Helm does the following:

  - **Load the Chart**

    - **Local Chart**: If the chart is stored locally, Helm loads it directly from your machine.
    - **Remote Chart**: If the chart is hosted in a remote repository, Helm pulls the chart from the repository before loading it.
    - **Dependencies**: Helm resolves and loads any dependencies specified in the `Chart.yaml` file.

  - **Parse Values**
    - Helm parses the default `values.yaml` file and any other values files included in the chart.
    - These files contain the default values used to render the templates in the `templates` directory.
    - If the user provides custom values using the `--set` or `--values` flags, Helm overrides the default values with the custom values.

  - **Render Templates**
    - Helm uses the provided values (custom or default) to render the Go templates in the `templates` directory.
    - This step generates YAML files that define the Kubernetes resources to be created.

  - **Parse YAML into Kubernetes Objects**
    - The generated YAML files are parsed into Kubernetes objects (e.g., Deployments, Services, ConfigMaps).
    - Helm does not simply hand over the raw YAML to Kubernetes; it first ensures the YAML is structured correctly.

  - **Validate Kubernetes Objects**
    - Helm validates the Kubernetes objects against the Kubernetes schema.
    - If the objects are valid, Helm proceeds to generate the final YAML manifest.
    - If the objects are invalid, Helm returns an error to the user and stops the process.
  
  - **Send YAML to Kubernetes API Server**
    - Helm sends the final YAML manifest to the Kubernetes API server.
    - The API server processes the YAML and begins creating the specified resources on the cluster.
  
  - **Command Success**
    - Helm considers the `helm install` command successful as soon as the YAML is handed over to the Kubernetes API server.
    - **Note**: Helm does not wait for the resources to be fully created on the cluster. It simply ensures the YAML is accepted by the API server.

  - **Store Release Metadata**
    - Helm creates a release object to track the installation.
    - The release metadata (e.g., release name, version, values) is stored in Kubernetes Secrets or ConfigMaps for future reference.

  - **User Verification**
    - The user can verify the installation using the `helm status` command.
    - This command provides details about the release, including its status and the resources created.

- Summary
  - The `helm install` process involves loading the chart, parsing values, rendering templates, validating Kubernetes objects, and handing over the final YAML to the Kubernetes API server. Helm considers the command successful once the YAML is accepted by the API server, without waiting for resource creation. Release metadata is stored for future reference, and users can verify the installation using `helm status`.

<br/>

  ```mermaid
  flowchart TD
    A[helm install] --> B{Is chart local or remote?}
    B -->|Local| C[Load chart from local machine]
    B -->|Remote| D[Pull chart from repository]
    C --> E[Load chart dependencies]
    D --> E
    E --> F[Parse default values.yaml and other values files]
    F --> G{Does user provide custom values?}
    G -->|Yes| H[User provides values via --set or --values]
    G -->|No| I[Use default values from chart]
    H --> J[Helm overrides default values with custom values]
    I --> J
    J --> K[Render templates using provided values]
    K --> L[Generate YAML files from templates]
    L --> M[Parse YAML files into Kubernetes Objects]
    M --> N[Validate Kubernetes Objects against Kubernetes Schema]
    N -->|Valid| O[Generate final YAML manifest]
    N -->|Invalid| P[Helm returns error to user]
    O --> Q[Send YAML manifest to Kubernetes API Server]
    Q --> R[Kubernetes API Server processes YAML]
    R --> S[Helm considers command successful]
    S --> T[Helm creates and stores release metadata]
    T --> U[User can verify installation using helm status]
    U --> V[End]

  ```