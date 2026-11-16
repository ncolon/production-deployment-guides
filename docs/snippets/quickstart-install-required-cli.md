## Install required CLIs

- Install the [Github CLI](https://github.com/cli/cli) (version 1.14.0+)
- Install the [OpenShift CLI](https://docs.openshift.com/container-platform/4.8/cli_reference/openshift_cli/getting-started-cli.html) `oc` (version 4.7 or 4.8)

    - Log into your OCP cluster, substituting the `--token` and `--server` parameters with your values:

        ```bash
        oc login --token=<token> --server=<server>
        ```

        If you are unsure of these values, click your user ID in the OpenShift web console and select `Copy login command`.

- Install the [kubeseal CLI](https://github.com/bitnami-labs/sealed-secrets#homebrew) 

    The kubeseal utility uses asymmetric crypto to encrypt secrets that only the controller can decrypt. 
    These encrypted secrets are encoded in a SealedSecret resource which can safely be checked into your gitops repo. The controller will decrypt the secretes and install into your cluster. 


