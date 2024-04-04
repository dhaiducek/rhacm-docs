# Generating the API docs

Source: https://github.com/jboxman-rh/openshift-apidocs-gen/tree/master

## Prerequisites

- NodeJS - https://nodejs.org/en/download/current
- `jq` - https://jqlang.github.io/jq/
- `yq` - https://github.com/mikefarah/yq/#install
- `asciidoctor` - https://asciidoctor.org/

## Generating the API docs

1. Install the latest `openshift-apidocs-gen` CLI:

   ```shell
   npm install -g @jboxman-rh/openshift-apidocs-gen
   ```

1. Connect to a cluster that has the desired version of RHACM deployed, and fetch the OpenAPI specification from the
   cluster, filtering for open-cluster-management.io endpoints:

   ```shell
   oc get --raw /openapi/v2 | \
   jq '.paths |= with_entries(select(.key | test(".*\\.open-cluster-management.io.*")))' | \
   jq '.definitions |= with_entries(select(.key | startswith("io.open-cluster-management.")))' \
   > apis/openapi.json
   ```

1. Generate an API Map:

   ```shell
   sed -i '/apiMap:/q' apis/config.yaml
   openshift-apidocs-gen create-resources apis/openapi.json >> apis/config.yaml
   ```

1. Generate the API docs:

   ```shell
   openshift-apidocs-gen build -c apis/config.yaml -o apis/ apis/openapi.json
   ```
