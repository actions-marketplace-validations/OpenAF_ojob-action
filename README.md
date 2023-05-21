# ojob-action
OpenAF's oJob GitHub action

## Usage

### Example of a GitHub action to run an ojob.io job:

````yaml
on: [push]

jobs:
  Test:
    runs-on: ubuntu-latest
    name: Get env variables
    steps:
    - uses: actions/checkout@v3
    - uses: nmaguiar/myOJobs@nightly
      with:
        ojob: 'ojob.io/envs'
````

### Example of a GitHub action to run an ojob.io job with arguments:

````yaml
on: [push]

jobs:
  Test:
    runs-on: ubuntu-latest
    name: Echo arguments
    steps:
    - uses: actions/checkout@v3
    - uses: nmaguiar/myOJobs@nightly
      with:
        ojob: 'ojob.io/echo'
        args: 'abc=123 xyz=abc'
````

### Example of a GitHub action to retrieve the installed version and distribution of OpenAF:

````yaml
on: [push]

jobs:
  Test:
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
