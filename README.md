# ojob-action

![version](.github/ojobs/version.svg)

OpenAF's oJob GitHub action to run generic [OpenAF](https://docs.openaf.io) [oJobs](https://docs.openaf.io/docs/concepts/oJob.html).

## Usage

On a GitHub action step add the following entry:

````yaml
  - name: Executing an oJob
    uses: openaf/ojob-action@v2
    with:
      # the reference to a local oJob yaml/json file or a remote oJob.io
      ojob: '...' 
      # given the oJob referenced the key/value arguments to provide to it
      args: 'key1=value1 key2=value2 ...'
      # the public OpenAF distribution to use as runtime (e.g. nightly)
      # if no value is provided it defaults to the stable
      dist: '...'
````

After the first use, in a job, the installation of the OpenAF runtime is reused thus making additional steps using this action faster. It's also possible to cache the runtime and corresponding oPacks between job execution with [GitHub Actions caching](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#comparing-artifacts-and-dependency-caching) (saving, at least, around ~4/5 seconds of action execution) but you will have to be carefull with the management of the cache. Example:

````yaml
  - name: Cache OpenAF runtime
    uses: actions/cache@v3
    with:
      # you need to manage the cache in the best way specifically for your case
      key : oaf-nightly
      path: /tmp/oaf

  - name: Executing an oJob
    uses: openaf/ojob-action@v2
    with:
      ojob: '...' 
      args: 'key1=value1 key2=value2 ...'
      dist: 'nightly'
````

## Examples

*Example of a GitHub action to run an ojob.io job:*

````yaml
name: Get Envs
on: [push]

jobs:
  Get-Envs:
    runs-on: ubuntu-latest
    name   : Get env variables
    steps  :
    - uses: actions/checkout@v3

    - name: Retrieve env variables for testing
      uses: openaf/ojob-action@v2
      with:
        ojob: 'ojob.io/envs'
````

*Example of a GitHub action to run an ojob.io job with arguments:*

````yaml
name: Echo arguments
on: [push]

jobs:
  Echo-Arguments:
    runs-on: ubuntu-latest
    name   : Echo arguments
    steps  :
    - uses: actions/checkout@v3

    - name: Echo input args for testing
      uses: openaf/ojob-action@v2
      with:
        ojob: 'ojob.io/echo'
        args: 'abc=123 xyz=abc'
````

*Example of a GitHub action to security scan a container image and produce a badge:*

````yaml
name: Scan Images

on:
  push:
    branches: [ "master" ]
  schedule:
    - cron: '00 1 * * 6'

jobs:
  Scan-Images:
    runs-on    : ubuntu-latest
    permissions:
      contents: write
    name       : Scan images
    steps      :
    - uses: actions/checkout@v3

    - name: Scan some/image:latest
      uses: openaf/ojob-action@v2
      with:
        ojob: 'ojob.io/sec/genSecBadge'
        args: 'image=some/image:latest file=.github/sec-latest.svg'
        dist: 'nightly'

    - name: Scan some/image:build
      uses: openaf/ojob-action@v2
      with:
        ojob: 'ojob.io/sec/genSecBadge'
        args: 'image=some/image:build file=.github/sec-build.svg'
        dist: 'nightly'

    - name: Add the generated badges 
      run : |
        # user identification
        git config user.name github-actions
        git config user.email github-actions@github.com
        # only add/commit/push if new contents exist
        if [[ $(git status --porcelain) ]]; then
          git add .github/sec-latest.svg
          git add .github/sec-build.svg
          git commit -m "update badge"
          git push
        fi
````

*Example of a GitHub action to retrieve the installed version and distribution of OpenAF:*

````yaml
name: Get Version
on: [push]

jobs:
  Get-Version:
    runs-on: ubuntu-latest
    name: Get version
    steps:
    - uses: actions/checkout@v3

    - name: Generating getVersion.yaml oJob
      run : |
        cat <<EOF > getVersion.yaml
        todo:
        - check version
        
        jobs:
        - name: check version
          exec: |
            var data = { version: getVersion(), distribution: getDistribution() }
            ow.oJob.output(data, args)
        EOF

    - name: Running getVersion.yaml oJob
      uses: openaf/ojob-action@v2
      with:
        ojob: 'getVersion.yaml'
        dist: 'nightly'
````
