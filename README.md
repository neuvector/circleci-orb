# NeuVector Orb for CircleCI

This orb provides NeuVector vulnerability scanning to your CircleCI workflows.

Currently support scanning public registry images with a running NeuVector Allinone or Controller accessible by CircleCi.

## Setup:

### 1. Create a context in your CircleCI app.

![Set context](images/context.png?raw=true)

### 2. Add environment variables for `controller_ip`, `controller_port`, `controller_username`, `controller_password` to your new context.

![Set env](images/env.png?raw=true)

### 3. Add the NeuVector orb to your current build config.yml

```
orbs:
  neuvector: neuvector/neuvector-orb@1.0.0
```

Add neuvector/scan-image with parameters to your current workflow.

The registry_url is a public registry where the image is saved.

Set up your vulnerability criteria to fail the build. It will fail if the number of high or medium vulnerability found in your image exceeds your criteria.

Usage example:

```
version: 2.1
orbs:
  neuvector: neuvector/neuvector-orb@1.0.0
workflows:
  scan-image:
    jobs:
      - neuvector/scan-image:
          context: myContext
          registry_url: https://registry.hub.docker.com
          repository: alpine
          tag: "3.4"
          scan_layers: false
          high_vul_to_fail: 0
          medium_vul_to_fail: 3
```

## Result:

### 1. Pass the criteria you set

![Pass criteria](images/pass.png?raw=true)

### 2. Fail the criteria you set

![Fail criteria](images/fail.png?raw=true)

