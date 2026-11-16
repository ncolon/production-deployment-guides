## Explore the -apps repository

<!--- cSpell:ignore apic configmaps -->

The `multi-tenancy-gitops-apps` repository contains the YAMLs of the application
components running in the cluster. For example, if there are 10 applications, 2
databases, 4 queue managers and 2 workflow engines running in the cluster, then
there will be YAMLs in this repository corresponding to each of these instances.

Let's explore the structure of the `multi-tenancy-gitops-apps` repository to see
how it works.

1. *Ensure environment variables are set*

    !!! tip
        Ensure you're in the `multi-tenancy-gitops-apps`  **terminal
        window**.

    !!! tip
        If you're returning to the tutorial after restarting your computer,
        ensure that the $GIT_ORG, $GIT_BRANCH and $GIT_ROOT environment
        variables are set.

    (Replace `<your organization name>` appropriately:)

    ```bash
    export GIT_BRANCH=master
    export GIT_ORG=<your organization name>
    export GIT_ROOT=$HOME/git/$GIT_ORG-root
    ```

    You can verify your environment variables as follows:

    ```bash
    echo $GIT_BRANCH
    echo $GIT_ORG
    echo $GIT_ROOT
    ```

    <br>

2. *Locate the multi-tenancy-gitops-apps repository*

    Issue the following command to change to your GitOps repository:

    ```bash
    cd $GIT_ROOT
    cd multi-tenancy-gitops-apps
    ```

    <br>

3. *A folder for each application component type*

    Let's explore the top level folder structure of the the repository.

    Issue the following command:

    ```bash
    tree . -L 1
    ```

    to show the top level folders:

    ```{ .text .no-copy }
    .
    ├── ace
    ├── apic
    ├── cntk
    ├── cp4a
    ├── mq
    ├── pm
    ├── scripts
    ├── shared
    ├── slack-notifications
    └── soapserver
    ```

    Notice that there's one high level folder for each of the different
    application components deployed to the cluster. For example, the YAMLs
    defining the API gateways, ACE integration servers and MQ queue managers are
    stored in the `apic`, `ace` and `mq` folders, respectively.

    <br>

4. *An application component folder*

    Each of these application component folders has a similar structure. Let's
    look at the MQ folder as a typical example.

    Issue the following command:

    ```bash
    tree ./mq -L 2
    ```

    to show the structure of the MQ folder:

    ```{ .text .no-copy }
    ./mq
    ├── config
    │   └── argocd
    └── environments
        ├── base
        ├── ci
        ├── dev
        ├── prod
        ├── staging
        └── tools
    ```

    Notice how:

      * the `config/argocd` folder contains the ArgoCD applications that manage the MQ
        application components deployed to the cluster.
      * the `environments` folder contains one subfolder for each environment on
        the cluster, including MQ applications and queue managers.

    <br>

5. *ArgoCD applications*

    Let's explore the folder structure to see how ArgoCD manages the cluster for
    these application components.

    Issue the following command:

    ```bash
    tree ./mq/config/argocd -L 2
    ```

    to show the ArgoCD applications:

    ```{ .text .no-copy }
    ./mq/config/argocd
    ├── ci
    │   ├── ci-app-rest.yaml
    │   └── ci-scc.yaml
    ├── dev
    │   ├── dev-mq-metric-samples-instance.yaml
    │   ├── dev-mq-qm01-instance.yaml
    │   └── dev-mq-spring-app-instance.yaml
    ├── prod
    │   ├── prod-mq-metric-samples-instance.yaml
    │   ├── prod-mq-qm01-instance.yaml
    │   └── prod-mq-spring-app-instance.yaml
    ├── staging
    │   ├── staging-mq-metric-samples-instance.yaml
    │   ├── staging-mq-qm01-instance.yaml
    │   └── staging-mq-spring-app-instance.yaml
    └── tools
        └── tools-cert-manager-artifacts.yaml
    ```

    Notice how:

    * The `config/argocd` folder contains ArgoCD applications organized by
      folder namespace.
    * The name of the ArgoCD applications corresponds to the resource it
      manages; for example `dev/dev-mq-qm01-instance.yaml` will manage a queue
      manager in the `dev` namespace.

    <br>

    The number of ArgoCD YAMLs in these folders will grow as application
    components are added to the cluster. For this tutorial, we've pre-populated
    the repository with the ArgoCD YAMLs for the initial application components we're
    going to deploy to the cluster.

    <br>

