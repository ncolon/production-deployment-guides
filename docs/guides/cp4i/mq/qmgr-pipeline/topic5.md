# Enabling dynamic configurations in Queue Manager (Optional)

<!--- cSpell:ignore  pipelinerun resyncs mqsc msqc runmqsc dockerconfigjson queuemanager QMID podman ibmgaragecloud cntk cnkt eventid gitrevision gitrepositoryurl odowdaibm MQSC replicaset eventlistener triggerbinding triggertemplate mqcicd qmgr Artifactory configmaps OIDC CHLAUTH templating sealedsecret -->

**Audience**: Architects, Application developers, Administrators

**Timing**: 40 minutes

## Overview

In the [previous topic](topic1.md) of this chapter, we installed and customized
the queue manager repository for `QM1` and configured the queue manager pipeline
to access it.

In this chapter, we're going to enable dynamic configurations in the queue manager.
This will require us to install some additional components into our cluster. Also, this chapter is an optional one and is not mandate. Please only follow these instructions if you are interested in setting up dynamic configurations to the queue manager.

In this topic, we're going to:

* Review the sample queue manager configurations
* Enable the dynamic configurations
* Set up the GitOps environment

By the end of this topic we'll have the set up done for the queue manager to support dynamic configurations and the queue manager will be able to consume all the configuration changes with out any down time.

---

## Pre-requisites

Before attempting this topic, you should have successfully completed the
[previous topic](../qmgr-pipeline/topic1.md).

---

## Review the sample queue manager configurations

1. *Make sure to be in MQ QM01 repository*

      Let's ensure we're in the correct folder.  Again, we've used typical defaults:

      ```bash
      cd $GIT_ROOT
      cd mq-qm01/
      ```

### Static Configurations

1. *Review the queue manager configurations*

      By default, the sample queue manager uses static configurations.

      We can examine the static configurations using the below command:

      ```bash
      cat kustomize/base/generic-qmgr/static-definitions.mqsc
      ```
      See how the configurations are defined in the mqsc file :

      ``` { .text .no-copy }
      DEFINE QLOCAL(IBM.DEMO.Q) BOQNAME(IBM.DEMO.Q.BOQ) BOTHRESH(3) REPLACE
      DEFINE QLOCAL(IBM.DEMO.Q.BOQ) REPLACE
      * Use a different dead letter queue, for undeliverable messages
      DEFINE QLOCAL('DEV.DEAD.LETTER.QUEUE') REPLACE
      ALTER QMGR DEADQ('DEV.DEAD.LETTER.QUEUE')
      DEFINE CHANNEL('IBM.APP.SVRCONN') CHLTYPE(SVRCONN)
      ALTER QMGR CHLAUTH (DISABLED)
      * DEFINE CHANNEL('MONITORING_CHL') CHLTYPE(SVRCONN)
      * SET CHLAUTH(MONITORING_CHL) TYPE(BLOCKUSER) USERLIST(NOBODY)
      REFRESH SECURITY TYPE(CONNAUTH)
      * optional
      * DEFINE SERVICE(APPLY_MQSC) CONTROL(QMGR) SERVTYPE(SERVER) STARTCMD('/mq-config/start-mqsc.sh') STARTARG(QM1) STOPCMD('/mq-config/start-mqsc.sh') STOPARG('') STDOUT('') STDERR('')
      ```

2. *Review the queue manager configmap generation*

      We are using `configMapGenerator` in Kustomize to build the `configmap` out of
      the `mqsc` definitions provided to the queue manager. Similarly, we can
      also use `ini` files. The `kustomization.yaml` includes those definitions.

      We can examine the `kustomization.yaml` using the below command:

      ```bash
      cat kustomize/base/generic-qmgr/kustomization.yaml
      ```
      See how the `configMapGenerator` definition is included in the `kustomization.yaml` file :

      ``` { .yaml .no-copy }
      resources:
      - ./queuemanager.yaml

      generatorOptions:
       disableNameSuffixHash: true
      # We use a configMapGenerator because it allows us to build up the mqsc from regular MQSC files.
      configMapGenerator:
      # Create an MQSC configMap using generic MQSC which will be added to all queue managers and applied during bootstrap.
      - name: mqsc-configmap
        behavior: create
        files:
        - static-definitions.mqsc

      # Add the configMap that will be used for dynamic updates, this should be used queue manager wide i.e. stay the same in each environment.
      # components:
      # - ../../components/dynamic-mqsc/generic-qmgr
      # - ../../components/scripts
      ```

