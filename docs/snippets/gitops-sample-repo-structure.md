## The sample GitOps repository

<!--- cSpell:ignore serviceaccount -->

Let's understand how GitOps works in practice by using it to install the
components we've highlighted in the above diagram.

Let's first look at the high level structure of the `multi-tenancy-gitops`
GitOps repository.

1. *Ensure you're logged in to the cluster*

    !!! tip
        Ensure you're in the **terminal window** that you used to [set up
        your cluster](../gitops-tekton-argocd#setting-up-github), i.e. in the
        `multi-tenancy-gitops` subfolder.

    Log into your OCP cluster, substituting the `--token` and `--server`
    parameters with your values:

    ```bash
    oc login --token=<token> --server=<server>
    ```

    If you are unsure of these values, click your user ID in the OpenShift web
    console and select "Copy Login Command".

    <br>

2. *Locate your GitOps repository*

    If necessary, change to the root of your GitOps repository, which is stored
    in the `$GIT_ROOT` environment variable.

    Issue the following command to change to your GitOps repository:

    ```bash
    cd $GIT_ROOT
    cd multi-tenancy-gitops
    ```

    <br>

3. *Explore the high level folder structure*

    Use the following command to display the folder structure:

    ```bash
    tree . -d -L 2
    ```

    We can see the different folders in the GitOps repository:

    ```{ .text .no-copy }
    .
    ├── 0-bootstrap
    │   ├── others
    │   └── single-cluster
    ├── doc
    │   ├── diagrams
    │   ├── experimental
    │   ├── images
    │   └── scenarios
    ├── scripts
    │   └── bom
    └── setup
        ├── ocp47
        └── ocp4x
    ```

    The `0-bootstrap` folder is the key folder that contains different profiles
    for different cluster topologies. A **Cluster Profile** such as
    `single-cluster` will control resources are deployed to a single cluster.
    We'll be using the `single-cluster` profile, although you can see that other
    profiles are available.

    There are other folders containing utility scripts and some documentation;
    we'll explore these later.

    <br>

4. *The* `0-bootstrap` *folder*

    The process of installing components into a cluster is called
    **bootstrapping** because it's the first thing that happens to a cluster
    after it has been created. We will bootstrap our cluster using the
    `0-bootstrap` folder in the `multi-tenancy` repository.

    Let us examine the `0-bootstrap` folder structure:

    ```bash
    tree ./0-bootstrap/ -d -L 2
    ```

    We can see the different folders:

    ```{ .text .no-copy }
    ./0-bootstrap/
    ├── others
    │   ├── 1-shared-cluster
    │   ├── 2-isolated-cluster
    │   └── 3-multi-cluster
    └── single-cluster
        ├── 1-infra
        ├── 2-services
        └── 3-apps
    ```

    Notice the different **cluster profiles**. We're going to use the
    `single-cluster` profile. See how this profile has three sub-folders
    corresponding to three layers of components: **infrastructure**,
    **services** and **applications**. Every component in our architecture will
    be in one of these layers.

    <br>

5. *The* `single-cluster` *profile in more detail*

    Use the following command to display `single-cluster` folder in more detail:

    ```bash
    tree ./0-bootstrap/single-cluster/ -L 2
    ```

    We can see the different folders:

    ```{ .text .no-copy }
    ./0-bootstrap/single-cluster/
    ├── 1-infra
    │   ├── 1-infra.yaml
    │   ├── argocd
    │   └── kustomization.yaml
    ├── 2-services
    │   ├── 2-services.yaml
    │   ├── argocd
    │   └── kustomization.yaml
    ├── 3-apps
    │   ├── 3-apps.yaml
    │   ├── argocd
    │   └── kustomization.yaml
    ├── bootstrap.yaml
    └── kustomization.yaml
    ```

    Again, see the different layers of the architecture: **infrastructure**,
    **service** and **application**.

    Notice how each of these high level folders (`1-infra`, `2-services`,
    `3-applications`) has an `argocd` folder. These `argocd` folders contain the
    **ArgoCD applications** that control which resources in that architectural
    layer are deployed to the cluster.

    <br>

6. *ArgoCD applications*

    Later in this tutorial, we'll see in detail how these ArgoCD applications
    work. For now, let's explore the range of ArgoCD applications that control
    the infrastructure components deployed to the cluster.

    Type the following command :

    ```bash
    tree 0-bootstrap/single-cluster/1-infra/
    ```

    It shows a list of ArgoCD applications that are used to manage Kubernetes
    **infrastructure** resources:

    ```{ .text .no-copy }
    0-bootstrap/single-cluster/1-infra/
    ├── 1-infra.yaml
    ├── argocd
    │   ├── consolelink.yaml
    │   ├── consolenotification.yaml
    │   ├── infraconfig.yaml
    │   ├── machinesets.yaml
    │   ├── namespace-baas.yaml
    │   ├── namespace-ci.yaml
    │   ├── namespace-cloudpak.yaml
    │   ├── namespace-db2.yaml
    │   ├── namespace-dev.yaml
    │   ├── namespace-ibm-common-services.yaml
    │   ├── namespace-instana-agent.yaml
    │   ├── namespace-istio-system.yaml
    │   ├── namespace-mq.yaml
    │   ├── namespace-openldap.yaml
    │   ├── namespace-openshift-storage.yaml
    │   ├── namespace-prod.yaml
    │   ├── namespace-robot-shop.yaml
    │   ├── namespace-sealed-secrets.yaml
    │   ├── namespace-spp-velero.yaml
    │   ├── namespace-spp.yaml
    │   ├── namespace-staging.yaml
    │   ├── namespace-tools.yaml
    │   ├── scc-wkc-iis.yaml
    │   ├── serviceaccounts-db2.yaml
    │   ├── serviceaccounts-ibm-common-services.yaml
    │   ├── serviceaccounts-mq.yaml
    │   ├── serviceaccounts-tools.yaml
    │   └── storage.yaml
    └── kustomization.yaml
    ```

    Notice the many `namespace-` YAMLs; we'll see in a moment how these each
    define an ArgoCD application dedicated to managing a Kubernetes namespace in
    our cluster. the `serviceaccount-` YAMLs can similarly manage service
    accounts.

    There are similar ArgoCD applications for the **service** and
    **application** layers.  Feel free to examine their corresponding folders;
    we will look at them in more detail later.

    <br>

7. *A word on terminology*

    As we get started with ArgoCD, it can be easy to confuse the term **ArgoCD
    application** with your application. That's because ArgoCD uses the term
    **application** to refer to a Kubernetes **custom resource** that was
    initially designed to manage a set of application resources. However, an
    **ArgoCD application** can automate the deployment of any Kubernetes
    resource within a cluster, such as a namespace, as we'll see a little later.
