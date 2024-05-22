# Tools

<p align="center">
    <a href="https://stackoverflow.com/questions/tagged/moryx">
        <img src="https://img.shields.io/badge/stackoverflow-ask-orange.svg" alt="Stackoverflow">
    </a>
</p>

## Overview

This repo contains reusable GitHub workflows, used by the MORYX team to maintain 
our CI pipelines.

## Example

```yml
jobs:
  Variables:
    runs-on: ubuntu-latest
    steps:
      - name: Set Release variables 
        id: set-variables
        run: echo ''
    outputs:
      dotnet_sdk_version: 6
      PACKAGE_TARGET: <nuget-package-feed>
      PACKAGE_TARGET_V3: <nuget-package-feed/v3>
      NPM_PACKAGE_SOURCE: <npm-package-source>

  Build:
    needs: [Variables]
    uses: moryx-industry/tools/.github/workflows/build.yml@v1
    with:
      dotnet_sdk_version: ${{ needs.Variables.outputs.dotnet_sdk_version }}
      NPM_PACKAGE_SOURCE: ${{ needs.Variables.outputs.NPM_PACKAGE_SOURCE }}
    secrets:
      npm_auth_token: ${{ secrets.MYGET_TOKEN }}
      myget_auth_token: ${{ secrets.MYGET_TOKEN }}
      myget_user: ${{ secrets.MYGET_USER }}
      myget_pass: ${{ secrets.MYGET_PASS }}

  Tests:
    needs: [Variables, Build]
    uses: moryx-industry/tools/.github/workflows/run-unit-tests.yml@v1
    with:
      dotnet_sdk_version: ${{ needs.Variables.outputs.dotnet_sdk_version }}
      npm_package_source: ${{ needs.Variables.outputs.NPM_PACKAGE_SOURCE }}
    secrets:
      npm_auth_token: ${{ secrets.MYGET_TOKEN }}
      myget_auth_token: ${{ secrets.MYGET_TOKEN }}
      myget_user: ${{ secrets.MYGET_USER }}
      myget_pass: ${{ secrets.MYGET_PASS }}

  Licensing:
    needs: [Tests]
    uses: moryx-industry/tools/.github/workflows/licensing.yml@v1

  create-report:
    needs: [Licensing]
    concurrency:
      group: publish
    uses: moryx-industry/tools/.github/workflows/create-test-report.yml@v1
                
  publish-reports:
    needs: [create-report]
    concurrency:
      group: publish
    uses: moryx-industry/tools/.github/workflows/publish-test-coverage.yml@v1
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  Publish-Packages:
    needs: [Variables, Licensing]
    uses: moryx-industry/tools/.github/workflows/publish-packages.yml@v1
    with:
      dotnet_sdk_version: ${{ needs.Variables.outputs.dotnet_sdk_version }}
      npm_package_source: ${{ needs.Variables.outputs.NPM_PACKAGE_SOURCE }}
      nuget_package_target: ${{ needs.Variables.outputs.PACKAGE_TARGET }}
      nuget_package_target_v3: ${{ needs.Variables.outputs.PACKAGE_TARGET_V3 }}
    secrets:
      npm_auth_token: ${{ secrets.MYGET_TOKEN }}
      myget_auth_token: ${{ secrets.MYGET_TOKEN }}
      myget_user: ${{ secrets.MYGET_USER }}
      myget_pass: ${{ secrets.MYGET_PASS }}
    if: |
      ${{ github.ref_name }} == 'dev' ||
      ${{ github.ref_name }} == 'future' ||
      ${{ (startsWith(github.ref, 'refs/tags/v')) }}
       
```

## Contribute

If you have an idea to improve a template or can think of a new useful template,
please make your changes based on one of the template branches and open a pull
request. If you want to add a template, extend the branch list in one commit and
the template definition in another. This way we can easily put your template
into a separate branch. **Note:** All branches except *master* will be rebased
regularly, to keep grafting them easy. To avoid losing previous merge request
information, all branch merge requests are merged by rebase squashing.