# example-helm-repo

There is an inconsistency in how `helm dependency update` and `helm dependency
build` work. To show this I have created a simple umbrella chart which only
contains a single dependency.

## Requirements

1. Helm version (3.2.0) is installed
1. Bitnamis repository should not be in the list of helm repositories. (`helm
repo list` should not contain a repo with the url `https://charts.bitnami.com/bitnami`)
   * if it does, remove it with `helm repo remove`

## Problem

Helm allows you to define chart dependencies on other helm charts and keeps
track of them, we call these dependencies subcharts. Helm keeps track of which
version of the subcharts should be installed using a `Chart.lock` file.

The command `helm dependency update` does two things; pulls the charts for the
dependencies and stores it under the /charts folder and updates the
`Chart.lock` file with the version that it updated to.

The command `helm dependency build` looks at the `Chart.lock` file and pulls
the exact chart version specified there. This allows you to always use the same
versions of a subchart installed. If there is no `Chart.lock` file this will
run `helm dependency update` instead.

The problem here is that the two commands behave differently if the repository
has not been added. While `Helm dependency update` does not require that the
repository has been added `Helm dependency build` does.

This means that if there is a `Chart.lock` file present but the helm repository
has not been added `Helm dependency build` will fail while `Helm dependency
update` will succeed.

This is even more confusing since if the `Chart.lock` file isn't present `Helm
dependency build` will succeed.

## How to replicate

Go to the `umbrella/` directory and run `Helm dependency build`, this will
result in:

```
$ helm dependency build
Error: no repository definition for https://charts.bitnami.com/bitnami. Please add the missing repos via 'helm repo add'
```

Remove the `Chart.lock` file and run `Helm dependency build`, this will result
in:

```bash
$ helm dependency build
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading wordpress from repo https://charts.bitnami.com/bitnami
Deleting outdated charts
```

This successfully downloaded the subcharts and updated the `Chart.lock` file.
Running the command again will fail as it did before.

## Solution

We should make `Helm dependency build` not require the user to have added the
repository beforehand.
