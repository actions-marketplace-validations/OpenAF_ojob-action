# ojob-action

![version](.github/ojobs/version.svg) [![Tests](https://github.com/OpenAF/ojob-action/actions/workflows/tests.yml/badge.svg)](https://github.com/OpenAF/ojob-action/actions/workflows/tests.yml)

OpenAF's oJob GitHub action to run generic [OpenAF](https://docs.openaf.io) [oJobs](https://docs.openaf.io/docs/concepts/oJob.html).

## Basic usage

On a GitHub action step add the following entry:

````yaml
  - name: Executing an oJob
    uses: openaf/ojob-action@v7
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
    uses: actions/cache@v4
    with:
      # you need to manage the cache in the best way specifically for your case
      key : oaf-nightly
      path: /tmp/oaf

  - name: Executing an oJob
    uses: openaf/ojob-action@v7
    with:
      ojob: '...' 
      args: 'key1=value1 key2=value2 ...'
      dist: 'nightly'
      pwd : '/some/dir'
````

### Embedding an oJob definition

It's possible to add the oJob YAML definition directly as the action parameters:

````yaml
  - name: Executing Hello World
    uses: openaf/ojob-action@v7
    with:
      def : |
        todo:
        - Say hello
        
        jobs:
        - name : Say hello
          check:
            in:
              name: isString.default("someone")
          exec : |
            tprint("Hi {{name}}!", args)
            
      args: "name=Scott"
````

### Executing an OpenAF script

If you have a local OpenAF script you can also run directly. 

````yaml
  - name: Executing hello.js
    uses: openaf/ojob-action@v7
    with:
      oaf : scripts/hello.js
      args: "name=Tiger"
````

### Embedding an OpenAF script

If necessary, it's also possible to embeed a script directly with _script_:

````yaml
  - name: Executing hello.js
    uses: openaf/ojob-action@v7
    with:
      script: |
        var params = processExpr(" ")
        
        if (isDef(params.name)) tprint("Hi {{name}}!", params)
        
      args  : "name=Tiger"
````

## Examples

Checkout also the [catalog of uses](USES.md).

*Example of a GitHub action to run an ojob.io job:*

````yaml
name: Get Envs
on:
  workflow_dispatch:
  push:

jobs:
  Get-Envs:
    runs-on: ubuntu-latest
    name   : Get env variables
    steps  :
    - uses: actions/checkout@v4

    - name: Retrieve env variables for testing
      uses: openaf/ojob-action@v7
      with:
        ojob: 'ojob.io/envs'
````

*Example of a GitHub action to run an ojob.io job with arguments:*

````yaml
name: Echo arguments
on:
  workflow_dispatch:
  push:

jobs:
  Echo-Arguments:
    runs-on: ubuntu-latest
    name   : Echo arguments
    steps  :
    - uses: actions/checkout@v4

    - name: Echo input args for testing
      uses: openaf/ojob-action@v7
      with:
        ojob: 'ojob.io/echo'
        args: 'abc=123 xyz=abc'
````

*Example of a GitHub action to security scan a container image and produce a badge:*

````yaml
name: Scan Images

on:
  workflow_dispatch:
  push:
    branches: [ "master" ]
  schedule:
    - cron: '00 1 * * 6'

jobs:
  Scan-Images:
    runs-on    : ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      
    name       : Scan images
    steps      :
    - uses: actions/checkout@v4

    - name: Scan some/image:latest
      uses: openaf/ojob-action@v7
      with:
        ojob: 'ojob.io/sec/genSecBadge'
        args: 'image=some/image:latest file=.github/sec-latest.svg'
        dist: 'nightly'

    - name: Scan some/image:build
      uses: openaf/ojob-action@v7
      with:
        ojob: 'ojob.io/sec/genSecBadge'
        args: 'image=some/image:build file=.github/sec-build.svg'
        dist: 'nightly'

    - name: Add the generated badges 
      uses: openaf/ojob-action@v7
      env :
        GH_TOKEN: ${{ github.token }}
      with:
        ojob: 'ojob.io/git/githubCheckIn'
        args: 'message="Badges\ update"'
        dist: 'nightly'
````

*Example of a GitHub action to retrieve the installed version and distribution of OpenAF:*

````yaml
name: Get Version
on:
  workflow_dispatch:
  push:

jobs:
  Get-Version:
    runs-on: ubuntu-latest
    name: Get version
    steps:
    - uses: actions/checkout@v4

    - name: Running get version
      uses: openaf/ojob-action@v7
      with:
        def : |
          todo:
          - check version

          jobs:
          - name: check version
            exec: |
              var data = { version: getVersion(), distribution: getDistribution() }
              ow.oJob.output(data, args)
        dist: nightly
````
