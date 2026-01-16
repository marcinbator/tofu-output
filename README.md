# Tofu output Gthub Actions composite

## Description

This action composite allows to export OpenTofu outputs in json format to GITHUB_OUTPUT using base64 encoding.

## Inputs

```yml
inputs:
  workdir:
    required: true
    description: TF working directory
```

## Example usage

```yml
on:
  workflow_call:
    outputs:
      tf_outputs:
        value: ${{ jobs.rds.outputs.tf_outputs }}

jobs:
  rds:
    runs-on: ubuntu-latest
    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_DEFAULT_REGION: ${{ secrets.AWS_DEFAULT_REGION }}

      WORKDIR: ./rds
    outputs:
      tf_outputs: ${{ steps.output.outputs.tf_outputs }}
    defaults:
      run:
        working-directory: ${{ env.WORKDIR }}

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup OpenTofu
        uses: opentofu/setup-opentofu@v1
        with:
          tofu_version: 1.11.2

      - name: TF init
        run: tofu init

      - name: TF Apply
        run: tofu apply -auto-approve -input=false

      - id: output
        name: Output TF vars
        uses: marcinbator/tofu-output@1.0.0
        with:
          workdir: ${{ env.WORKDIR }}
```
