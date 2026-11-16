## Connect ArgoCD to the GitOps repository

<!--- cSpell:ignore resynched uncommenting operatorgroup clusterrole clusterrolebinding consolenotification subfolders rolebindings rolebinding CICD qube cntk autoplay allowfullscreen -->

Let's now connect your customized GitOps repository to the instance of ArgoCD
running in the cluster. Once connected, ArgoCD will use the contents of the
repository to create necessary resources.

  1. *Locate your GitOps repository*

    If necessary, change to the root of your GitOps repository, which is stored
    in the `$GIT_ROOT` environment variable.

    Issue the following command to change to your GitOps repository:

    ```bash
    cd $GIT_ROOT
    cd multi-tenancy-gitops
    ```

  2. *Review ArgoCD infrastructure folder*

    Let's examine the `0-bootstrap/single-cluster/1-infra/kustomization.yaml`
    to see how ArgoCD manages the resources deployed to the cluster.

    Issue the following command:

    ```bash
    cat 0-bootstrap/single-cluster/1-infra/kustomization.yaml
    ```

    We can see the contents of the `kustomization.yaml`:

    ```{ .yaml .no-copy }
    resources:
    #- argocd/consolelink.yaml
    #- argocd/consolenotification.yaml
    #- argocd/namespace-ibm-common-services.yaml
    #- argocd/namespace-ci.yaml
    #- argocd/namespace-dev.yaml
    #- argocd/namespace-staging.yaml
    #- argocd/namespace-prod.yaml
    #- argocd/namespace-cloudpak.yaml
    #- argocd/namespace-istio-system.yaml
    #- argocd/namespace-openldap.yaml
    #- argocd/namespace-sealed-secrets.yaml
    #- argocd/namespace-tools.yaml
    #- argocd/namespace-instana-agent.yaml
    #- argocd/namespace-robot-shop.yaml
    #- argocd/namespace-openshift-storage.yaml
    #- argocd/namespace-spp.yaml
    #- argocd/namespace-spp-velero.yaml
    #- argocd/namespace-baas.yaml
    #- argocd/namespace-db2.yaml
    #- argocd/namespace-mq.yaml
    #- argocd/serviceaccounts-ibm-common-services.yaml
    #- argocd/serviceaccounts-tools.yaml
    #- argocd/serviceaccounts-db2.yaml
    #- argocd/serviceaccounts-mq.yaml
    #- argocd/scc-wkc-iis.yaml
    #- argocd/storage.yaml
    #- argocd/infraconfig.yaml
    #- argocd/machinesets.yaml
    patches:
    - target:
        group: argoproj.io
        kind: Application
        labelSelector: "gitops.tier.layer=infra"
      patch: |-
        - op: add
          path: /spec/source/repoURL
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops-infra.git
        - op: add
          path: /spec/source/targetRevision
          value: master
    ```

    Notice that the resources that needs to be applied to the cluster are all
    `inactivate` and commented out.

    Let us `enable` the resources that are needed by un-commenting them.

  3. *Deploy Kubernetes resources with* `kustomization.yaml`

    Open `0-bootstrap/single-cluster/1-infra/kustomization.yaml` and uncomment the below resources:

    ```{ .yaml .no-copy }
    argocd/consolenotification.yaml
    argocd/namespace-ibm-common-services.yaml
    argocd/namespace-sealed-secrets.yaml
    argocd/namespace-tools.yaml
    ```

    You will have the following resources un-commented for infrastructure:

    ```{ .yaml .no-copy }
    resources:
    #- argocd/consolelink.yaml
    - argocd/consolenotification.yaml
    - argocd/namespace-ibm-common-services.yaml
    #- argocd/namespace-ci.yaml
    #- argocd/namespace-dev.yaml
    #- argocd/namespace-staging.yaml
    #- argocd/namespace-prod.yaml
    #- argocd/namespace-cloudpak.yaml
    #- argocd/namespace-istio-system.yaml
    #- argocd/namespace-openldap.yaml
    - argocd/namespace-sealed-secrets.yaml
    - argocd/namespace-tools.yaml
    #- argocd/namespace-instana-agent.yaml
    #- argocd/namespace-robot-shop.yaml
    #- argocd/namespace-openshift-storage.yaml
    #- argocd/namespace-spp.yaml
    #- argocd/namespace-spp-velero.yaml
    #- argocd/namespace-baas.yaml
    #- argocd/namespace-db2.yaml
    #- argocd/namespace-mq.yaml
    #- argocd/serviceaccounts-ibm-common-services.yaml
    #- argocd/serviceaccounts-tools.yaml
    #- argocd/serviceaccounts-db2.yaml
    #- argocd/serviceaccounts-mq.yaml
    #- argocd/scc-wkc-iis.yaml
    #- argocd/storage.yaml
    #- argocd/infraconfig.yaml
    #- argocd/machinesets.yaml
    patches:
    - target:
        group: argoproj.io
        kind: Application
        labelSelector: "gitops.tier.layer=infra"
      patch: |-
        - op: add
          path: /spec/source/repoURL
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops-infra.git
        - op: add
          path: /spec/source/targetRevision
          value: master
    ```

    Commit and push changes to your git repository:

    ```bash
    git add .
    git commit -s -m "Intial boostrap setup for infrastructure"
    git push origin $GIT_BRANCH
    ```

    The changes have now been pushed to your GitOps repository:

    ```{ .text .no-copy }
    [master aa591ea] Intial boostrap setup for infrastructure
     1 file changed, 4 insertions(+), 4 deletions(-)
    Enumerating objects: 11, done.
    Counting objects: 100% (11/11), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (6/6), done.
    Writing objects: 100% (6/6), 576 bytes | 576.00 KiB/s, done.
    Total 6 (delta 5), reused 0 (delta 0)
    remote: Resolving deltas: 100% (5/5), completed with 5 local objects.
    To https://github.com/tutorial-org-123/multi-tenancy-gitops.git
       a900c39..aa591ea  master -> master
    ```

  4. *Examine* `bootstrap.yaml` residing in `0-bootstrap/single-cluster/`.

    The `bootstrap.yaml` file is used to create our first ArgoCD application
    called `bootstrap-single-cluster`. This initial ArgoCD application will create
    all the other ArgoCD applications that control the application, service, and
    infrastructure resources (such as the tools and ibm-common-services namespaces) deployed to the
    cluster.

    Examine the YAML that defines the ArgoCD bootstrap application:

    ```bash
    cat 0-bootstrap/single-cluster/bootstrap.yaml
    ```

    Notice also how this ArgoCD application has been customized to use the GitOps
    repository repoURL: `https://github.com/tutorial-org-123/multi-tenancy-gitops.git`.

    ```{ .yaml .no-copy }
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: bootstrap-single-cluster
      namespace: openshift-gitops
    spec:
      destination:
        namespace: openshift-gitops
        server: https://kubernetes.default.svc
      project: default
      source:
        path: 0-bootstrap/single-cluster
        repoURL: https://github.com/tutorial-org-123/multi-tenancy-gitops.git
        targetRevision: master
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    ```

    Most importantly, see how `path: 0-bootstrap/single-cluster` refers to the
    `0-bootstrap/single-cluster` folder within this repository. This will result
    in the creation of individual ArgoCD applications to manage our cluster
    resources.

    Access the `0-bootstrap/single-cluster/kustomization.yaml`:

    ```bash
    cat 0-bootstrap/single-cluster/kustomization.yaml
    ```

    Let us for now only deploy `infra` resources to the cluster. Open `0-bootstrap/single-cluster/kustomization.yaml`
    and comment out the `2-services/2-services.yaml` and `3-apps/3-apps.yaml` as follows:

    ```{ .yaml .no-copy }
    resources:
    - 1-infra/1-infra.yaml
    # - 2-services/2-services.yaml
    # - 3-apps/3-apps.yaml
    patches:
    - target:
        group: argoproj.io
        kind: Application
        labelSelector: "gitops.tier.layer=gitops"
      patch: |-
        - op: add
          path: /spec/source/repoURL
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops.git
        - op: add
          path: /spec/source/targetRevision
          value: master
    - target:
        group: argoproj.io
        kind: AppProject
        labelSelector: "gitops.tier.layer=infra"
      patch: |-
        - op: add
          path: /spec/sourceRepos/-
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops.git
        - op: add
          path: /spec/sourceRepos/-
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops-infra.git
    - target:
        group: argoproj.io
        kind: AppProject
        labelSelector: "gitops.tier.layer=services"
      patch: |-
        - op: add
          path: /spec/sourceRepos/-
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops.git
        - op: add
          path: /spec/sourceRepos/-
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops-services.git
    - target:
        group: argoproj.io
        kind: AppProject
        labelSelector: "gitops.tier.layer=applications"
      patch: |-
        - op: add
          path: /spec/sourceRepos/-
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops.git
        - op: add
          path: /spec/sourceRepos/-
          value: https://github.com/tutorial-org-123/multi-tenancy-gitops-apps.git
    ```

    Commit and push changes to your git repository:

    ```bash
    git add .
    git commit -s -m "Using only infra"
    git push origin $GIT_BRANCH
    ```

    The changes have now been pushed to your GitOps repository:

    ```{ .text .no-copy }
    [master aeb56c6] Using only infra
     1 file changed, 2 insertions(+), 2 deletions(-)
    Enumerating objects: 9, done.
    Counting objects: 100% (9/9), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (5/5), done.
    Writing objects: 100% (5/5), 456 bytes | 456.00 KiB/s, done.
    Total 5 (delta 4), reused 0 (delta 0)
    remote: Resolving deltas: 100% (4/4), completed with 4 local objects.
    To https://github.com/tutorial-org-123/multi-tenancy-gitops.git
       e3f696d..aeb56c6  master -> master
    ```

  5. *Apply ArgoCD* `bootstrap.yaml`

    Recall that you pushed the customized local copy of the GitOps repository to
    your GitHub account. The repository contains a `bootstrap-single-cluster` ArgoCD application
    that is watching this repository and using its contents to manage the cluster.

    When the `bootstrap-single-cluster` ArgoCD application is applied to the cluster, it will
    **continuously** ensure that all the activated resources are applied to the
    cluster.

    Apply the bootstrap YAML to the cluster:

    ```bash
    oc apply -f 0-bootstrap/single-cluster/bootstrap.yaml
    ```

    Kubernetes will confirm that the `bootstrap` **ArgoCD application** has been
    created:

    ```{ .text .no-copy }
    application.argoproj.io/bootstrap-single-cluster created
    ```

    The `bootstrap` ArgoCD application will watch the `0-bootstrap/single-cluster`
    folder in our GitOps repository on GitHub.

    In this way, as resources are added to the **infrastructure**, **service** and
    **application** folders, they will be deployed to the cluster automatically.

    This is therefore the only direct cluster operation we need to perform; from
    now on, all cluster operations will be performed via Git operations to this
    repository.

  6. *Verify the bootstrap deployment*

    Verify that the bootstrap ArgoCD application is running with the following
    command:

    ```bash
    oc get app/bootstrap-single-cluster -n openshift-gitops
    ```

    You should see that the bootstrap application was recently updated:

    ```{ .text .no-copy }
    NAME                       SYNC STATUS   HEALTH STATUS
    bootstrap-single-cluster   Synced        Healthy
    ```

    `HEALTH_STATUS` may temporarily show `Missing`; simply re-issue the command to
    confirm it moves to `Healthy`.

  7. *Using the UI to view the newly deployed ArgoCD applications*

    In the previous section of this chapter you logged on to the ArgoCD web
    console. Switch back to that console, refresh the page and you should see
    the `bootstrap-single-cluster` ArgoCD application together with many other
    ArgoCD applications:

    !!! tip
        You may need to select `List` view rather than the `Tiles` view.

    ![argocd61](/snippets/images-gitops-argocd-connect/argo61-cp4d.png){: style="max-height:400px"}

    We can see that **six** ArgoCD applications have been deployed to the
    cluster as a result of applying `bootstrap.yaml`. In the next section of the
    tutorial, we'll examine these applications to see *how and why* they were
    created, but for now let's focus on one of them -- the `namespace-tools` ArgoCD
    application.

  8. *Examining the* `namespace-tools` *ArgoCD application resources*

    Let's examine the Kubernetes resources applied to the cluster by the
    `namespace-tools` ArgoCD application.

    In the ArgoCD application list, click on `namespace-tools`:

    ![argocd62](/snippets/images-gitops-argocd-connect/argo62-cp4d.png){: style="max-height:400px"}

    (You may need to `clear filters` to see this screenshot.)

    The directed graph shows that the `namespace-tools` ArgoCD app has created 2
    Kubernetes resources; our `tools` namespace and one operatorgroup.

  9. *Verify the namespace using the* `oc` *CLI*

    We've seen the new namespace definition in the GitOps repository and visually
    in the ArgoCD UI. Let's also verify it via the command line:

    Type the following command:

    ```bash
    oc get namespace tools -o yaml
    ```
    
    If you have a newer version of oc cli and want to view the managed fields:

    ```bash
    oc get namespace tools -o yaml --show-managed-fields=true
    ```

    This will list the full details of the `tools` namespace:

    ```{ .yaml .no-copy }
    apiVersion: v1
    kind: Namespace
    metadata:
      annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
          {"apiVersion":"v1","kind":"Namespace","metadata":{"annotations":{},"labels":{"app.kubernetes.io/instance":"namespace-tools"},"name":"tools"}}
        openshift.io/sa.scc.mcs: s0:c26,c15
        openshift.io/sa.scc.supplemental-groups: 1000680000/10000
        openshift.io/sa.scc.uid-range: 1000680000/10000
      creationTimestamp: "2022-01-23T04:23:38Z"
      labels:
        app.kubernetes.io/instance: namespace-tools
        olm.operatorgroup.uid/a74fb725-eb30-4a32-98d8-9f9913854bf5: ""
      managedFields:
      - apiVersion: v1
        fieldsType: FieldsV1
        fieldsV1:
          f:metadata:
            f:annotations:
              .: {}
              f:kubectl.kubernetes.io/last-applied-configuration: {}
            f:labels:
              .: {}
              f:app.kubernetes.io/instance: {}
          f:status:
            f:phase: {}
        manager: argocd-application-controller
        operation: Update
        time: "2022-01-23T04:23:38Z"
      - apiVersion: v1
        fieldsType: FieldsV1
        fieldsV1:
          f:metadata:
            f:annotations:
              f:openshift.io/sa.scc.mcs: {}
              f:openshift.io/sa.scc.supplemental-groups: {}
              f:openshift.io/sa.scc.uid-range: {}
        manager: cluster-policy-controller
        operation: Update
        time: "2022-01-23T04:23:38Z"
      - apiVersion: v1
        fieldsType: FieldsV1
        fieldsV1:
          f:metadata:
            f:labels:
              f:olm.operatorgroup.uid/a74fb725-eb30-4a32-98d8-9f9913854bf5: {}
        manager: olm
        operation: Update
        time: "2022-01-23T04:23:39Z"
      name: tools
      resourceVersion: "773049"
      selfLink: /api/v1/namespaces/tools
      uid: 8edcda20-b9a8-45e9-9c32-81d663658e65
    spec:
      finalizers:
      - kubernetes
    status:
      phase: Active
    ```

    Notice how `manager: argocd-application-controller` identifies that this
    namespace was created by ArgoCD.

    It's important to understand the sequence of actions. We simply deployed the
    `bootstrap-single-cluster` ArgoCD application, and it ultimately resulted in
    the creation of the `namespace-tools` ArgoCD application which created the `tools`
    namespace.

    We don't directly apply resources to the cluster once the `bootstrap-single-cluster`
    ArgoCD application has been applied; the cluster state is determined by an ArgoCD
    application YAML in the corresponding **application**, **service** or
    **infrastructure** related folders. It's these ArgoCD applications that
    create and manage the underlying Kubernetes resources using the GitOps
    repository as the source of truth.

  10. *Understanding the* `namespace-tools` ArgoCD *application*

    Let's examine the ArgoCD application `namespace-tools` to see how it created the
    `tools` namespace and one operatorgroup in the cluster.

    Issue the following command to examine its YAML:

    ```bash
    cat 0-bootstrap/single-cluster/1-infra/argocd/namespace-tools.yaml
    ```

    Notice that `apiVersion` and `kind` identify this as an ArgoCD application:

    ```{ .yaml .no-copy }
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: namespace-tools
      labels:
        gitops.tier.layer: infra
      annotations:
        argocd.argoproj.io/sync-wave: "100"
    spec:
      destination:
        namespace: tools
        server: https://kubernetes.default.svc
      project: infra
      source:
        path: namespaces/tools
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    ```

    Most importantly, see how this `namespace-tools` ArgoCD application monitors the
    folder `path: namespaces/tools` in `https://github.com/tutorial-org-123/multi-tenancy-gitops-infra.git`;
    it then applies the contents of this folder to the cluster whenever its content changes.

    Notice the `SyncPolicy` of `automated`; any changes to this folder will
    automatically be applied to the cluster; we do not need to perform a manual
    `Sync` operation from the ArgoCD UI.

  11. *Examine the* `tools` *namespace YAML*

    To examine this, navigate to `multi-tenancy-gitops-infra` repository you cloned previously.

    ```bash
    cd $GIT_ROOT
    cd multi-tenancy-gitops-infra
    ```

    Let's examine the `tools` namespace YAML in the `namespaces/tools` folder:

    ```bash
    cat namespaces/tools/namespace.yaml
    ```

    It's a very simple YAML:

    ```{ .yaml .no-copy }
    apiVersion: v1
    kind: Namespace
    metadata:
      name: tools
    ```

    This is a YAML that we would normally apply to the cluster manually or via a
    script. However, when we use GitOps, we push the **ArgoCD application** that
    uses this YAML to GitHub, and it applies the `tools` namespace YAML to the
    cluster. This is the essence of GitOps;  we **declare** what we want to appear
    in the cluster using Git and ArgoCD synchronizes the cluster with this
    declaration.

  12. *Examine the* `tools` *operatorgroup YAML*

    Now that we've seen how the namespace was created, let's see how the
    operatorgroup was created by the `namespace-tools` ArgoCD application.

    In the same `namespace/tools` folder as the `tools` namespace YAML, there is a
    `operatorgroup.yaml` file. This file will be also applied to the cluster by the
    `namespace-tools` ArgoCD application which is continuously watching this folder.

    Examine this file with the following command:

    ```bash
    cat namespaces/tools/operatorgroup.yaml
    ```

    This YAML is quite straightforward.

    ```{ .yaml .no-copy }
    apiVersion: operators.coreos.com/v1
    kind: OperatorGroup
    metadata:
      name: tools
    spec:
      targetNamespaces:
        - tools
    ```

    You'll see that there is an OperatorGroup named `tools` defined in this YAML.

    This confirms why we saw two resources created by the `namespace-tools`
    ArgoCD application in the ArgoCD UI: one namespace and one operatorgroup.

    Again, notice the pattern: a single ArgoCD application manages one or more
    Kubernetes resources in the cluster -- using one or more YAML files in which
    those resources are defined.

  13. *The* `bootstrap-single-cluster` *ArgoCD application in more detail*

    In the ArgoCD UI **Applications** view, click on the `bootstrap-single-cluster` application:

    ![argocd63](/snippets/images-gitops-argocd-connect/argo63-cp4d.png){: style="max-height:400px"}

    You can see the bootstrap application creates two Kubernetes resources, the
    `infra` ArgoCD application and the `infra` ArgoCD project.

    An ArgoCD **project** is a mechanism by which we can group related resources;
    we keep all our ArgoCD applications that manage infrastructure in the `infra`
    project. Later, we'll create a `services` project for the ArgoCD applications
    that manage the services we want deployed to the cluster such as the Cloud Pak 
    operators and services.

  14. *The* `infra` *ArgoCD application*

    Let's examine the `infra` ArgoCD application in more detail to see how it
    works.

    In the ArgoCD UI **Applications** view, click on the `open application` icon
    for the`infra` application:

    ![argocd64](/snippets/images-gitops-argocd-connect/argo64-cp4d.png){: style="max-height:400px"}

    We can see that the `infra` ArgoCD application creates four ArgoCD applications,
    each of which is responsible for applying specific YAMLs to the cluster
    according to the folder the ArgoCD application is watching.

    It's the `infra` ArgoCD application that watches the
    `0-bootstrap/single-cluster/1-infra` folder for ArgoCD applications
    that apply infrastructure resources to our cluster. It was the `infra`
    application that created the `namespace-tools` ArgoCD application which manages
    the `tools` namespace that we've been exploring in this section of the tutorial.

    We'll continually reinforce these relationships as we work through the
    tutorial. You might like to spend some time exploring the ArgoCD UI and ArgoCD
    YAMLs before you proceed, though it's not necessary, as you'll get lots of
    practice as we proceed.