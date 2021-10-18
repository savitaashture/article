# Getting started with Tekton
Tekton is a Kubernetes-native open source framework for creating continuous integration and delivery (CI/CD) systems. It also helps to do end-to-end(build, test, deploy) application development across multiple cloud providers or on-premises systems by abstracting away the underlying implementation details.
# Introduction to Tekton
[Tekton](https://github.com/tektoncd/pipeline), originally known as [Knative Build](https://github.com/knative/build) but spun-off as its own open source project with its own [governance organization](https://cd.foundation/), a [Linux Foundation](https://www.linuxfoundation.org/projects/) project. Tekton attempts to fill the void of an in-cluster container image build and deployment workflow — that is continuous integration (CI) and continuous delivery (CD). It consists of Tekton Pipelines, which provides the building blocks, and of supporting components, such as Tekton CLI, Triggers, Catalog, that make Tekton a complete ecosystem. 

It's a Kubernetes native as Tekton installs and runs as an extension on a Kubernetes cluster and comprises a set of Kubernetes Custom Resources that define the building blocks you can create and reuse for your pipelines.
## Users of Tekton
Following are the categories for Tekton users:

* **Platform engineers** who build CI/CD systems for the developers in their organization.

* **Developers** who use CI/CD systems to do their work.

## Benefits of Tekton
Tekton provides the following benefits to builders and users of CI/CD systems:

* **Customizable.** Tekton entities are fully customizable, allowing for a high degree of flexibility. Platform engineers can define a highly detailed catalog of building blocks for developers to use in a wide variety of scenarios.
* **Reusable.** Tekton entities are fully portable, so once defined, anyone within the organization can use a given pipeline and reuse its building blocks. This allows developers to quickly build complex pipelines without “reinventing the wheel.”
* **Expandable.** Tekton Catalog is a community-driven repository of Tekton building blocks. You can quickly create new and expand existing pipelines using pre-made components from the Tekton Catalog.
* **Standardized.** Tekton installs and runs as an extension on your Kubernetes cluster and uses the well-established Kubernetes resource model. Tekton workloads execute inside Kubernetes containers.
* **Scalable.** To increase your workload capacity, you can simply add nodes to your cluster. Tekton scales with your cluster without the need to redefine your resource allocations or any other modifications to your pipelines.

**credits :** [**tekton.dev**](https://tekton.dev/docs/overview/#what-are-the-benefits-of-tekton)

## Components of Tekton
* [**Pipeline :**](https://github.com/tektoncd/pipeline) Pipeline defines a set of Kubernetes Custom Resources that act as building blocks from which you can assemble CI/CD pipelines.
* [**Triggers :**](https://github.com/tektoncd/triggers) Triggers is a Kubernetes Custom Resources that allows you to instantiate pipelines based on information it extracts from event payloads. For example, you can trigger the instantiation and execution of a pipeline every time a PR is opened against a GitHub repository.
* [**CLI :**](https://github.com/tektoncd/cli) CLI provides a command-line interface called tkn, built on top of the Kubernetes CLI, that allows you to interact with Tekton.
* [**Dashboard :**](https://github.com/tektoncd/dashboard) Dashboard is a Web-based graphical interface for Tekton Pipelines that displays information about the execution of your pipelines.
* [**Catalog :**](https://github.com/tektoncd/catalog) Catalog is a repository of high-quality, community-contributed Tekton building blocks - Tasks, Pipelines, and so on - that are ready for use in your own pipelines.
* [**Hub :**](https://github.com/tektoncd/hub) Hub is a Web-based graphical interface for accessing the Tekton Catalog.
* [**Operator :**](https://github.com/tektoncd/operators) Operator is a Kubernetes [Operator pattern](https://operatorhub.io/what-is-an-operator) that allows you to install, update, upgrade, and remove Tekton projects on Kubernetes cluster.
* [**Chains :**](https://github.com/tektoncd/chains) Chains is a Kubernetes Custom Resource Definition (CRD) controller that allows you to manage your supply chain security in Tekton. It is currently a work-in-progress.
* [**Results :**](https://github.com/tektoncd/results) Results aims to help users logically group CI/CD workload history and separate out long term result storage away from the Pipeline controller.
## Terminographical definition



Credits: https://tekton.dev/docs/concepts/concept-tasks-pipelines.png

* [**Step :**](https://github.com/tektoncd/pipeline/blob/main/docs/tasks.md#defining-steps) A step is a basic entity in CI/CD workflow, such as running some unit tests for a Python web app, or the compilation of a Java program. Tekton performs each step with a provided container image.
* [**Task :**](https://github.com/tektoncd/pipeline/blob/main/docs/tasks.md) A task is a collection of steps in order. Tekton runs a task in the form of a [Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/), where each step becomes a running container in the pod.
* [**Pipelines :**](https://github.com/tektoncd/pipeline/blob/main/docs/pipelines.md) A pipeline is a collection of tasks in order. Tekton collects all the tasks, connects them in a directed acyclic graph (DAG), and executes the graph in sequence. In other words, it creates a number of Kubernetes pods and ensures that each pod completes running successfully as desired.

Credits: https://tekton.dev/docs/concepts/concept-runs.png

* [**PipelineRun :**](https://github.com/tektoncd/pipeline/blob/main/docs/pipelineruns.md) A pipelineRun, as its name implies, is a specific execution of a pipeline.
* [**TaskRun :**](https://github.com/tektoncd/pipeline/blob/main/docs/taskruns.md) A taskRun is a specific execution of a task. TaskRuns are also available when you choose to run a task outside a pipeline, with which you may view the specifics of each step execution in a task.
#Writing Tekton Pipeline yaml from scratch for cloning the code
The below example is very simple and show how steps, task, pipeline and pipeline run are connected and to know how task will be reusable for multiple pipelines please refer [catalog](https://github.com/tektoncd/catalog) project
* **Step**
```yaml
  steps:
    - name: clone
      image: "gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/git-init:v0.21.0"
      env:
        - name: PARAM_URL
          value: $(params.url)
        - name: PARAM_REVISION
          value: $(params.revision)
        - name: WORKSPACE_OUTPUT_PATH
          value: $(workspaces.output.path)
      script: |
        #!/usr/bin/env sh
        set -eu

        CHECKOUT_DIR="${WORKSPACE_OUTPUT_PATH}"

        /ko-app/git-init \
          -url="${PARAM_URL}" \
          -revision="${PARAM_REVISION}" \
          -path="${CHECKOUT_DIR}"
        cd "${CHECKOUT_DIR}"
        EXIT_CODE="$?"
        if [ "${EXIT_CODE}" != 0 ] ; then
          exit "${EXIT_CODE}"
        fi
        # Verify clone is success by reading readme file.
        cat ${CHECKOUT_DIR}/README.md
```

* **Task**
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: git-clone
spec:
  workspaces:
    - name: output
      description: The git repo will be cloned onto the volume backing this Workspace.
  params:
    - name: url
      description: Repository URL to clone from.
      type: string
    - name: revision
      description: Revision to checkout. (branch, tag, sha, ref, etc...)
      type: string
      default: ""
  steps:
    - name: clone
      image: "gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/git-init:v0.21.0"
      env:
        - name: PARAM_URL
          value: $(params.url)
        - name: PARAM_REVISION
          value: $(params.revision)
        - name: WORKSPACE_OUTPUT_PATH
          value: $(workspaces.output.path)
      script: |
        #!/usr/bin/env sh
        set -eu

        CHECKOUT_DIR="${WORKSPACE_OUTPUT_PATH}"

        /ko-app/git-init \
          -url="${PARAM_URL}" \
          -revision="${PARAM_REVISION}" \
          -path="${CHECKOUT_DIR}"
        cd "${CHECKOUT_DIR}"
        EXIT_CODE="$?"
        if [ "${EXIT_CODE}" != 0 ] ; then
          exit "${EXIT_CODE}"
        fi
        # Verify clone is success by reading the readme file.
        cat ${CHECKOUT_DIR}/README.md
```

* **Pipeline**
```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: cat-branch-readme
spec:
  params:
    - name: repo-url
      type: string
      description: The git repository URL to clone from.
    - name: branch-name
      type: string
      description: The git branch to clone.
  workspaces:
    - name: shared-data
      description: |
        This workspace will receive the cloned git repo and be passed
        to the next Task for the repo's README.md file to be read.
  tasks:
    - name: fetch-repo
      taskRef:
        name: git-clone
      workspaces:
        - name: output
          workspace: shared-data
      params:
        - name: url
          value: $(params.repo-url)
        - name: revision
          value: $(params.branch-name)
```

* **PipelineRun**
```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: git-clone-checking-out-a-branch
spec:
  pipelineRef:
    name: cat-branch-readme
  workspaces:
    - name: shared-data
      volumeClaimTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
  params:
    - name: repo-url
      value: https://github.com/tektoncd/pipeline.git
    - name: branch-name
      value: release-v0.12.x
```

**Wrap up**

This article helps to understand how tekton introduced, how it is native to Kubernetes and its benefits
The given examples gives clear idea how one can write tekton yamls from scratch.