---

## ArgoCD example

All ArgoCD applications work in a similar way. Let's examine one in a little
more detail.

1. *An example ArgoCD application*

    Let's look at the YAML of a typical ArgoCD application that manages a queue
    manager.

    Issue the following command:

    ```bash
    cat ./mq/config/argocd/dev/dev-mq-qm01-instance.yaml
    ```

    to see the YAML to manage an MQ queue manager running in the cluster:

    ``` { .text .no-copy }
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: dev-mq-qm01-instance
      annotations:
        argocd.argoproj.io/sync-wave: "300"
      finalizers:
        - resources-finalizer.argocd.argoproj.io
    spec:
      destination:
        namespace: dev
        server: https://kubernetes.default.svc
      project: applications
      source:
        path: mq/environments/dev/mq-qm01/
        repoURL: https://github.com/cloud-native-toolkit-demos/multi-tenancy-gitops-apps.git
        targetRevision: master
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - Replace=true
    ```

    Notice how this ArgoCD application will use:

    * the `path: mq/environments/mq-infra/overlays/dev` element to identify the
      folder location containing the queue manager YAMLs that will be applied to
      the cluster.
    * the `repoURL:` identifies the GitHUb repository; we'll need to customize
      this value for our deployment.

    You can learn more about the different YAML elements in an ArgoCD
    application in the [ArgoCD API
    reference](https://argocd-operator.readthedocs.io/en/latest/reference/argocd/).

    <br>

    There are **many** ArgoCD applications like this in the `-apps` repository; each
    one is responsible for managing the deployment of any application component
    to the cluster, including its related resources such as configmaps and
    certificates.

    <br>

2. *The* `environments` *folder*

    As we've seen in our example, the ArgoCD applications refer to YAMLs in the
    `environments` folder; let's explore the `environments` folder in more
    detail.

    Issue the following command:

    ```bash
    tree ./mq/environments -L 2
    ```

    to see the wide range of YAMLs in this folder:

    ```{ .text .no-copy }
    ./mq/environments
    ├── base
    │   ├── kustomization.yaml
    │   ├── mq-metric-samples
    │   ├── mq-qm01
    │   └── mq-spring-app
    ├── ci
    │   ├── certificates
    │   ├── configmaps
    │   ├── eventlisteners
    │   ├── kustomization.yaml
    │   ├── pipelines
    │   ├── roles
    │   ├── routes
    │   ├── scc
    │   ├── secrets
    │   ├── tasks
    │   ├── triggerbindings
    │   └── triggertemplates
    ├── dev
    │   ├── kustomization.yaml
    │   ├── mq-metric-samples
    │   ├── mq-qm01
    │   └── mq-spring-app
    ├── prod
    │   ├── kustomization.yaml
    │   ├── mq-metric-samples
    │   ├── mq-qm01
    │   └── mq-spring-app
    ├── staging
    │   ├── kustomization.yaml
    │   ├── mq-metric-samples
    │   ├── mq-qm01
    │   └── mq-spring-app
    └── tools
        ├── certificates
        ├── cluster-issuers
        └── kustomization.yaml

    31 directories, 6 files
    ```

    As the cluster grows, more YAMLs will be added to the `environments` folder.
    These YAMLs are referenced by corresponding ArgoCD YAMLs in the
    `.../config/argocd/` folder.

    <br>

3. *The resource YAMLs*

    When a pipeline successfully builds and tests an application component from
    a source repository, it stores its output YAML in the `-apps` repository.

    Issue the following command:

    ```bash
    tree ./mq/environments/ -L 3
    ```

    to show the contents of the folder that will contain the YAMLs for the
    `mq-qm01`, `mq-spring-app` and `mq-metric-samples` :

    ```{ .text .no-copy }
    ./mq/environments/
    ├── base
    │   ├── kustomization.yaml
    │   ├── mq-metric-samples
    │   │   ├── deployment.yaml
    │   │   ├── kustomization.yaml
    │   │   ├── role.yaml
    │   │   ├── rolebinding.yaml
    │   │   ├── route.yaml
    │   │   ├── secret.yaml
    │   │   ├── service.yaml
    │   │   ├── serviceaccount.yaml
    │   │   └── servicemonitor.yaml
    │   ├── mq-qm01
    │   │   ├── kustomization.yaml
    │   │   └── queuemanager.yaml
    │   └── mq-spring-app
    │       ├── deployment.yaml
    │       ├── jaegerconfigmap.yaml
    │       ├── kustomization.yaml
    │       ├── route.yaml
    │       ├── service.yaml
    │       └── serviceaccount.yaml
    ├── ci
    │   ├── certificates
    │   │   ├── ci-mq-client-certificate.sh
    │   │   ├── ci-mq-client-certificate.yaml_template
    │   │   ├── ci-mq-server-certificate.sh
    │   │   └── ci-mq-server-certificate.yaml_template
    │   ├── configmaps
    │   │   ├── gitops-repo-configmap.sh
    │   │   └── gitops-repo-configmap.yaml_template
    │   ├── eventlisteners
    │   │   ├── cntk-event-listener.sh
    │   │   ├── cntk-event-listener.yaml_template
    │   │   ├── mq-qm-dev-eventlistener.yaml
    │   │   ├── mq-qm-stage-eventlistener.yaml
    │   │   ├── mq-spring-app-dev-eventlistener.yaml
    │   │   └── mq-spring-app-stage-eventlistener.yaml
    │   ├── kustomization.yaml
    │   ├── pipelines
    │   │   ├── ibm-test-pipeline-for-dev.yaml
    │   │   ├── ibm-test-pipeline-for-stage.yaml
    │   │   ├── java-maven-dev-pipeline.yaml
    │   │   ├── mq-metric-samples-dev-pipeline.yaml
    │   │   ├── mq-pipeline-dev.yaml
    │   │   └── mq-spring-app-dev-pipeline.yaml
    │   ├── roles
    │   │   ├── custom-ci-pipeline-sa-rolebinding.yaml
    │   │   ├── custom-dev-pipeline-sa-rolebinding.yaml
    │   │   ├── custom-pipeline-sa-clusterrole.yaml
    │   │   ├── custom-pipeline-sa-role.yaml
    │   │   ├── custom-prod-pipeline-sa-rolebinding.yaml
    │   │   └── custom-staging-pipeline-sa-rolebinding.yaml
    │   ├── routes
    │   │   ├── cntk-route.yaml
    │   │   ├── mq-qm-dev-route.yaml
    │   │   ├── mq-qm-stage-route.yaml
    │   │   ├── mq-spring-app-dev-route.yaml
    │   │   └── mq-spring-app-stage-route.yaml
    │   ├── scc
    │   │   ├── Chart.yaml
    │   │   └── values.yaml
    │   ├── secrets
    │   │   ├── artifactory-access-secret.sh
    │   │   ├── artifactory-access-secret.yaml
    │   │   ├── git-credentials-secret.sh
    │   │   ├── git-credentials-secret.yaml
    │   │   ├── ibm-entitled-registry-credentials-secret.sh
    │   │   ├── ibm-entitled-registry-credentials-secret.yaml
    │   │   ├── mq-client-jks-password-secret.sh
    │   │   └── mq-client-jks-password-secret.yaml
    │   ├── tasks
    │   │   ├── 10-gitops-for-mq.yaml
    │   │   ├── 10-gitops-promotion.yaml
    │   │   ├── 10-gitops.yaml
    │   │   ├── 11-app-name.yaml
    │   │   ├── 12-functional-tests.yaml
    │   │   ├── 13-cphtestp-performance-test.yaml
    │   │   ├── 13-jmeter-performance-test.yaml
    │   │   ├── 4-smoke-tests-mq.yaml
    │   │   ├── 4-smoke-tests.yaml
    │   │   ├── ibm-build-tag-push-v2-6-13.yaml
    │   │   ├── ibm-helm-release-v2-6-13.yaml
    │   │   ├── ibm-img-release-v2-6-13.yaml
    │   │   ├── ibm-img-scan-v2-6-13.yaml
    │   │   ├── ibm-java-maven-test-v2-6-13.yaml
    │   │   ├── ibm-setup-v2-6-13.yaml
    │   │   ├── ibm-tag-release-v2-6-13.yaml
    │   │   └── mq-metrics-build-tag-push.yaml
    │   ├── triggerbindings
    │   │   └── cntk-binding.yaml
    │   └── triggertemplates
    │       ├── mq-qm-dev-triggertemplate.sh
    │       ├── mq-qm-dev-triggertemplate.yaml_template
    │       ├── mq-qm-dev.yaml
    │       ├── mq-qm-stage-triggertemplate.sh
    │       ├── mq-qm-stage-triggertemplate.yaml_template
    │       ├── mq-spring-app-dev-triggertemplate.sh
    │       ├── mq-spring-app-dev-triggertemplate.yaml_template
    │       ├── mq-spring-app-dev.yaml
    │       ├── mq-spring-app-stage-triggertemplate.sh
    │       └── mq-spring-app-stage-triggertemplate.yaml_template
    ├── dev
    │   ├── kustomization.yaml
    │   ├── mq-metric-samples
    │   │   ├── components
    │   │   ├── configmap
    │   │   ├── kustomization.yaml
    │   │   └── mq-metric-samples
    │   ├── mq-qm01
    │   │   ├── certificates
    │   │   ├── components
    │   │   ├── configmap
    │   │   ├── kustomization.yaml
    │   │   ├── queuemanager
    │   │   └── secrets
    │   └── mq-spring-app
    │       ├── certificates
    │       ├── components
    │       ├── configmap
    │       ├── deployment
    │       ├── kustomization.yaml
    │       └── secrets
    ├── prod
    │   ├── kustomization.yaml
    │   ├── mq-metric-samples
    │   │   ├── components
    │   │   ├── configmap
    │   │   ├── kustomization.yaml
    │   │   └── mq-metric-samples
    │   ├── mq-qm01
    │   │   ├── certificates
    │   │   ├── components
    │   │   ├── configmap
    │   │   ├── kustomization.yaml
    │   │   ├── queuemanager
    │   │   └── secrets
    │   └── mq-spring-app
    │       ├── certificates
    │       ├── components
    │       ├── configmap
    │       ├── deployment
    │       ├── kustomization.yaml
    │       └── secrets
    ├── staging
    │   ├── kustomization.yaml
    │   ├── mq-metric-samples
    │   │   ├── components
    │   │   ├── configmap
    │   │   ├── kustomization.yaml
    │   │   └── mq-metric-samples
    │   ├── mq-qm01
    │   │   ├── certificates
    │   │   ├── components
    │   │   ├── configmap
    │   │   ├── kustomization.yaml
    │   │   ├── queuemanager
    │   │   ├── roles
    │   │   └── secrets
    │   └── mq-spring-app
    │       ├── certificates
    │       ├── components
    │       ├── configmap
    │       ├── deployment
    │       ├── kustomization.yaml
    │       └── secrets
    └── tools
        ├── certificates
        │   └── mq-selfsigned-certificate
        ├── cluster-issuers
        │   ├── mq-selfsigned-clusterissuer
        │   └── selfsigned-clusterissuer
        └── kustomization.yaml

    74 directories, 99 files
    ```

    Notice how:

    * There is a folder corresponding to the `dev`, `stage` and `prod`
      instances of the queue manager, sample spring applications and metrics sample.
    * Each of these folders will use Kustomize to override the `base` YAMLs.

    <br>

    We'll explore this folder structure in more detail as we progress through
    the tutorial.

    <br>