3. *Review the queue manager configmap consumption*

      As per the above definition, configMapGenerator generates a configmap named
      `mqsc-configmap` and this in turn will be consumed by the queue manager.

      We can examine the `queuemanager.yaml` using the below command:

      ```bash
      cat kustomize/base/generic-qmgr/queuemanager.yaml
      ```
      See how the `configmap` is being consumed by the queue manager :

      ``` { .yaml hl_lines="28 29 30 31 32" .no-copy }
      apiVersion: mq.ibm.com/v1beta1
      kind: QueueManager
      metadata:
        name: qm1
        annotations:
          argocd.argoproj.io/sync-wave: "300"
          helm.sh/hook-weight: "300"
      spec:
        license:
          accept: true

          license: L-RJON-BZFQU2

          use: NonProduction
        queueManager:
          debug: false
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 1
            initialDelaySeconds: 90
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
          logFormat: Basic
          metrics:
            enabled: true
          name: QM1
          mqsc:
            - configMap:
                name: mqsc-configmap
                items:
                  - static-definitions.mqsc
          readinessProbe:
            failureThreshold: 1
            initialDelaySeconds: 10
            periodSeconds: 5
            successThreshold: 1
            timeoutSeconds: 3
          resources:
            limits:
              cpu: "1"
              memory: 1Gi
            requests:
              cpu: "1"
              memory: 1Gi

          availability:
            type: SingleInstance

          image: "replace"
          imagePullPolicy: Always

          storage:
            persistedData:
              enabled: false
            queueManager:
              type: ephemeral
            recoveryLogs:
              enabled: false

        securityContext:
          initVolumeAsRoot: false
        template:
          pod:
            containers:
            - name: qmgr
              env:
              - name: MQSNOAUT
                value: "yes"
        terminationGracePeriodSeconds: 30
        tracing:
          agent: {}
          collector: {}
          enabled: false
          namespace: ""
        version: 9.2.3.0-r1
        web:
          enabled: true
      ```

### Dynamic Configurations

1. *Review the queue manager dynamic configurations*

      By default, the sample queue manager uses static configurations. If we want to change the existing configurations, we can alter the `static-definitions.mqsc` and redeploy the queue manager for the new configurations to come in to place.

      To avoid this downtime, we can optionally enable dynamic configurations. Once we enable this dynamic configurations, any changes in the `dynamic-definitions.mqsc` will get automatically reflected in the queue manager and there is no need of restarting the queue manager.

      To define these dynamic configurations, we are using [Kustomize Components](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/components.md).

      We can examine the dynamic configurations using the below command:

      ```bash
      cat kustomize/components/dynamic-mqsc/generic-qmgr/dynamic-definitions.mqsc
      ```
      See how the configurations are defined in the mqsc file :

      ``` { .text .no-copy }
      * Use this file for MQSC that you want to be able to update without restarting the queue manager.
      DEFINE QLOCAL(TEST.DYNAMIC.QUEUE)
      ```

2. *Review the queue manager configmap generation*

      Kustomize is using `configMapGenerator` to build the `configmap` out of
      the `mqsc` definitions provided to the queue manager. Similarly, we can
      also use `ini` files. The `kustomization.yaml` includes those definitions.

      We can examine the `kustomization.yaml` using the below command:

      ```bash
      cat kustomize/components/dynamic-mqsc/generic-qmgr/kustomization.yaml
      ```
      See how the `configMapGenerator` definition is included in the `kustomization.yaml` file :

      ``` { .yaml .no-copy }
      apiVersion: kustomize.config.k8s.io/v1alpha1
      kind: Component

      generatorOptions:
       disableNameSuffixHash: true

      # Create a configMap that will be used with a volume that is dynamically updated.
      configMapGenerator:
      - name: dynamic-mqsc-configmap
        behavior: create
        files:
        - dynamic-definitions.mqsc

      patchesStrategicMerge:
        - ./queuemanager.yaml
      ```

