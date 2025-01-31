# build-and-push-ecr
Defines an action that aids building and pushing ECR images to the IDP ECR registry. 

# Connecting and authorizing
Githubs OIDC provider is used to authorize access to ECR. This must be setup in advance to
using this action. 

We use prefixes in combination with role names, to determine if a given repo is allowed 
to create or push to a certain repository. 

Input parameters should be fed both a role name - which is assumed upon AWS login - and 
a prefix. 

# Simple run
If you just want to build a Dockerfile in the current folder of the action, tag it 
and push it to ECR (optionally creating the repository if it does not exist), you should 
do something like

```yaml
name: Build docker image using action
on:
  push:
  workflow_dispatch:
jobs:
  build_and_push:
    runs-on: ubuntu-24.04
    permissions: write-all
    strategy:
      fail-fast: false
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Build and push
        uses: jppol-idp/build-and-push-ecr@v2
        with:
          image_name: mf-create-by-action
          namespace: idp
          image_tag: ${{ github.run_number }}
```
In this example we get a push to `354918371398.dkr.ecr.eu-west-1.amazonaws.com/idp/mf-create-by-action`


# Building outside the action
In the standard scenario the action is expected to build a Dockerfile during run. If this 
is not desired it can be disabled using the settings. In this scenario the caller must 
ensure that an image has been tagged as expected by another step in the workflow. 

The full image name with registry referance and tag can be produced by running this 
actions once with both pushing and building enabled and capturing the output `fully_qualified_name`.

Example: 
```yaml

name: Custom build and action push
on:
  push:
  workflow_dispatch:
jobs:
  build_and_push:
    runs-on: ubuntu-24.04
    permissions: write-all
    strategy:
      fail-fast: false
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Create tag
        id: create_tag
        uses: jppol-idp/build-and-push-ecr@v2
        with:
          image_name: testing-ecr-push
          namespace: idp
          image_tag: ${{ github.run_number }}
          run_build: false
          run_push: false
      - name: Build
        run: |
          docker build . --build-arg DUMMY=FOO2 -t ${{ steps.create_tag.outputs.fully_qualified_name }}
      - name: Push 
        id: push
        uses: jppol-idp/build-and-push-ecr@v2
        with:
          image_name: testing-ecr-push
          namespace: idp
          image_tag: ${{ github.run_number }}
          run_build: false
```

# Other parameters
|parameter |description | default |
|----------|------------|---------|
|image\_name| Name of the image | none |
|image\_tag | Tag of this specific build. Tags are immutable, use run\_number from github context or other changing value. | none |
| docker\_folder | Use if Dockerfile is in other location than context root. Should be relative to context root. | `.` |
| namespace | Prefixed to image name and concatenated with a slash | none |
| ecr\_account\_id | The ECR account. Use default. | ------- |
| aws\_region | Region for ECR. | `eu-west-1` |
| run\_build | When set to false, no docker build is run. | `true` |
| run\_push | When true push to ECR is attempted. | `true` |

