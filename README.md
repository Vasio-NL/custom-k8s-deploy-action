## Kubernetes deploy action

This Github Action deploys images from a container registry to a kubernetes cluster.
For deploying images from Amazon ECR, see the [ECR deploy action](https://github.com/Vasio-NL/custom-k8s-deploy-ECR-action).

This action is intended to be used with the custom [build and push action](https://github.com/Vasio-NL/custom-build-and-push-action).

Fetches the latest version of the given image from a kubernetes configmap, then gets the image from the container registry and pushes it to the kubernetes cluster.

### Inputs

| Name                    | Description | Required |
|-------------------------| --- | --- |
| kube-config-base64      | The base64 encoded kubeconfig needed to connect to the cluster | true |
| container-registry-url  | URL for the container registry | true |
| container-registry-name | The name of the container registry | true |

The container registry name is the name that is prefixed to the image name. An example image:

`registry.digitalocean.com/vasio/cool-project:latest`

In this example:
- The <b>registry url</b> is `registry.digitalocean.com`.
- The <b>container registry name</b> is `vasio`.
- The <b>image name</b> is `cool-project`.

### Example usage

The following is an example release job:

```
release:
    runs-on: ubuntu-latest
    environment: ${{ github.ref_name }}
    needs: [ build_app ]
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Deploy to kubernetes
        uses: Vasio-NL/custom-k8s-deploy-action@v1
        with:
          container-registry-url: ${{ vars.REGISTRY_URL }}
          container-registry-name: vasio
          kube-config-base64: ${{ secrets.KUBE_CONFIG_B64 }}
```
### Deployment manifests

This action uses [kustomize](https://kustomize.io/) to bundle the deployment manifests. It assumes that the manifests are located in the `manifests` directory at the root level of the repository.
It checks for a folder containing the name of the branch. An exception is made for the default branch, which will always check `manifests/production`.

<b>Important:</b> The action assumes your repositoy's default branch is the production branch, and will always look for the production manifests when running this action from the default branch.


#### Example manifests structure

```
manifests
├── base
│   ├── kustomization.yml
│   ├── deployment.yml
│   └── service.yml
├── production
│   ├── kustomization.yml
│   ├── update-deployment.yml
│   └── update-service.yml
└── staging
    ├── kustomization.yml
    ├── update-deployment.yml
    └── update-service.yml
```

Recommended usage is to have a base folder containing your manifests, for example `manifests/base`.
Inside should be a kustomization.yml, which references the manifests you want to deploy.

An example:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yml
  - service.yml
```

You can then patch these manifest files using kustomize for each environment stored in `manifests/<environment>` by adding a kustomize.yml file which contains a patchesStrategicMerge section.
The environment name should match the branch name from which the action is running.

An example:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../base
patchesStrategicMerge:
  - update-deployment.yml
  - update-service.yml
```



### For developers: Updating the action
When making changes, make sure to tag new versions so they can be used in Github workflows. The action uses semantic versioning.

To tag the version (v1.0.3 in the example):

`git tag v1.0.3`

also update the major version tag (v1 in the example):

`git tag v1 -f`

When finished tagging, make sure to push the tags:

`git push --tags -f`