3. *Review the queue manager configmap consumption*

      As per the above definition, configMapGenerator generates a configmap named
      `dynamic-mqsc-configmap` and this in turn will be consumed by the queue manager.

      We can examine the `queuemanager.yaml` using the below command:

      ```bash
      cat kustomize/components/dynamic-mqsc/generic-qmgr/queuemanager.yaml
      ```
      See how the `configmap` is being consumed by the queue manager :

      ``` { .yaml hl_lines="16 17 18 19" .no-copy }
      apiVersion: mq.ibm.com/v1beta1
      kind: QueueManager
      metadata:
        name: qm1
        annotations:
          argocd.argoproj.io/sync-wave: "300"
          helm.sh/hook-weight: "300"
      spec:
        template:
          pod:
            volumes:
              - name: config-volume-scripts
                configMap:
                  name: scripts-configmap
                  defaultMode: 0777
              - name: dynamic-config-volume-mqsc
                configMap:
                  name: dynamic-mqsc-configmap
                  defaultMode: 0777
            containers:
              - env:
                  - name: MQSNOAUT
                    value: 'yes'
                name: qmgr
                volumeMounts:
                - name: config-volume-scripts
                  mountPath: /mq-config
                  readOnly: true
                  #optional: true
                - name: dynamic-config-volume-mqsc
                  mountPath: /dynamic-mq-config-mqsc
                  readOnly: true
                  #optional: true
      ```

4. *Review the scripts that enable dynamic configurations*

      Below scripts continuously monitor the `dynamic-definitions.mqsc` for updates. If there are any new changes, these scripts will load the latest definition into the queue manager.

      We can examine the scripts using the below command:

      ```bash
      cat kustomize/components/scripts/start-mqsc.sh
      ```
      See how the start script is defined :

      ``` { .bash .no-copy }
      #!/bin/bash
      #
      # A simple MVP script that will run MQSC against a queue manager.
      ckksum=""

      # Outer loop that keeps the MQ service running
      while true; do

         tmpCksum=`cksum /dynamic-mq-config-mqsc/dynamic-definitions.mqsc | cut -d" " -f1`

         if (( tmpCksum != cksum ))
         then
            cksum=$tmpCksum
            echo "Applying MQSC"
            runmqsc $1 < /dynamic-mq-config-mqsc/dynamic-definitions.mqsc
         else
            sleep 3
         fi

      done
      ```

      ```bash
      cat kustomize/components/scripts/stop-mqsc.sh
      ```
      See how the stop script is defined :

      ``` { .bash .no-copy }
      #!/bin/bash
      echo "done"
      ```

5. *Review the definition to auto-run these scripts in queue manager*

      In order to auto run these scripts inside the queue manager, the following definitions are required.

      We can examine the definition using the below command:

      ```bash
      cat kustomize/base/generic-qmgr/static-definitions.mqsc
      ```
      See how the `SERVICE` is defined :

      ``` { .text hl_lines="12" .no-copy }
      DEFINE QLOCAL(IBM.DEMO.Q) BOQNAME(IBM.DEMO.Q.BOQ) BOTHRESH(3) REPLACE
      DEFINE QLOCAL(IBM.DEMO.Q.BOQ) REPLACE
      * Use a different dead letter queue, for undeliverable messages
      DEFINE QLOCAL('DEV.DEAD.LETTER.QUEUE') REPLACE
      ALTER QMGR DEADQ('DEV.DEAD.LETTER.QUEUE')
      DEFINE CHANNEL('IBM.APP.SVRCONN') CHLTYPE(SVRCONN)
      ALTER QMGR CHLAUTH (DISABLED)
      * DEFINE CHANNEL('MONITORING_CHL') CHLTYPE(SVRCONN)
      * SET CHLAUTH(MONITORING_CHL) TYPE(BLOCKUSER) USERLIST(NOBODY)
      REFRESH SECURITY TYPE(CONNAUTH)
      * optional
      * DEFINE SERVICE(APPLY_MQSC) CONTROL(QMGR) SERVTYPE(SERVER) STARTCMD('/mq-config/start-mqsc.sh') STARTARG(QM1) STOPCMD('/mq-config/start-mqsc.sh') STOPARG('') STDOUT('') STDERR('')
      ```

      This service is responsible for starting and stopping the scripts which monitors the dynamic configurations and loads them into the queue manager.

