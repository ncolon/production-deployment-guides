## Why customize

We're now going to customize two of the sample GitOps repositories to enable
them to be used by ArgoCD to deploy Kubernetes resources to **your** particular
cluster. The process of customization is important to understand; we will use it
throughout the tutorial.

Recall the different roles of the four repositories:

- `multi-tenancy-gitops` is the main GitOps repository. It contains the ArgoCD
YAMLs for the fixed components in the cluster such as a namespace or a Tekton or
SonarQube instance.
- `multi-tenancy-gitops-infra` is a library repository. It contains
infrastructure YAMLs referred to by the ArgoCD YAMLs that manage
**infrastructure** in the cluster, such as a namespace.
- `multi-tenancy-gitops-services` is a library repository. It contains the
service YAMLs referred to the ArgoCD YAMLs that manage **services** in the
cluster such as Tekton or SonarQube.
- `multi-tenancy-gitops-apps` is a GitOps repository containing the user
application components that are deployed to the cluster. These components
include applications, databases and queue managers, for example.

It is the `multi-tenancy-gitops` and `multi-tenancy-gitops-apps` repositories that
need to be customized for your cluster. The other two repositories don't need to
be customized because they perform the function of a library.

## Tailor `multi-tenancy-gitops`

Let now customize the `multi-tenancy-gitops` repository for your organization.
The repository provides a script that uses the `$GIT_ORG` environment variable
to replace the generic values in the cloned repository with those of your GitHub
organization.

Once customized, we'll push our updated repository back to GitHub where it can
be accessed by ArgoCD.

1. *Set your git branch*

    In this tutorial, we use the `master` branch of the `multi-tenancy-gitops`
    repository. We will store this in the `$GIT_BRANCH` environment variable for
    use by various scripts and commands.

    Issue the following command to set its value:

    ```bash
    export GIT_BRANCH=master
    ```

    You can verify your `$GIT_BRANCH` as follows:

    ```bash
    echo $GIT_BRANCH
    ```

    <br>

2. *Run the customization script*

    We customize the cloned `multi-tenancy-gitops` repository to the relevant
    values for our cluster using the `set-git-source.sh` script. This script
    will replace various YAML elements in this repository to refer to your git
    organization via a GitHub URL.

    Now run the script:

    ```bash
    ./scripts/set-git-source.sh
    ```

    The script will list customizations it will use and all the files that it
    customizes:

    ```{ .text .no-copy }
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-infra.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-services.git on branch master
    Setting kustomization patches to https://github.com/prod-ref-guide/multi-tenancy-gitops-apps.git on branch master
    done replacing variables in kustomization.yaml files
    git commit and push changes now
    ```

    <br>

3. *Explore the customization changes*

    You can easily identify all the files that have been customized using the
    `git status` command.

    Issue the following command:

    ```bash
    git status
    ```

    to view the complete set of changed files in the `multi-tenancy-gitops` repository:

    ```{ .text .no-copy }
    On branch master
    Your branch is up to date with 'origin/master'.

    Changes not staged for commit:
      (use "git add <file>..." to update what will be committed)
      (use "git restore <file>..." to discard changes in working directory)
            modified:   0-bootstrap/bootstrap.yaml
            modified:   0-bootstrap/others/1-shared-cluster/bootstrap-cluster-1-cicd-dev-stage-prod.yaml
            modified:   0-bootstrap/others/1-shared-cluster/bootstrap-cluster-n-prod.yaml
            ...
            modified:   0-bootstrap/single-cluster/kustomization.yaml
    ```

    (We've abbreviated the list.)

    You can see the kind of changes that the script has made with `git diff` command:

    ```bash
    git diff 0-bootstrap/bootstrap.yaml
    ```

    which calculates the differences in the file as:

    ```{ .text .no-copy }
    diff --git a/0-bootstrap/bootstrap.yaml b/0-bootstrap/bootstrap.yaml
    index 0754133..7303c53 100644
    --- a/0-bootstrap/bootstrap.yaml
    +++ b/0-bootstrap/bootstrap.yaml
    @@ -10,8 +10,8 @@ spec:
       project: default
       source:
         path: 0-bootstrap/single-cluster
    -    repoURL: ${GIT_BASEURL}/${GIT_ORG}/${GIT_GITOPS}
    -    targetRevision: ${GIT_GITOPS_BRANCH}
    +    repoURL: https://github.com/tutorial-org-123/multi-tenancy-gitops.git
    +    targetRevision: master
       syncPolicy:
         automated:
           prune: true
    ```

    See how the `repoURL` and `targetRevision` YAML elements now refer to **your
    organization** in GitHub rather than a generic name.

    By making YAMLs like this active in your cluster, ArgoCD is able to refer to
    your repository to manage its contents.

    <br>

4. *Add the changes to a git index, ready to push to GitHub*

    Now that we've customized the local clone of the `multi-tenancy-gitops`
    repository, we should commit the changes, and push them back to GitHub where
    they can be accessed by ArgoCD.

    Firstly, we the changes to a git index:

    ```bash
    git add .
    ```

    <br>

5. *Commit the changes to git*

    We then commit these changes to git index:

    ```bash
    git commit -s -m "GitOps customizations for organization and cluster"
    ```

    which will show the commit message:

    ```{ .text .no-copy }
    [master a900c39] GitOps customizations for organization and cluster
     46 files changed, 176 insertions(+), 176 deletions(-)
    ```

    <br>

6. *Push changes to GitHub*

    Finally, we push this commit back to the `master` branch on GitHub:

    ```bash
    git push origin $GIT_BRANCH
    ```

    which shows that the changes have now been pushed to your GitOps repository:

    ```{ .text .no-copy }
    Enumerating objects: 90, done.
    Counting objects: 100% (90/90), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (47/47), done.
    Writing objects: 100% (48/48), 4.70 KiB | 1.57 MiB/s, done.
    Total 48 (delta 32), reused 0 (delta 0)
    remote: Resolving deltas: 100% (32/32), completed with 23 local objects.
    To https://github.com/tutorial-org-123/multi-tenancy-gitops.git
       d95eca5..a900c39  master -> master
    ```

    We've now customized the `multi-tenancy-gitops` repository for your
    organization, and successfully pushed it to GitHub.

  <br>
