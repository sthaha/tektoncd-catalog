# Go Source-to-Image

This task can be used for building `GO` apps as reproducible Docker 
images using Source-to-Image. [Source-to-Image (S2I)](https://github.com/openshift/source-to-image)
is a toolkit and a workflow for building reproducible container images
from source code. This tasks uses the s2i-go image build from [sclorg/golang-container](https://github.com/sclorg/golang-container).

GO versions currently provided are:

- 1.11

## Installing the Go Task

```
kubectl apply -f https://raw.githubusercontent.com/openshift/pipelines-catalog/master/s2i-go/s2i-go-task.yaml
```

## Inputs

### Parameters

* **PATH_CONTEXT**: Source path from where S2I command needs to be run
  (_default: ._)
* **TLSVERIFY**: Verify the TLS on the registry endpoint (for push/pull to a
  non-TLS registry) (_default:_ `true`)


### Resources

* **source**: A `git`-type `PipelineResource` specifying the location of the
  source to build.

## Outputs

### Resources

* **image**: An `image`-type `PipelineResource` specifying the image that should
  be built.

## Creating a ServiceAccount

S2I builds an image and pushes it to the destination registry which is
defined as a parameter. The image needs proper credentials to be 
authenticated by the remote container registry. These credentials can 
be provided through a serviceaccount. See [Authentication](https://github.com/tektoncd/pipeline/blob/master/docs/auth.md#basic-authentication-docker)
for further details.

If you run on OpenShift, you also need to allow the service
account to run privileged containers. Due to security considerations 
OpenShift does not allow containers to run as privileged containers 
by default.

Run the following in order to create a service account named
`pipelines` on OpenShift and allow it to run privileged containers:

```
oc create serviceaccount pipeline
oc adm policy add-scc-to-user privileged -z pipeline
oc adm policy add-role-to-user edit -z pipeline
```

## Creating the taskrun

This TaskRun runs the Go Task to fetch a Git repository and builds and 
pushes a container image using S2I and a Go builder image.

```
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: s2i-go-taskrun
spec:
  # Use service account with git and image repo credentials
  serviceAccount: pipeline
  taskRef:
    name: s2i-go
  inputs:
    resources:
    - name: source
      resourceSpec:
        type: git
        params:
        - name: url
          value: https://github.com/username/reponame
  outputs:
    resources:
    - name: image
      resourceSpec:
        type: image
        params:
        - name: url
          value: quay.io/my-repo/my-image-name
```