## Enable the dynamic configurations

1. *Enable the service that initiates the scripts in the queue manager*

      Let’s examine the `kustomize/base/generic-qmgr/static-definitions.mqsc`
      to see how to enable the service that initiates the scripts in the queue manager.

      Issue the following command:

      ```bash
      cat kustomize/base/generic-qmgr/static-definitions.mqsc
      ```

      We can see the contents of the `static-definitions.mqsc`:

      ``` { .text .no-copy }
      DEFINE QLOCAL(IBM.DEMO.Q) BOQNAME(IBM.DEMO.Q.BOQ) BOTHRESH(3) REPLACE
      DEFINE QLOCAL(IBM.DEMO.Q.BOQ) REPLACE
      * Use a different dead letter queue, for undeliverable messages
      DEFINE QLOCAL('DEV.DEAD.LETTER.QUEUE') REPLACE
      ALTER QMGR DEADQ('DEV.DEAD.LETTER.QUEUE')
      DEFINE CHANNEL('IBM.APP.SVRCONN') CHLTYPE(SVRCONN)
      ALTER QMGR CHLAUTH (DISABLED)
      * DEFINE CHANNEL('MONITORING_CHL') CHLTYPE(SVRCONN)
      * SET CHLAUTH(MONITORING_CHL) TYPE(BLOCKUSER) USERLIST(NOBODY)
      REFRESH SECURITY TYPE(CONNAUTH)
      * optional
      * DEFINE SERVICE(APPLY_MQSC) CONTROL(QMGR) SERVTYPE(SERVER) STARTCMD('/mq-config/start-mqsc.sh') STARTARG(QM1) STOPCMD('/mq-config/start-mqsc.sh') STOPARG('') STDOUT('') STDERR('')
      ```

      Open `kustomize/base/generic-qmgr/static-definitions.mqsc` and uncomment the service definition in the above file as follows :

      ```
      DEFINE QLOCAL(IBM.DEMO.Q) BOQNAME(IBM.DEMO.Q.BOQ) BOTHRESH(3) REPLACE
      DEFINE QLOCAL(IBM.DEMO.Q.BOQ) REPLACE
      * Use a different dead letter queue, for undeliverable messages
      DEFINE QLOCAL('DEV.DEAD.LETTER.QUEUE') REPLACE
      ALTER QMGR DEADQ('DEV.DEAD.LETTER.QUEUE')
      DEFINE CHANNEL('IBM.APP.SVRCONN') CHLTYPE(SVRCONN)
      ALTER QMGR CHLAUTH (DISABLED)
      * DEFINE CHANNEL('MONITORING_CHL') CHLTYPE(SVRCONN)
      * SET CHLAUTH(MONITORING_CHL) TYPE(BLOCKUSER) USERLIST(NOBODY)
      REFRESH SECURITY TYPE(CONNAUTH)
      * optional
      DEFINE SERVICE(APPLY_MQSC) CONTROL(QMGR) SERVTYPE(SERVER) STARTCMD('/mq-config/start-mqsc.sh') STARTARG(QM1) STOPCMD('/mq-config/start-mqsc.sh') STOPARG('') STDOUT('') STDERR('')
      ```

      Commit and push changes to your git repository:

      ```bash
      git add .
      git commit -s -m "Enable svc to pickup the dynamic scripts"
      git push origin $GIT_BRANCH
      ```

      The changes have now been pushed to your GitOps repository:

      ```{ .text .no-copy }
      Enumerating objects: 11, done.
      Counting objects: 100% (11/11), done.
      Delta compression using up to 8 threads
      Compressing objects: 100% (6/6), done.
      Writing objects: 100% (6/6), 548 bytes | 548.00 KiB/s, done.
      Total 6 (delta 3), reused 0 (delta 0)
      remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
      To https://github.com/prod-ref-guide/mq-qm01
         b1edbac..9001fc5  master -> master
      ```

