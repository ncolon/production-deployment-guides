# Automate Promotion Pipeline Workflow

<!--- cSpell:ignore  pipelinerun mqsc msqc runmqsc dockerconfigjson queuemanager QMID podman ibmgaragecloud cntk cnkt eventid gitrevision gitrepositoryurl odowdaibm MQSC replicaset eventlistener triggerbinding triggertemplate triggertemplates mqcicd qmgr Artifactory configmaps OIDC CHLAUTH templating sealedsecret PostSync post-sync favourite eventlisteners -->

## Overview

In the previous topics, we promoted the queue manager and sample application
from [dev to staging](../cloud-native/promoting-environments-dev-stage.md)
environment and then from [staging to prod ](../cloud-native/promoting-environments-stage-prod.md)
environment. We then deployed them using ArgoCD.

In this topic, we're going to make this promotion processes fully automatic.
As we perviously saw, a change to the queue manager source repository or sample
application source repository will automatically result in a pipeline run for it. If the pipeline is successful, new kubernetes resources will be built and automatically deployed by ArgoCD into the `dev` namespace. Once the Queuemanager or the sample application is successfully deployed in dev namespace, the argocd post sync trigger in dev namespace will automatically generate a pipelinerun. If the pipeline is successful, new resources will be built and automatically deployed by ArgoCD into the `stage` namespace. Following the same approach again, once the Queuemanager or the sample application is successfully deployed in stage namespace, the argocd post sync trigger in stage namespace will automatically generate a pipelinerun. If the pipeline is successful, new resources will be built and automatically deployed by ArgoCD into the `prod` namespace. To recap, the whole process will be automatic from end-to-end.

Look at the following diagram:

![mqcicd7](./images/mq-cicd7.drawio.png){: style="max-height:500px"}

We've already seen most of the highlighted components. Notice that we've added
a `post sync trigger`. This trigger will be responsible to fire promotion pipelines every
time the `QM1` or `Sample application` is successfully deployed in the respective namespaces.

In this topic, we're going to:

* Set up post sync triggers.
* Test the end to end promotion process.

By the end of this topic we'll have a fully automated promotion process set up in place to
deploy the latest changes made to QueueManager or Sample application to the cluster across all environments in an automated fashion.

---

## Pre-requisites

Before attempting this topic, you should have successfully completed the
[previous topic](../cloud-native/promoting-environments-stage-prod.md).

---

## Set up the ArgoCD Post-sync hook

An ArgoCD PostSync hook is useful to run certain checks after the deployment. For instance, if we want to validate the health or perform some integration tests, we can use PostSync hook to do these things for us. Basically, this hook executes after all the the Sync hooks of the application are successful and completed.

