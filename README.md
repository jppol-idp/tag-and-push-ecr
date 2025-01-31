# build-and-push-ecr
Defines an action that aids tagging and pushing ECR images to the JP/Politikens Hus IDP ECR registry. 

# Connecting and authorizing
Githubs OIDC provider is used to authorize access to ECR. This must be setup in advance to
using this action. 

We use prefixes in combination with role names, to determine if a given repo is allowed 
to create or push to a certain repository. 

Input parameters should be fed both a role name - which is assumed upon AWS login - and 
a prefix. 

The imaage must be present on the runner prior to pushing and the action needs a specific 
source tag (or sha) to be able to tag and push.

# Example
If you just want to build a Dockerfile in the current folder of the action, tag it 
and push it to ECR (optionally creating the repository if it does not exist), you should 
do something like

```yaml

name: Custom build and tag...
on:
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
      - name: Build
        id: build-the-schizz
        run: |
          docker build . --build-arg DUMMY=FOO2 -t local-build
      - name: Push 
        id: push
        uses: jppol-idp/tag-and-push-ecr@main
        with:
          source_tag: local-build
          image_name: testing-ecr-tag
          namespace: idp
          image_tags: dev-${{ github.run_number }} test-${{ github.run_number }} ${{ github.sha }}
```
In this example we get a push to `354918371398.dkr.ecr.eu-west-1.amazonaws.com/idp/testing-ecr-tag`