2. *Enable the components in the queue manager (Optional)*

     This is an optional step and is only useful if the developers want to test the dynamic configurations from their end while building the queue manager source code locally.

    !!! note
        Enabling dynamic configurations at this step is purely optional. If you don't want to enable them in the queue manager source repository, you can simply ignore this step and move to the next step. This is only used to test the queue manager locally and these resources are not consumed by the pipeline.

      Examine the `kustomize/base/generic-qmgr/kustomization.yaml` and
      enable the necessary components.

      Issue the following command:

      ```bash
      cat kustomize/base/generic-qmgr/kustomization.yaml
      ```

      We can see the contents of the `kustomization.yaml`:

      ``` { .yaml .no-copy }
      resources:
      - ./queuemanager.yaml

      generatorOptions:
       disableNameSuffixHash: true
      # We use a configMapGenerator because it allows us to build up the mqsc from regular MQSC files.
      configMapGenerator:
      # Create an MQSC configMap using generic MQSC which will be added to all queue managers and applied during bootstrap.
      - name: mqsc-configmap
        behavior: create
        files:
        - static-definitions.mqsc

      # Add the configMap that will be used for dynamic updates, this should be used queue manager wide i.e. stay the same in each environment.
      # components:
      # - ../../components/dynamic-mqsc/generic-qmgr
      # - ../../components/scripts
      ```

      Open `kustomize/base/generic-qmgr/kustomization.yaml` and uncomment the below lines :

      ``` { .yaml .no-copy }
      components:
      - ../../components/dynamic-mqsc/generic-qmgr
      - ../../components/scripts
      ```

      You will have the following contents after the modification:

      ```{ .yaml .no-copy }
      resources:
      - ./queuemanager.yaml

      generatorOptions:
       disableNameSuffixHash: true
      # We use a configMapGenerator because it allows us to build up the mqsc from regular MQSC files.
      configMapGenerator:
      # Create an MQSC configMap using generic MQSC which will be added to all queue managers and applied during bootstrap.
      - name: mqsc-configmap
        behavior: create
        files:
        - static-definitions.mqsc

      # Add the configMap that will be used for dynamic updates, this should be used queue manager wide i.e. stay the same in each environment.
      components:
      - ../../components/dynamic-mqsc/generic-qmgr
      - ../../components/scripts
      ```

      Commit and push changes to your git repository:

      ```bash
      git add .
      git commit -s -m "Enable dynamic mqsc components"
      git push origin $GIT_BRANCH
      ```

      The changes have now been pushed to your GitOps repository:

      ```{ .text .no-copy }
      Enumerating objects: 11, done.
      Counting objects: 100% (11/11), done.
      Delta compression using up to 8 threads
      Compressing objects: 100% (6/6), done.
      Writing objects: 100% (6/6), 578 bytes | 578.00 KiB/s, done.
      Total 6 (delta 3), reused 0 (delta 0)
      remote: Resolving deltas: 100% (3/3), completed with 3 local objects.
      To https://github.com/prod-ref-guide/mq-qm01
         9001fc5..be5d38d  master -> master
      ```

      With this settings, we will now be able to deploy a queue manager with dynamic configurations.

## Set up the GitOps environment

1. *Navigate to multi-tenancy-gitops-apps repository*

      Issue the below command to navigate to apps repository:

      ```bash
      cd $GIT_ROOT
      cd multi-tenancy-gitops-apps
      ```

2. *Check the queue manager resources in dev*

      Issue the following command to see how the dev components of queue manager are structured out:

      ```bash
      tree mq/environments/dev/mq-qm01/ -L 3
      ```

      You should see a set of resources in the GitOps framework at the `mq-qm01` dev layer:

      ``` { .text .no-copy }
      mq/environments/dev/mq-qm01/
      ├── certificates
      │   ├── dev-mq-client-certificate.sh
      │   ├── dev-mq-client-certificate.yaml_template
      │   ├── dev-mq-server-certificate.sh
      │   └── dev-mq-server-certificate.yaml_template
      ├── components
      │   ├── dynamic-mqsc
      │   │   └── generic-qmgr
      │   └── scripts
      │       ├── kustomization.yaml
      │       ├── start-mqsc.sh
      │       └── stop-mqsc.sh
      ├── configmap
      ├── kustomization.yaml
      ├── queuemanager
      │   └── hooks
      │       ├── post-sync-job.sh
      │       └── post-sync-job.yaml_template
      └── secrets
          ├── mq-client-jks-password-secret.sh
          └── mq-client-jks-password-secret.yaml

      9 directories, 12 files
      ```

