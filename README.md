# Export Tofu output Gthub Actions composite

## Description

This action composite allows to export OpenTofu outputs in json format to GITHUB_OUTPUT using base64 encoding.

The original idea for this was found on <https://github.com/orgs/community/discussions/25225#discussioncomment-6776295> discussion and belongs to [rdhar](https://github.com/rdhar).

## Inputs

```yml
inputs:
  workdir:
    required: true
    description: Tofu working directory
```

## Example usage

This composite always puts all the Tofu outputs together in single `tf_outputs` GITHUB_OUTPUTS variable. It can be accessed later only if the step has an `id` attribute.

To use exported outputs in another Tofu job/workflow you can use `marcinbator/import-tofu-output@v1.0.0` (<https://github.com/marcinbator/import-tofu-output>) composite which combines all of exported outputs into one .tfvars.json file that can be used by Tofu (as shown in [Full example](#full-example) below).

Make sure every Tofu output has unique name to avoid errors during Tofu execution.

```yml
- id: output
  name: Output TF vars
  uses: marcinbator/export-tofu-output@v1.0.0
  with:
    workdir: ${{ env.WORKDIR }}
```

## Full example

```yml
jobs:
  rds: # this job exports outputs to GITHUB_OUTPUT using marcinbator/export-tofu-output@v1.0.0. Outputs are always exported to `tf_outputs` output.
    runs-on: ubuntu-latest
    outputs:
      tf_outputs: ${{ steps.output.outputs.tf_outputs }}
    defaults:
      run: working-directory:./rds

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
        uses: marcinbator/export-tofu-output@v1.0.0 # here
        with:
          workdir: ./rds

  db: # this job uses outputs exported in previous job
    runs-on: ubuntu-latest
    needs: rds
    defaults:
      run:
        working-directory: ./db

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup OpenTofu
        uses: opentofu/setup-opentofu@v1
        with:
          tofu_version: 1.11.2

      - name: Extract TF variables
        uses: marcinbator/import-tofu-output@v1.0.0 # here
        with:
          workdir: ./db
        env:
          rds_outputs: ${{ needs.rds.outputs.tf_outputs }} #outputs needs to be passed in env variable with `_outputs` suffix.

      - name: TF init
        run: tofu init

      - name: TF Apply
        run: tofu apply -auto-approve -input=false
```
