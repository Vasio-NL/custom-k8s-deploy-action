## Kubernetes deploy action

This Github Action deploys images from a container registry to a kubernetes cluster.
This action is intended to be used with the custom [build and push action](https://github.com/Vasio-NL/custom-build-and-push-action).

Fetches the latest version of the given image from a kubernetes configmap, then gets the image from the container registry and pushes it to the kubernetes cluster.

### Example usage

The following is an example release job:

```
release:
    runs-on: ubuntu-latest
    environment: ${{ github.ref_name }}
    needs: [ build_app ]
    steps:
      - name: Deploy to kubernetes
        uses: Vasio-NL/custom-k8s-deploy-action@v1
        with:
          container-registry-url: ${{ vars.REGISTRY_URL }}
          container-repository-name: vasio
          kube-config-base64: ${{ secrets.KUBE_CONFIG_B64 }}
```

The container repository name is the name that is prefixed to the image name. An example image:

`registry.digitalocean.com/vasio/cool-project:latest`

In this example:
- The <b>registry url</b> is `registry.digitalocean.com`.
- The <b>container repository name</b> is `vasio`. 
- The <b>image name</b> is `cool-project`.

### For developers: Updating the action
When making changes, make sure to tag new versions so they can be used in Github workflows. The action uses semantic versioning.

To tag the version (v1.0.3 in the example):

`git tag v1.0.3`

also update the major version tag (v1 in the example):

`git tag v1 -f`

When finished tagging, make sure to push the tags:

`git push --tags -f`