3. *Add the components to the queue manager*

      Examine the `kustomization.yaml` of `mq-qm01` at `dev` environment using the below command:

      ```bash
      cat mq/environments/dev/mq-qm01/kustomization.yaml
      ```

      See how the resources and components are defined :

      ``` { .yaml .no-copy }
      resources:
      - ../../base/mq-qm01
      # - queuemanager/hooks/post-sync-job.yaml
      # - certificates/dev-mq-client-certificate.yaml
      # - certificates/dev-mq-server-certificate.yaml
      # - secrets/mq-client-jks-password-secret.yaml

      # Add the configMap that will be used for dynamic updates, this should be used queue manager wide i.e. stay the same in each environment.
      # components:
      # - components/dynamic-mqsc/generic-qmgr
      # - components/scripts

      generatorOptions:
       disableNameSuffixHash: true
      # We use a configMapGenerator because it allows us to build up the mqsc from regular MQSC files.
      configMapGenerator:
      # Create an MQSC configMap using generic MQSC which will be added to all queue managers and applied during bootstrap.
      - name: mqsc-configmap
        behavior: create
        files:
        - configmap/static-definitions.mqsc
      patchesStrategicMerge:
       - queuemanager/queuemanager.yaml
       # - components/dynamic-mqsc/generic-qmgr/queuemanager.yaml
      ```

      Open `mq/environments/dev/mq-qm01/kustomization.yaml` and
      uncomment the below resources:  

      ``` { .yaml .no-copy }
      components:
      - components/dynamic-mqsc/generic-qmgr
      - components/scripts

      patchesStrategicMerge:
       - components/dynamic-mqsc/generic-qmgr/queuemanager.yaml
      ```

      You will have the following definition now:

      ``` { .yaml .no-copy }
      resources:
      - ../../base/mq-qm01
      # - queuemanager/hooks/post-sync-job.yaml
      # - certificates/dev-mq-client-certificate.yaml
      # - certificates/dev-mq-server-certificate.yaml
      # - secrets/mq-client-jks-password-secret.yaml

      # Add the configMap that will be used for dynamic updates, this should be used queue manager wide i.e. stay the same in each environment.
      components:
      - components/dynamic-mqsc/generic-qmgr
      - components/scripts

      generatorOptions:
       disableNameSuffixHash: true
      # We use a configMapGenerator because it allows us to build up the mqsc from regular MQSC files.
      configMapGenerator:
      # Create an MQSC configMap using generic MQSC which will be added to all queue managers and applied during bootstrap.
      - name: mqsc-configmap
        behavior: create
        files:
        - configmap/static-definitions.mqsc
      patchesStrategicMerge:
       - queuemanager/queuemanager.yaml
       - components/dynamic-mqsc/generic-qmgr/queuemanager.yaml
      ```

      Commit and push changes to your git repository:

      ```bash
      git add .
      git commit -s -m "Enable qmgr dynamic configurations"
      git push origin $GIT_BRANCH
      ```

      The changes have now been pushed to your GitOps repository:

      ```{ .text .no-copy }
      Enumerating objects: 13, done.
      Counting objects: 100% (13/13), done.
      Delta compression using up to 8 threads
      Compressing objects: 100% (7/7), done.
      Writing objects: 100% (7/7), 595 bytes | 595.00 KiB/s, done.
      Total 7 (delta 6), reused 0 (delta 0)
      remote: Resolving deltas: 100% (6/6), completed with 6 local objects.
      To https://github.com/prod-ref-guide/multi-tenancy-gitops-apps.git
         3696437..3c4f300  master -> master
      ```

      The intention of this operation is to indicate that we'd like the resources
      and components declared in `mq/environments/dev/mq-qm01/kustomization.yaml` to be
      deployed in the cluster.


!!! success "Congratulations!"

      You've now done the set up for the enabling the dynamic configurations on the queue manager. In the next topic of this chapter, we're going to use the previous pipeline we built to deploy a fully tested queue manager `QM1` with dynamic configurations enabled to the `dev` namespace. We'll explore the pipeline, tasks and steps in more detail to see exactly how they work.
