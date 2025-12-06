# Uses

A catalog of common uses of the ojob action.

## Check In

This oJob.io job allows for check-in changed files following the repository rules. This means that it will try to commit changes first and if not possible it will create a PR with the corresponding branch, It will try to auto-approve and if not possible it will leave it for "human review". In case that a PR cannot be created it will rollback all actions.

````yaml
permissions:
  contents: write
  pull-requests: write

steps:
- name: Commit changes
  env :
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  uses: openaf/ojob-action@v7
    with:
      ojob: "ojob.io/git/hub/contentIn"
      args: "message=\"Update files\" title=\"Automated update PR\" paths=\"README.md,changed/path/\" branch=\"${{ github.ref_name }}\""
      dist: nightly
````

See more options, including branch prefix, custom PR body content and others in [ojob.io/git/hub/contentIn](https://ojob.io/git/hub/contentIn.md).

## Generate a container image scan badge

This oJob.io job will use a docker container to scan the provided image with Trivy, summarize the number of critical, high, medium and low issues and produce a SVG banner to be included into a markdown file. The color will depend on the highest severity with issues. It can also generate a YAML file with the Trivy results to drill down for details.

````yaml
steps:
- name: Scan latest
  uses: openaf/ojob-action@v7
  with:
    ojob: 'ojob.io/sec/genSecBadge'
    args: 'image=openaf/oaf:latest file=.github/sec-latest.svg reportFile=.github/sec-latest.yaml'
    dist: nightly
````

See more options in [ojob.io/sec/genSecBadge](https://ojob.io/sec/genSecBadge.md)

You can also convert the YAML results into a markdown with a fixed-width tree display for easier reading:

````yaml
steps:
- name: Generate sec-latest.md
  uses: openaf/ojob-action@v7
  with:
    ojob: 'ojob.io/util/toMDTree'
    args: 'inputFile=.github/sec-latest.yaml file=.github/sec-latest.md'
````

The result will be something similar to:

[![generated badge](https://raw.githubusercontent.com/OpenAF/openaf-dockers/master/.github/sec-latest.svg)](https://github.com/OpenAF/openaf-dockers/blob/master/.github/sec-latest.md)

On the target markdown file add:
````markdown
[![generated badge](.github/sec-latest.svg)](.github/sec-latest-md)
````

> Use the "Check In" oJob to commit the generated files

## Preload Trivy database for image scan badge

To retrieve the trivy database to cache:

```yaml
steps:
- name: Retrieve trivy java database
  uses: openaf/ojob-action@v7
  with:
    ojob: 'ojob.io/sec/trivy'
    args: 'dockerOptions="-v trivy-db:/root/.cache/trivy" options="image --download-java-db-only"'

- name: Retrieve trivy database
  uses: openaf/ojob-action@v7
  with:
    ojob: 'ojob.io/sec/trivy'
    args: 'dockerOptions="-v trivy-db:/root/.cache/trivy" options="image --download-db-only"'

- name: Store the trivy databases in cache
  run : |
    docker run --rm -v trivy-db:/volume -v /tmp/oaf/trivy-db:/backup busybox tar czf /backup/trivy-db.tgz -C /volume .
```

To restore the trivy database from cache:

```yaml
- name: Restore the trivy databases from cache
  run : |
    docker run --rm -v trivy-db:/volume -v /tmp/oaf/trivy-db:/backup busybox tar xzf /backup/trivy-db.tgz -C /volume
```

## Generate a custom badge

You can also write a quick oJob definition to generate a badge with custom values. Here is an example of retriving of extracting the current version from a package.yaml file and creating a badge with it:

````yaml
  - name: Update version badge
    uses: openaf/ojob-action@v7
    with:
      def : |
        todo:
        - Update badge with version

        ojob:
          opacks:
          - Badgen

        include:
        - badgen.yaml

        jobs:
        # -------------------------------
        - name: Update badge with version
          from: Get version
          to  :
          - name: Badgen generate file
            args: 
              labelColor: grey3
              color     : blue
              icon      : "openaf_grey.svg"
              label     : "plugin-XLS"
              file      : ".github/version.svg"

        # -----------------
        - name: Get version
          exec: args.status = io.readFileYAML(".package.yaml").version
      dist: nightly
````

The result will be similar to:

![generated badge](https://raw.githubusercontent.com/OpenAF/openaf-opacks/master/.github/badges/plugin-XLS.svg)

On the target markdown file add:
````markdown
![generated badge](.github/version.svg)
````

> Use the "Check In" oJob to commit the generated files
