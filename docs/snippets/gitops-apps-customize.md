##  Tailor the -apps repository

As we've just seen, before the cloned `multi-tenancy-gitops-apps` can be used to
store application component YAML files, it must be customized.

Once we've customized the local repository, we'll push our updates back to our
repository branch on GitHub where it can be accessed by ArgoCD running in our
cluster.

1. *Locate the multi-tenancy-gitops-apps repository*

    !!! tip
        Ensure you're in the `multi-tenancy-gitops-apps`  **terminal
        window** before you start to customize the `-apps` repository.

    Issue the following command to change to your GitOps repository:

    ```bash
    cd $GIT_ROOT
    cd multi-tenancy-gitops-apps
    ```

    <br>

2. *Run the customization script*

    Let's customize the cloned `multi-tenancy-gitops-apps` repository with the
    relevant `$GIT_BRANCH` and `$GIT_ORG` values for our cluster.

    The sample GitOps repository provides the `set-git-source.sh` script to make
    this task easy.

    Run script to replace the git url and branch to your git organization where
    you created the git repositories:

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

    You can examine your local clone of the GitOps repository to verify these
    customizations. You should see lots of amended ArgoCD YAMLs that point to your
    GitOps repository on GitHub.

    <br>

3. *Add the changes to a git index, ready to push to GitHub*

    We've now customized our local clone of the `multi-tenancy-gitops-apps` repository. Let's
    commit these changes and make the customized repository available to the
    cluster via GitHub.

    Add all changes in the current folder to a git index:

    ```bash
    git add .
    ```

    <br>

4. *Commit the changes to git*

    Use the following command to create a commit record:

    ```bash
    git commit -s -m "GitOps customizations for organization and cluster"
    ```

    See the Git commit message for the customized files:

    ```{ .bash .no-copy }
    [master a900c39] GitOps customizations for organization and cluster
    36 files changed, 49 insertions(+), 49 deletions(-)
    ```

    <br>

5. *Push changes to GitHub*

    Push this commit back to the  branch on GitHub:

    ```bash
    git push origin $GIT_BRANCH
    ```

    The changes have now been pushed to your GitOps repository:

    ```{ .text .no-copy }
    Enumerating objects: 139, done.
    Counting objects: 100% (139/139), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (76/76), done.
    Writing objects: 100% (87/87), 4.70 KiB | 1.57 MiB/s, done.
    Total 87 (delta 40), reused 0 (delta 0)
    remote: Resolving deltas: 100% (40/40), completed with 14 local objects.
    To https://github.com/tutorial-org-123/multi-tenancy-gitops-apps.git
       d95eca5..a900c39  master -> master
    ```

    We've now successfully pushed the customized the `-apps` repository to our
    GitHub organization, ready for use by Tekton and ArgoCD.

   <br>
