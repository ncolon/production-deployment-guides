## Customizing the GitOps repositories

Let's now customize our repository to deploy Kubernetes resources to our cluster.

The cloned GitOps configuration repositories needs to be customized for your
cluster. The ArgoCD `Application` and `AppProject` resources and `kustomization.yaml`
resources contains environment variables which will need to be updated to your
specific GitHub organization and branch.

Have a look at the following `kustomization.yaml` as an example of what will need to
be updated.

```bash
cat 0-bootstrap/single-cluster/1-infra/kustomization.yaml
```

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
      value: ${GIT_BASEURL}/${GIT_ORG}/${GIT_GITOPS_INFRA}
    - op: add
      path: /spec/source/targetRevision
      value: ${GIT_GITOPS_INFRA_BRANCH}
```

Once we've customized the local repository, we'll push our updates back to our
repository branch on GitHub where it can be accessed by ArgoCD.

  1. *Run the customization script*

    Let's customize the cloned `multi-tenancy-gitops` repository with the relevant values for our
    cluster.

    The sample GitOps repository provides the `set-git-source.sh` script to make
    this task easy.

    Run script to replace the git url and branch to your git organization where
    you created the git repositories:

    ```bash
    GIT_ORG=$GIT_ORG GIT_BRANCH=master ./scripts/set-git-source.sh
    ```

    The script will list customizations it will use and all the files that it
    customizes:

    ```{ .text .no-copy }
    Setting kustomization patches to https://github.com/tutorial-org-123/multi-tenancy-gitops.git on branch master
    Setting kustomization patches to https://github.com/tutorial-org-123/multi-tenancy-gitops-infra.git on branch master
    Setting kustomization patches to https://github.com/tutorial-org-123/multi-tenancy-gitops-services.git on branch master
    Setting kustomization patches to https://github.com/tutorial-org-123/multi-tenancy-gitops-apps.git on branch master
    done replacing variables in kustomization.yaml files
    git commit and push changes now
    ```

    You can examine your local clone of the GitOps repository to verify these
    customizations. You should see lots of amended ArgoCD YAMLs that point to your
    GitOps repository on GitHub.

    This script has also made some customizations that we'll use much later in the
    tutorial; we'll refer to those at the relevant time.

  2. *Add the changes to a git index, ready to push to GitHub*

    We've now customized our local clone of the `multi-tenancy-gitops` repository. Let's
    commit these changes and make the customized repository available to the
    cluster via GitHub.

    Add all changes in the current folder to a git index:

    ```bash
    git add .
    ```

  3. *Commit the changes to git*

    Use the following command to create a commit record:

    ```bash
    git commit -s -m "GitOps customizations for organization and cluster"
    ```

    See the Git commit message for the customized files:

    ```{ .text .no-copy }
    [master a900c39] GitOps customizations for organization and cluster
     46 files changed, 176 insertions(+), 176 deletions(-)
    ```

  4. *Set your GitOps repository branch*

    For this guide, we will be using the `master` branch for the `multi-tenancy-gitops` repository.

    ```bash
    export GIT_BRANCH=master
    ```

    You can verify your `$GIT_BRANCH` as follows:

    ```bash
    echo $GIT_BRANCH
    ```

  5. *Push changes to GitHub*

    Push this commit back to the  branch on GitHub:

    ```bash
    git push origin $GIT_BRANCH
    ```

    The changes have now been pushed to your GitOps repository:

    ```{ .text .no-copy }
    Enumerating objects: 51, done.
    Counting objects: 100% (51/51), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (30/30), done.
    Writing objects: 100% (30/30), 3.92 KiB | 174.00 KiB/s, done.
    Total 30 (delta 21), reused 0 (delta 0), pack-reused 0
    remote: Resolving deltas: 100% (21/21), completed with 10 local objects.
    To https://github.com/tutorial-org-123/multi-tenancy-gitops.git
       d95eca5..a900c39  master -> master
    ```