If you want to know more about this, check [Resource Hooks](https://argo-cd.readthedocs.io/en/stable/user-guide/resource_hooks/) out.

In this topic section we're going to set up the components we need for our PostSync hook:

![mqcicd8](./images/mq-cicd8.drawio.png)

A PostSync hook will be triggered by ArgoCD once the deployment is successful
and is sent to an **event listener**. In Kubernetes, this event listener
comprises a normal `route` and `service` front-ending a Tekton
`eventlistener` custom resource. The event listener is driven by the ArgoCD
PostSync hook, and runs the pipeline using a Tekton **trigger**.

### Set up trigger templates

1. *Select the GitOps terminal window*

      **Return to the terminal window** you're using for the `multi-tenancy-gitops-apps`
      GitOps repository.

      We're going to make use of a script in the sample GitOps apps repository that makes
      use of the `$GIT_ORG` and `$GIT_BRANCH_QM1` environment variables you've seen
      earlier. Previously we used them in the `mq-infra` terminal window; now we're
      going to use this in the `multi-tenancy-gitops-apps` terminal window.

      Again, **make sure** in you're in the  `multi-tenancy-gitops-apps` terminal window.

      Issue the following command to ensure you're in the correct starting folder:

      ```bash
      cd $GIT_ROOT/multi-tenancy-gitops-apps
      ```

2. *Set up the* `$GIT_ORG` and `$GIT_BRANCH` *environment variables*

      Let's set up the environment variables.

      Replace `<git-org>` in the following command with your GitHub user name:

      ```bash
      export GIT_ORG=<git-org>
      ```

      Then create the `$GIT_BRANCH` environment variable:

      ```bash
      export GIT_BRANCH=master
      ```

      Verify that your `$GIT_USER` and `$GIT_BRANCH` environment variables are set:

      ```bash
      echo $GIT_ORG
      echo $GIT_BRANCH
      ```

      For example:

      ``` { .text .no-copy }
      (base) user/git/multi-tenancy-gitops echo GIT_ORG
      prod-ref-guide
      (base) user/git/multi-tenancy-gitops echo GIT_BRANCH
      master
      ```

<!-- 3. *The trigger templates for Queuemanager PostSync hook in `dev` environment*

      Issue the following commands to generate a trigger template for Queuemanager PostSync hook in `dev` environment:

      ```bash
      cd mq/environments/ci/triggertemplates
      ./mq-infra-dev-triggertemplate.sh
      cd ../../../../
      ```

      Issue the following command to verify:

      ```bash
      cat mq/environments/ci/triggertemplates/mq-infra-dev-triggertemplate.yaml
      ```

      to show the trigger template we created earlier:

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: TriggerTemplate
      metadata:
        labels:
          app: mq-infra-post-dev
        name: mq-infra-post-dev
      spec:
        resourcetemplates:
          - apiVersion: tekton.dev/v1beta1
            kind: PipelineRun
            metadata:
              annotations:
                argocd.argoproj.io/compare-options: IgnoreExtraneous
                argocd.argoproj.io/sync-options: Prune=false
              labels:
                tekton.dev/pipeline: ibm-mq-promote-dev-stage
              generateName: ibm-mq-promote-dev-stage-
            spec:
              params:
              - name: git-url
                value: https://github.com/prod-ref-guide/mq-infra.git
              pipelineRef:
                name: ibm-mq-promote-dev-stage
      ```

4. *The trigger templates for MQ sample application PostSync hook in `dev` environment*

      Issue the following commands to generate a trigger template for MQ Sample application PostSync hook in `dev` environment:

      ```bash
      cd mq/environments/ci/triggertemplates
      ./mq-spring-app-dev-triggertemplate.sh
      cd ../../../../
      ```

      Issue the following command to verify:

      ```bash
      cat mq/environments/ci/triggertemplates/mq-spring-app-dev-triggertemplate.yaml
      ```

      to show the trigger template we created earlier:

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: TriggerTemplate
      metadata:
        labels:
          app: mq-spring-app-post-dev
        name: mq-spring-app-post-dev
      spec:
        resourcetemplates:
          - apiVersion: tekton.dev/v1beta1
            kind: PipelineRun
            metadata:
              annotations:
                argocd.argoproj.io/compare-options: IgnoreExtraneous
                argocd.argoproj.io/sync-options: Prune=false
              labels:
                tekton.dev/pipeline: ibm-mq-promote-dev-stage
              generateName: ibm-mq-promote-dev-stage-
            spec:
              params:
              - name: git-url
                value: https://github.com/prod-ref-guide/mq-spring-app.git
              pipelineRef:
                name: ibm-mq-promote-dev-stage
      ``` -->

5. *The trigger templates for Queuemanager PostSync hook in `stage` environment*

      Issue the following commands to generate a trigger template for Queuemanager PostSync hook in `stage` environment:

      ```bash
      cd mq/environments/ci/triggertemplates
      ./mq-infra-stage-triggertemplate.sh
      cd ../../../../
      ```

      Issue the following command to verify:

      ```bash
      cat mq/environments/ci/triggertemplates/mq-infra-stage-triggertemplate.yaml
      ```

      to show the trigger template we created earlier:

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: TriggerTemplate
      metadata:
        labels:
          app: mq-infra-post-stage
        name: mq-infra-post-stage
      spec:
        resourcetemplates:
          - apiVersion: tekton.dev/v1beta1
            kind: PipelineRun
            metadata:
              annotations:
                argocd.argoproj.io/compare-options: IgnoreExtraneous
                argocd.argoproj.io/sync-options: Prune=false
              labels:
                tekton.dev/pipeline: ibm-mq-promote-stage-prod
              generateName: ibm-mq-promote-stage-prod-
            spec:
              params:
              - name: git-url
                value: https://github.com/prod-ref-guide/mq-infra.git
              pipelineRef:
                name: ibm-mq-promote-stage-prod
      ```

<!-- 6. *The trigger templates for MQ sample application PostSync hook in `stage` environment*

      Issue the following commands to generate a trigger template for MQ Sample application PostSync hook in `stage` environment:

      ```bash
      cd mq/environments/ci/triggertemplates
      ./mq-spring-app-stage-triggertemplate.sh
      cd ../../../../
      ```

      Issue the following command to verify:

      ```bash
      cat mq/environments/ci/triggertemplates/mq-spring-app-stage-triggertemplate.yaml
      ```

      to show the trigger template we created earlier:

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: TriggerTemplate
      metadata:
        labels:
          app: mq-spring-app-post-stage
        name: mq-spring-app-post-stage
      spec:
        resourcetemplates:
          - apiVersion: tekton.dev/v1beta1
            kind: PipelineRun
            metadata:
              annotations:
                argocd.argoproj.io/compare-options: IgnoreExtraneous
                argocd.argoproj.io/sync-options: Prune=false
              labels:
                tekton.dev/pipeline: ibm-mq-promote-stage-prod
              generateName: ibm-mq-promote-stage-prod-
            spec:
              params:
              - name: git-url
                value: https://github.com/prod-ref-guide/mq-spring-app.git
              pipelineRef:
                name: ibm-mq-promote-stage-prod -->
      ```

7. *Push GitOps changes to GitHub*

      Let’s make these GitOps changes and push them.

      Add all changes in the current folder to a git index, commit them, and push them to GitHub:

      ```bash
      git add mq/environments/ci/triggertemplates
      git commit -s -m "Defining trigger templates for promotion automation"
      git push origin $GIT_BRANCH
      ```

### Review the eventlisteners

1. *The eventlistener for QueueManager in `dev` environment*

      Issue the following commands to verify the eventlistener definition in `dev` environment for the QueueManager:

      ```bash
      cat mq/environments/ci/eventlisteners/mq-infra-dev-eventlistener.yaml
      ```

      to show the eventlistener:

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: EventListener
      metadata:
        name: mq-infra-post-dev
      spec:
        serviceAccountName: pipeline
        triggers:
          - name: mq-infra-post-dev
            template:
              ref: mq-infra-post-dev
      ```

2. *The eventlistener for MQ sample application in `dev` environment*

      Issue the following commands to verify the eventlistener definition in `dev` environment for the MQ sample application:

      ```bash
      cat mq/environments/ci/eventlisteners/mq-spring-app-dev-eventlistener.yaml
      ```

      to show the eventlistener:

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: EventListener
      metadata:
        name: mq-spring-app-post-dev
      spec:
        serviceAccountName: pipeline
        triggers:
          - name: mq-spring-app-post-dev
            template:
              ref: mq-spring-app-post-dev
      ```

3. *The eventlistener for QueueManager in `stage` environment*

      Issue the following commands to verify the eventlistener definition in `stage` environment for the QueueManager:

      ```bash
      cat mq/environments/ci/eventlisteners/mq-infra-stage-eventlistener.yaml
      ```

      to show the eventlistener:

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: EventListener
      metadata:
        name: mq-infra-post-stage
      spec:
        serviceAccountName: pipeline
        triggers:
          - name: mq-infra-post-stage
            template:
              ref: mq-infra-post-stage
      ```

4. *The eventlistener for MQ sample application in `stage` environment*

      Issue the following commands to verify the eventlistener definition in `stage` environment for the MQ sample application:

      ```bash
      cat mq/environments/ci/eventlisteners/mq-spring-app-stage-eventlistener.yaml
      ```

      to show the eventlistener:

      ``` { .yaml .no-copy }
      apiVersion: triggers.tekton.dev/v1alpha1
      kind: EventListener
      metadata:
        name: mq-spring-app-post-stage
      spec:
        serviceAccountName: pipeline
        triggers:
          - name: mq-spring-app-post-stage
            template:
              ref: mq-spring-app-post-stage
      ```

### Review the routes for the eventlisteners

1. *The route for QueueManager eventlistener in `dev` environment*

      Issue the following commands to verify the eventlistener route definition in `dev` environment for the QueueManager:

      ```bash
      cat mq/environments/ci/routes/mq-infra-dev-route.yaml
      ```

      to show the route we created earlier:

      ``` { .yaml .no-copy }
      apiVersion: route.openshift.io/v1
      kind: Route
      metadata:
        name: mq-infra-post-dev
        labels:
          app.kubernetes.io/managed-by: EventListener
          app.kubernetes.io/part-of: Triggers
          eventlistener: mq-infra-post-dev
        annotations:
          openshift.io/host.generated: 'true'
      spec:
        to:
          kind: "Service"
          name: el-mq-infra-post-dev
          weight: 100
        port:
          targetPort: http-listener
        wildcardPolicy: None
      ```

2. *The route for MQ sample application eventlistener in `dev` environment*

      Issue the following commands to verify the eventlistener route definition in `dev` environment for the MQ sample application:

      ```bash
      cat mq/environments/ci/routes/mq-spring-app-dev-route.yaml
      ```

      to show the eventlistener we created earlier:

      ``` { .yaml .no-copy }
      apiVersion: route.openshift.io/v1
      kind: Route
      metadata:
        name: mq-spring-app-post-dev
        labels:
          app.kubernetes.io/managed-by: EventListener
          app.kubernetes.io/part-of: Triggers
          eventlistener: mq-spring-app-post-dev
        annotations:
          openshift.io/host.generated: 'true'
      spec:
        to:
          kind: "Service"
          name: el-mq-spring-app-post-dev
          weight: 100
        port:
          targetPort: http-listener
        wildcardPolicy: None
      ```

3. *The route for QueueManager eventlistener in `stage` environment*

      Issue the following commands to verify the eventlistener route definition in `stage` environment for the QueueManager:

      ```bash
      cat mq/environments/ci/routes/mq-infra-stage-route.yaml
      ```

      to show the route we created earlier:

      ``` { .yaml .no-copy }
      apiVersion: route.openshift.io/v1
      kind: Route
      metadata:
        name: mq-infra-post-stage
        labels:
          app.kubernetes.io/managed-by: EventListener
          app.kubernetes.io/part-of: Triggers
          eventlistener: mq-infra-post-stage
        annotations:
          openshift.io/host.generated: 'true'
      spec:
        to:
          kind: "Service"
          name: el-mq-infra-post-stage
          weight: 100
        port:
          targetPort: http-listener
        wildcardPolicy: None
      ```

4. *The route for MQ sample application eventlistener in `stage` environment*

      Issue the following commands to verify the eventlistener route definition in `stage` environment for the MQ sample application:

      ```bash
      cat mq/environments/ci/routes/mq-spring-app-stage-route.yaml
      ```

      to show the eventlistener we created earlier:

      ``` { .yaml .no-copy }
      apiVersion: route.openshift.io/v1
      kind: Route
      metadata:
        name: mq-spring-app-post-stage
        labels:
          app.kubernetes.io/managed-by: EventListener
          app.kubernetes.io/part-of: Triggers
          eventlistener: mq-spring-app-post-stage
        annotations:
          openshift.io/host.generated: 'true'
      spec:
        to:
          kind: "Service"
          name: el-mq-spring-app-post-stage
          weight: 100
        port:
          targetPort: http-listener
        wildcardPolicy: None
      ```

### Activate the Triggers

1. *Activate the PostSync hook components*

      Now that we've created the event listener and triggers, let's activate all
      the components necessary to process our PostSync hook: the route, event
      listener and trigger template.

      Open the `kustomization.yaml` that is under `mq/environments/ci/`. This
      contains all the resources that belong to the `ci` namespace.

      ```bash
      cat mq/environments/ci/kustomization.yaml
      ```

      ```{ .yaml .no-copy }
      resources:
      #- certificates/ci-mq-client-certificate.yaml
      #- certificates/ci-mq-server-certificate.yaml
      - configmaps/gitops-repo-configmap.yaml
      - eventlisteners/cntk-event-listener.yaml
      - triggerbindings/cntk-binding.yaml
      - triggertemplates/mq-infra-dev.yaml
      - triggertemplates/mq-spring-app-dev.yaml
      - pipelines/ibm-test-pipeline-for-dev.yaml
      - pipelines/ibm-test-pipeline-for-stage.yaml
      #- pipelines/java-maven-dev-pipeline.yaml
      - pipelines/mq-pipeline-dev.yaml
      - pipelines/mq-spring-app-dev-pipeline.yaml
      - roles/custom-pipeline-sa-clusterrole.yaml
      - roles/custom-pipeline-sa-role.yaml
      - roles/custom-ci-pipeline-sa-rolebinding.yaml
      - roles/custom-dev-pipeline-sa-rolebinding.yaml
      - roles/custom-staging-pipeline-sa-rolebinding.yaml
      - roles/custom-prod-pipeline-sa-rolebinding.yaml
      - routes/cntk-route.yaml
      # - secrets/artifactory-access-secret.yaml
      - secrets/git-credentials-secret.yaml
      - secrets/ibm-entitled-registry-credentials-secret.yaml
      #- secrets/mq-client-jks-password-secret.yaml
      - tasks/10-gitops.yaml
      - tasks/10-gitops-for-mq.yaml
      - tasks/10-gitops-promotion.yaml
      - tasks/11-app-name.yaml
      - tasks/12-functional-tests.yaml
      - tasks/13-jmeter-performance-test.yaml
      - tasks/13-cphtestp-performance-test.yaml
      - tasks/4-smoke-tests-mq.yaml
      - tasks/4-smoke-tests.yaml
      - tasks/ibm-build-tag-push-v2-6-13.yaml
      - tasks/ibm-helm-release-v2-6-13.yaml
      - tasks/ibm-img-release-v2-6-13.yaml
      - tasks/ibm-img-scan-v2-6-13.yaml
      - tasks/ibm-java-maven-test-v2-6-13.yaml
      - tasks/ibm-setup-v2-6-13.yaml
      - tasks/ibm-tag-release-v2-6-13.yaml

      - triggertemplates/mq-infra-dev-triggertemplate.yaml
      - eventlisteners/mq-infra-dev-eventlistener.yaml
      - routes/mq-infra-dev-route.yaml

      # Automated promotion process triggers
      # - triggertemplates/mq-spring-app-dev-triggertemplate.yaml
      # - eventlisteners/mq-spring-app-dev-eventlistener.yaml
      # - routes/mq-spring-app-dev-route.yaml
      #
      # - triggertemplates/mq-infra-stage-triggertemplate.yaml
      # - eventlisteners/mq-infra-stage-eventlistener.yaml
      # - routes/mq-infra-stage-route.yaml
      #
      # - triggertemplates/mq-spring-app-stage-triggertemplate.yaml
      # - eventlisteners/mq-spring-app-stage-eventlistener.yaml
      # - routes/mq-spring-app-stage-route.yaml
      ```

      Open this file in your editor and uncomment the below resources.

      ```text
      - triggertemplates/mq-infra-dev-triggertemplate.yaml
      - eventlisteners/mq-infra-dev-eventlistener.yaml
      - routes/mq-infra-dev-route.yaml

      - triggertemplates/mq-spring-app-dev-triggertemplate.yaml
      - eventlisteners/mq-spring-app-dev-eventlistener.yaml
      - routes/mq-spring-app-dev-route.yaml

      - triggertemplates/mq-infra-stage-triggertemplate.yaml
      - eventlisteners/mq-infra-stage-eventlistener.yaml
      - routes/mq-infra-stage-route.yaml

      - triggertemplates/mq-spring-app-stage-triggertemplate.yaml
      - eventlisteners/mq-spring-app-stage-eventlistener.yaml
      - routes/mq-spring-app-stage-route.yaml
      ```

      You will have the following resources now:

      ```{ .yaml .no-copy }
      resources:
      #- certificates/ci-mq-client-certificate.yaml
      #- certificates/ci-mq-server-certificate.yaml
      - configmaps/gitops-repo-configmap.yaml
      - eventlisteners/cntk-event-listener.yaml
      - triggerbindings/cntk-binding.yaml
      - triggertemplates/mq-infra-dev.yaml
      - triggertemplates/mq-spring-app-dev.yaml
      - pipelines/ibm-test-pipeline-for-dev.yaml
      - pipelines/ibm-test-pipeline-for-stage.yaml
      #- pipelines/java-maven-dev-pipeline.yaml
      - pipelines/mq-pipeline-dev.yaml
      - pipelines/mq-spring-app-dev-pipeline.yaml
      - roles/custom-pipeline-sa-clusterrole.yaml
      - roles/custom-pipeline-sa-role.yaml
      - roles/custom-ci-pipeline-sa-rolebinding.yaml
      - roles/custom-dev-pipeline-sa-rolebinding.yaml
      - roles/custom-staging-pipeline-sa-rolebinding.yaml
      - roles/custom-prod-pipeline-sa-rolebinding.yaml
      - routes/cntk-route.yaml
      - secrets/artifactory-access-secret.yaml
      - secrets/git-credentials-secret.yaml
      - secrets/ibm-entitled-registry-credentials-secret.yaml
      #- secrets/mq-client-jks-password-secret.yaml
      - tasks/10-gitops.yaml
      - tasks/12-functional-tests.yaml
      - tasks/13-jmeter-performance-test.yaml
      - tasks/4-smoke-tests-mq.yaml
      - tasks/4-smoke-tests.yaml
      - tasks/ibm-build-tag-push-v2-6-13.yaml
      - tasks/ibm-helm-release-v2-6-13.yaml
      - tasks/ibm-img-release-v2-6-13.yaml
      - tasks/ibm-img-scan-v2-6-13.yaml
      - tasks/ibm-java-maven-test-v2-6-13.yaml
      - tasks/ibm-setup-v2-6-13.yaml
      - tasks/ibm-tag-release-v2-6-13.yaml

      # Automated promotion process triggers

      - triggertemplates/mq-infra-dev-triggertemplate.yaml
      - eventlisteners/mq-infra-dev-eventlistener.yaml
      - routes/mq-infra-dev-route.yaml

      - triggertemplates/mq-spring-app-dev-triggertemplate.yaml
      - eventlisteners/mq-spring-app-dev-eventlistener.yaml
      - routes/mq-spring-app-dev-route.yaml

      - triggertemplates/mq-infra-stage-triggertemplate.yaml
      - eventlisteners/mq-infra-stage-eventlistener.yaml
      - routes/mq-infra-stage-route.yaml

      - triggertemplates/mq-spring-app-stage-triggertemplate.yaml
      - eventlisteners/mq-spring-app-stage-eventlistener.yaml
      - routes/mq-spring-app-stage-route.yaml
      ```

2. *Update the GitOps repository*

      Let’s commit these changes to make the event listener and trigger resources
      active in the cluster.

      Issue the following command:

      ```bash
      git add mq/environments/ci/kustomization.yaml
      git commit -s -m "Add PostSync hook components"
      git push origin $GIT_BRANCH
      ```

      which shows that the changes are pushed to GitHub:

      ``` { .text .no-copy }
      Enumerating objects: 20, done.
      Counting objects: 100% (18/18), done.
      Delta compression using up to 16 threads
      Compressing objects: 100% (10/10), done.
      Writing objects: 100% (10/10), 928 bytes | 928.00 KiB/s, done.
      Total 10 (delta 8), reused 0 (delta 0), pack-reused 0
      remote: Resolving deltas: 100% (8/8), completed with 5 local objects.
      To https://github.com/prod-ref-guide/multi-tenancy-gitops-apps.git
        239e49d..3a32a17  master -> master      
      ```

      This change to the GitOps repository can now be used by ArgoCD.

### Enable PostSync hooks in dev

1. *Add PostSync hook to `mq-infra`*

      Issue the following commands to enable the postsync hook for `mq-infra`:

      ```bash
      cd mq/environments/mq-infra/overlays/dev/mq-infra/hooks/
      ./post-sync-job.sh
      cd ../../../../../../../
      ```

      Issue the following command to verify:

      ```bash
      cat mq/environments/mq-infra/overlays/dev/mq-infra/hooks/post-sync-job.yaml
      ```

      to show the `post-sync-job.yaml` we created earlier:

      ```{ .yaml .no-copy }
      apiVersion: batch/v1
      kind: Job
      metadata:
        name: trigger-dev-stage-promotion-pipeline-for-qmgr
        namespace: ci
        annotations:
          argocd.argoproj.io/hook: PostSync
      spec:
        template:
          spec:
            serviceAccountName: pipeline
            containers:
            - name: tests-in-dev-env
              image: quay.io/rhcanada/tkn-cli
              command:
                - tkn
              args:
                - -n
                - ci
                - pipeline
                - start
                - ibm-mq-promote-dev-stage
                - --param
                - git-url=https://github.com/prod-ref-guide/mq-infra.git
                - --param
                - git-revision=master
                - --param
                - src-environment=dev
                - --param
                - dest-environment=staging
                - --param
                - app-path="mq/environments"
                - --param
                - git-pr=true
                - --param
                - test-file="postman/mq-spring-app.postman_collection.json"
            restartPolicy: Never
        backoffLimit: 0
      ```

2. *Activate the PostSync hook for `mq-infra`*

      Now that we've created the PostSync job, let's activate it.

      Open the `kustomization.yaml` that is under `mq/environments/mq-infra/overlays/dev/`. This contains all the resources that belong to the `dev` namespace.

      ```bash
      cat mq/environments/mq-infra/overlays/dev/kustomization.yaml
      ```

      ```{ .text .no-copy }
      resources:
       - ../../base
       - configmap/configmap.yaml
       # - configmap-security/configmap.yaml
       # - mq-infra/hooks/post-sync-job.yaml
       # - certificates/dev-mq-client-certificate.yaml
       # - certificates/dev-mq-server-certificate.yaml
       # - secrets/mq-client-jks-password-secret.yaml
      patchesStrategicMerge:
       - mq-infra/qm-template.yaml
      ```

      Open this file in your editor and uncomment the below resources.

      ```
      - mq-infra/hooks/post-sync-job.yaml
      ```

      You will have the following resources now:

      ```{ .text .no-copy }
      resources:
       - ../../base
       - configmap/configmap.yaml
       # - configmap-security/configmap.yaml
       - mq-infra/hooks/post-sync-job.yaml
       # - certificates/dev-mq-client-certificate.yaml
       # - certificates/dev-mq-server-certificate.yaml
       # - secrets/mq-client-jks-password-secret.yaml
      patchesStrategicMerge:
       - mq-infra/qm-template.yaml
      ```

3. *Add PostSync hook to `mq-spring-app`*

      Issue the following commands to enable the postsync hook for `mq-spring-app`:

      ```bash
      cd mq/environments/mq-spring-app/overlays/dev/mq-spring-app/hooks/
      ./post-sync-job.sh
      cd ../../../../../../../
      ```


      Issue the following command to verify:

      ```bash
      cat mq/environments/mq-spring-app/overlays/dev/mq-spring-app/hooks/post-sync-job.yaml
      ```

      to show the `post-sync-job.yaml` we created earlier:

      ```{ .yaml .no-copy }
      apiVersion: batch/v1
      kind: Job
      metadata:
        name: trigger-dev-stage-promotion-pipeline-for-app
        namespace: ci
        annotations:
          argocd.argoproj.io/hook: PostSync
      spec:
        template:
          spec:
            serviceAccountName: pipeline
            containers:
            - name: tests-in-dev-env
              image: quay.io/rhcanada/tkn-cli
              command:
                - tkn
              args:
                - -n
                - ci
                - pipeline
                - start
                - ibm-mq-promote-dev-stage
                - --param
                - git-url=https://github.com/prod-ref-guide/mq-spring-app.git
                - --param
                - git-revision=master
                - --param
                - src-environment=dev
                - --param
                - dest-environment=staging
                - --param
                - app-path="mq/environments"
                - --param
                - git-pr=true
                - --param
                - test-file="postman/mq-spring-app.postman_collection.json"
            restartPolicy: Never
        backoffLimit: 0
      ```

4. *Activate the PostSync hook for `mq-spring-app`*

      Now that we've created the PostSync job, let's activate it.

      Open the `kustomization.yaml` that is under `mq/environments/mq-spring-app/overlays/dev/`. This contains all the resources that belong to the `dev` namespace.

      ```bash
      cat mq/environments/mq-spring-app/overlays/dev/kustomization.yaml
      ```

      ```{ .text .no-copy }
      resources:
       - ../../base
       - configmap/configmap.yaml
       - configmap/configmap-ccdt.yaml
       # - mq-spring-app/hooks/post-sync-job.yaml
       # - certificates/dev-mq-client-certificate.yaml
       # - certificates/dev-mq-server-certificate.yaml
       # - secrets/mq-client-jks-password-secret.yaml
      patchesStrategicMerge:
       - mq-spring-app/deployment.yaml
      ```

      Open this file in your editor and uncomment the below resources.

      ```
      - mq-spring-app/hooks/post-sync-job.yaml
      ```

      You will have the following resources now:

      ```{ .text .no-copy }
      resources:
       - ../../base
       - configmap/configmap.yaml
       - configmap/configmap-ccdt.yaml
       - mq-spring-app/hooks/post-sync-job.yaml
       # - certificates/dev-mq-client-certificate.yaml
       # - certificates/dev-mq-server-certificate.yaml
       # - secrets/mq-client-jks-password-secret.yaml
      patchesStrategicMerge:
       - mq-spring-app/deployment.yaml
      ```

5. *Push GitOps changes to GitHub*

      Add all changes in the current folder to a git index, commit them, and push them to GitHub:

      ```bash
      git add .
      git commit -s -m "Defining PostSync hooks to mq-infra and mq-spring-app for promotion automation from dev to stage"
      git push origin $GIT_BRANCH
      ```

### Enable PostSync hooks in stage

1. *Add PostSync hook to `mq-infra`*

      Issue the following commands to enable the postsync hook for `mq-infra`:

      ```bash
      cd mq/environments/mq-infra/overlays/staging/mq-infra/hooks/
      ./post-sync-job.sh
      cd ../../../../../../../
      ```

      Issue the following command to verify:

      ```bash
      cat mq/environments/mq-infra/overlays/staging/mq-infra/hooks/post-sync-job.yaml
      ```

      to show the `post-sync-job.yaml` we created earlier:

      ```{ .yaml .no-copy }
      apiVersion: batch/v1
      kind: Job
      metadata:
        name: trigger-dev-stage-promotion-pipeline-for-qmgr
        namespace: ci
        annotations:
          argocd.argoproj.io/hook: PostSync
      spec:
        template:
          spec:
            serviceAccountName: pipeline
            containers:
            - name: tests-in-dev-env
              image: quay.io/rhcanada/tkn-cli
              command:
                - tkn
              args:
                - -n
                - ci
                - pipeline
                - start
                - ibm-mq-promote-stage-prod
                - --param
                - git-url=https://github.com/prod-ref-guide/mq-infra.git
                - --param
                - git-revision=master
                - --param
                - src-environment=staging
                - --param
                - dest-environment=prod
                - --param
                - app-path="mq/environments"
                - --param
                - qm-security=false
                - --param
                - git-pr=true
                - --param
                - test-plan="jmeter/mq-spring-app.jmx"
            restartPolicy: Never
        backoffLimit: 0
      ```

2. *Activate the PostSync hook for `mq-infra`*

      Now that we've created the PostSync job, let's activate it.

      Open the `kustomization.yaml` that is under `mq/environments/mq-infra/overlays/staging/`. This contains all the resources that belong to the `stage` namespace.

      ```bash
      cat mq/environments/mq-infra/overlays/staging/kustomization.yaml
      ```

      ```{ .text .no-copy }
      resources:
       - ../../base
       - configmap/configmap.yaml
       # - configmap-security/configmap.yaml
       # - mq-infra/hooks/post-sync-job.yaml
       #- certificates/staging-mq-client-certificate.yaml
       #- certificates/staging-mq-server-certificate.yaml
       #- secrets/mq-client-jks-password-secret.yaml
      patchesStrategicMerge:
       - mq-infra/qm-template.yaml
      ```

      Open this file in your editor and uncomment the below resources.

      ```
      - mq-infra/hooks/post-sync-job.yaml
      ```

      You will have the following resources now:

      ```{ .text .no-copy }
      resources:
       - ../../base
       - configmap/configmap.yaml
       # - configmap-security/configmap.yaml
       - mq-infra/hooks/post-sync-job.yaml
       #- certificates/staging-mq-client-certificate.yaml
       #- certificates/staging-mq-server-certificate.yaml
       #- secrets/mq-client-jks-password-secret.yaml
      patchesStrategicMerge:
       - mq-infra/qm-template.yaml
      ```

3. *Add PostSync hook to `mq-spring-app`*

      Issue the following commands to enable the postsync hook for `mq-spring-app`:

      ```bash
      cd mq/environments/mq-spring-app/overlays/staging/mq-spring-app/hooks/
      ./post-sync-job.sh
      cd ../../../../../../../
      ```

      Issue the following command to verify:

      ```bash
      cat mq/environments/mq-spring-app/overlays/staging/mq-spring-app/hooks/post-sync-job.yaml
      ```

      to show the `post-sync-job.yaml` we created earlier:

      ```{ .yaml .no-copy }
      apiVersion: batch/v1
      kind: Job
      metadata:
        name: trigger-dev-stage-promotion-pipeline-for-app
        namespace: ci
        annotations:
          argocd.argoproj.io/hook: PostSync
      spec:
        template:
          spec:
            serviceAccountName: pipeline
            containers:
            - name: tests-in-dev-env
              image: quay.io/rhcanada/tkn-cli
              command:
                - tkn
              args:
                - -n
                - ci
                - pipeline
                - start
                - ibm-mq-promote-stage-prod
                - --param
                - git-url=https://github.com/mq-in-depth-kustomize/mq-spring-app.git
                - --param
                - git-revision=master
                - --param
                - src-environment=staging
                - --param
                - dest-environment=prod
                - --param
                - app-path="mq/environments"
                - --param
                - qm-security=false
                - --param
                - git-pr=true
                - --param
                - test-plan="jmeter/mq-spring-app.jmx"
            restartPolicy: Never
        backoffLimit: 0
      ```

4. *Activate the PostSync hook for `mq-spring-app`*

      Now that we've created the PostSync job, let's activate it.

      Open the `kustomization.yaml` that is under `mq/environments/mq-spring-app/overlays/staging/`. This contains all the resources that belong to the `stage` namespace.

      ```bash
      cat mq/environments/mq-spring-app/overlays/staging/kustomization.yaml
      ```

      ```{ .text .no-copy }
      resources:
       - ../../base
       - configmap/configmap.yaml
       - configmap/configmap-ccdt.yaml
       # - mq-spring-app/hooks/post-sync-job.yaml
       #- certificates/staging-mq-client-certificate.yaml
       #- certificates/staging-mq-server-certificate.yaml
       #- secrets/mq-client-jks-password-secret.yaml
      patchesStrategicMerge:
       - mq-spring-app/deployment.yaml
      ```

      Open this file in your editor and uncomment the below resources.

      ```
      - mq-spring-app/hooks/post-sync-job.yaml
      ```

      You will have the following resources now:

      ```{ .text .no-copy }
      resources:
       - ../../base
       - configmap/configmap.yaml
       - configmap/configmap-ccdt.yaml
       - mq-spring-app/hooks/post-sync-job.yaml
       #- certificates/staging-mq-client-certificate.yaml
       #- certificates/staging-mq-server-certificate.yaml
       #- secrets/mq-client-jks-password-secret.yaml
      patchesStrategicMerge:
       - mq-spring-app/deployment.yaml
      ```


5. *Push GitOps changes to GitHub*

      Add all changes in the current folder to a git index, commit them, and push them to GitHub:

      ```bash
      git add .
      git commit -s -m "Defining PostSync hooks to mq-infra and mq-spring-app for promotion automation from stage to prod"
      git push origin $GIT_BRANCH
      ```

---

!!! success "Congratulations!"
    You've now got a fully automated promotion process for your queue
    manager and MQ sample application.

    You started the chapter by configuring all the necessary components for a ArgoCD
    PostSync resource hook. As part of it, you created event listeners and triggers in your cluster to process these events and start the promotion pipeline run using the data in the event. After updating the `QM1` source
    repository or MQ Sample Application source repository, respective pipeline runs was
    triggered automatically.

    Initially when the changes are made, `mq-infra-dev` or `mq-spring-app-dev` pipeline will run. Once those are deployed in the `dev` namespace, ArgoCD PostSync hook in `dev` will trigger the `ibm-mq-promote-dev-stage` pipeline automatically which leaves a PR in the GitOps apps repository for `stage` resources. Once this PR is merged and the deployments are done in `stage` namespace, ArgoCD PostSync hook in `stage` will trigger the `ibm-mq-promote-stage-prod` pipeline automatically which leaves a PR in the GitOps apps repository for `prod` resources. Once this PR is merged, deployments will be done in `prod` namespace.
