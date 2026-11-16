# Installing Prerequisites

<!--- cSpell:ignore homebrew -->

**Audience**: Application developers, Administrators

## Overview

The production reference guide uses many different command line tools to allow
you interact with the cluster from your local machine. Tools such as `oc`, `git`
and `jq` help you to understand how a cloud native deployment works in a
hands-on manner.

For Windows users please follow these instructions [**windows.md**](./windows.md)

In this topic, we're going to:

* Learn how to install the prerequisite command line tools


=== "macOS"
=== "Linux"

Where appropriate, click on the **macOS** or **Linux** horizontal
tab to see the corresponding operating system instructions.

---

## Installing a package manager

A operating system specific package manager is used to install the prerequisite
tools on the different platforms.

=== "macOS"

    On macOS, we use the [**homebrew**](https://brew.sh/){:target="_blank" .external} package
    manager to install the prerequisite tools

    On the terminal command line, issue the following command to see if **homebrew**
    is installed:

    ```shell
    brew --version
    ```

    You should see the following output if homebrew is installed:

    ```{.shell .no-copy}
    Homebrew 3.3.5
    Homebrew/homebrew-core (git revision 63e2388f12d; last commit 2021-11-24)
    Homebrew/homebrew-cask (git revision b8049f243b; last commit 2021-11-24)
    ```

    If it is not installed, use the [Install Homebrew](https://brew.sh/){:target="_blank" .external}
    instructions.

=== "Linux"

    For Linux we will use RHEL 8 and `dnf` as an example, RHEL 7 users should be able to replace `dnf` with `yum`.

    Refer to the [**Red Hat Subscription Manager**](https://access.redhat.com/products/red-hat-subscription-management/){:target="_blank" .external} for instructions on setting up your RHEL subscriptions and repositories.

---

## Installing `git`

The `git` command line tool is used extensively throughout the tutorial to
interact with git and GitHub. We perform GitOps operations using commands such
as `git push` to make a change to a cluster.

Install `git` using the following command:

=== "macOS"

    ```bash
    brew install git
    ```

=== "Linux"

    ```bash
    dnf install git
    ```

Learn more about [`git`](https://git-scm.com/about){:target="_blank" .external}.

Use `git --version` to ensure you are using version 2 or higher.

---

## Installing `gh`

The `gh` command line tool is the official command line to interact with GitHub.
It is used some command scripts, for example to create a repository from a
GitHub template.

Install `gh` using the following command:

=== "macOS"

    ```bash
    brew install gh
    ```

=== "Linux"

    ```bash
    dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo
    dnf install gh
    ```


Learn more about [`gh`](https://cli.github.com/){:target="_blank" .external}.

Use `gh --version` to ensure you are using version 2 or higher.

---

## Installing `jq`

`jq` is a utility used to manipulate the JSON output typically returned by the
`oc` command when reading or updating Kubernetes resources.

Install `jq` using the following command:

=== "macOS"

    ```bash
    brew install jq
    ```

=== "Linux"

    ```bash
    dnf install jq
    ```

Learn more about [`jq`](https://stedolan.github.io/jq/){:target="_blank" .external}.

---

## Installing `node`, `npm` & `nvm`

This trio of technologies is used to install various
[`node.js`](https://nodejs.org/){:target="_blank" .external} based utilities
used in the tutorial.

!!! Note
    `nvm` is optional.

### node
Install `node` using the following command:

=== "macOS"

    ```bash
    brew install node
    ```

=== "Linux"

    ```bash
    dnf groupinstall 'Development Tools'
    dnf module install nodejs/development
    ```
### npm
`npm` is the package manager used to install different node.js packages.

Install `npm` using the following command:

=== "macOS"

    ```bash
    brew install npm
    ```

=== "Linux"

    `npm` gets installed as part of the 'Development Tools' in the `node` install step.


Learn more about [`npm`](https://www.npmjs.com/){:target="_blank" .external}.

### nvm
[nvm](https://github.com/nvm-sh/nvm#intro){:target="_blank" .external}
(node version manager) allows you to install and use different versions of `node.js`.

!!! Reminder
    Installing `nvm` is optional.

Install `nvm` using the following command:

=== "macOS"

    ```bash
    brew install nvm
    ```

=== "Linux"

    To install `nvm` on Linux see: [`nvm`](https://github.com/nvm-sh/nvm#installing-and-updating){:target="_blank" .external}

---

## Installing `tree`

`tree` is a command-line tool display a file system structure by displaying folders
and files in tree-like structure.

Install `tree` using the following command:

=== "macOS"

    ```bash
    brew install tree
    ```

=== "Linux"

    ```bash
    dnf install tree
    ```


Learn more about [`tree`](http://mama.indstate.edu/users/ice/tree/){:target="_blank" .external}.

---

## Installing `watch`

`watch` is a utility used to periodically monitor the output of a command. It is useful in checking the status of long running processes such as deployments.

Install `watch` using the following command:

=== "macOS"

    ```bash
    brew install watch
    ```

=== "Linux"

    ```bash
    dnf install watch
    ```


Learn more about [`watch`](https://gitlab.com/procps-ng/procps){:target="_blank" .external}.

---

## Installing `oc`

The `oc` command is used to interact with an OpenShift cluster. It is
effectively a synonym for the `kubectl` command with some additional options.

Use these
[instructions](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html)
to get the latest version of `oc`.

You can use `oc version` to confirm that you have:

  - `Client Version: 4.6` or higher
  - `Server Version: 4.7` or higher

The version of `oc` must be compatible with the version of the cluster you previously created.

!!! Tip
    Red Hat Linux users with an appropriate Red Hat subscription can also use `dnf` to install `oc`.

   ```bash
   subscription-manager list --available --matches '*OpenShift*'
   subscription-manager attach --pool=<pool id>
   subscription-manager repos --enable="rhocp-4.8-for-rhel-8-x86_64-rpms"
   dnf install openshift-clients
   ```
   Where 'pool id' is the id of a pool that has the OpenShift clients entitlement.

---
