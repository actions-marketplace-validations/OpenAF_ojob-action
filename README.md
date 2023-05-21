# ojob-action

![version](.github/ojobs/version.svg)

OpenAF's oJob GitHub action to run generic [OpenAF](https://docs.openaf.io) [oJobs](https://docs.openaf.io/docs/concepts/oJob.html).

## Usage

On a GitHub action step add the following entry:

````yaml
  - uses: openaf/ojob-action@v1
    with:
      # the reference to a local oJob yaml/json file or a remote oJob.io
      ojob: '...' 
      # given the oJob referenced the key/value arguments to provide to it
      args: 'key1=value1 key2=value2 ...'
      # the public OpenAF distribution to use as runtime (e.g. nightly)
      # if no value is provided it defaults to the stable
      dist: '...'
````

## Examples

### Example of a GitHub action to run an ojob.io job:

````yaml
name: Get Envs
on: [push]

jobs:
  Get-Envs:
    runs-on: ubuntu-latest
    name: Get env variables
    steps:
    - uses: actions/checkout@v3
    - uses: openaf/ojob-action@v1
      with:
        ojob: 'ojob.io/envs'
````

### Example of a GitHub action to run an ojob.io job with arguments:

````yaml
name: Echo arguments
on: [push]

jobs:
  Echo-Arguments:
    runs-on: ubuntu-latest
    name: Echo arguments
    steps:
    - uses: actions/checkout@v3
    - uses: openaf/ojob-action@v1
      with:
        ojob: 'ojob.io/echo'
        args: 'abc=123 xyz=abc'
````

### Example of a GitHub action to security scan a container image and produce a badge:

````yaml
name: Scan Images

on:
  push:
    branches: [ "master" ]
  schedule:
    - cron: '00 1 * * 6'

jobs:
  Scan-Images:
    runs-on: ubuntu-latest
    name: Scan images
    steps:
    - uses: actions/checkout@v3
    - uses: openaf/ojob-action@v1
      with:
        ojob: 'ojob.io/sec/genSecBadge'
        args: 'image=some/image:latest file=.github/sec-latest.svg'
        dist: 'nightly'
    - uses: openaf/ojob-action@v1
      with:
        ojob: 'ojob.io/sec/genSecBadge'
        args: 'image=some/image:build file=.github/sec-build.svg'
        dist: 'nightly'
    - run: |
        git config --global user.email "openaf@users.noreply.github.com"
        git config --global user.name "OpenAF"
        if [[ $(git status --porcelain) ]]; then
          git add .github/sec-latest.svg
          git add .github/sec-build.svg
          git commit -m "update badge"
          git push
        fi
````

### Example of a GitHub action to retrieve the installed version and distribution of OpenAF:

````yaml
name: Get Version
on: [push]

jobs:
  Get-Version:
    runs-on: ubuntu-latest
    name: Get version
    steps:
    - uses: actions/checkout@v3
    - run : |
        cat <<EOF > getVersion.yaml
        todo:
        - check version
        
        jobs:
        - name: check version
          exec: |
            var data = { version: getVersion(), distribution: getDistribution() }
            ow.oJob.output(data, args)
        EOF
    - uses: openaf/ojob-action@v1
      with:
        ojob: 'getVersion.yaml'
        dist: 'nightly'
````
