# NeuVector Orb for CircleCI
**CircleCI Orb Registry:** https://circleci.com/orbs/registry/orb/neuvector/neuvector-orb

This orb provides NeuVector vulnerability scanning to your CircleCI workflows.

NeuVector supports scanning images locally and/or from a registry.

## Setup:

### 1. Create a context in your CircleCI app.

![Set context](images/context.png?raw=true)

### 2. Add environment variables `controller_ip`, `controller_port`, `controller_username`, `controller_password`, `nv_license`, `nv_registry_username`, `nv_registry_password` to the context.

![Set env](images/env.png?raw=true)

### 3. Add the NeuVector orb to your current build config.yml

```
orbs:
  neuvector: neuvector/neuvector-orb@x.y.z
(Where x.y.z is the current orb versio)
```

Add neuvector/scan-image with parameters to your current workflow.

Usage examples:

#### a. Scan an image from a public registry

The registry_url is the public registry where the image is stored.

(Optional) Set up your vulnerability criteria to fail the build. 

It will fail if the number of high or medium vulnerability found in your image exceeds the criteria.

```
version: 2.1
orbs:
  neuvector: neuvector/neuvector-orb@1.0.2
workflows:
  scan-image:
    jobs:
      - neuvector/scan-image:
          context: myContext
          registry_url: https://registry.hub.docker.com
          repository: alpine
          tag: "3.12"
          scan_layers: false
          high_vul_to_fail: 0
          medium_vul_to_fail: 3
```

#### b. Scan an image from a private registry

Add variables "registry_username" and "registry_password" to the project
![Set env](images/env2.png?raw=true)

The registry_url is your private registry. 

The registry_username is the login user of your private registry. 

The registry_password is the login password of your private registry.

(Optional) Set up your vulnerability criteria to fail the build. 

It will fail if the number of high or medium vulnerability found in your image exceeds the criteria.

```
version: 2.1
orbs:
  neuvector: neuvector/neuvector-orb@1.0.2
workflows:
  scan-image:
    jobs:
      - neuvector/scan-image:
          context: myContext
          registry_url: 127.100.12.157:5000
          registry_username: ${registry_username}
          registry_password: ${registry_password}
          repository: ci_demo_image
          tag: "1.2"
          scan_layers: false
          high_vul_to_fail: 0
          medium_vul_to_fail: 3
```

#### c. Scan an image from a CircleCI build job

The boolean parameter scan_local_image is an indicator to scan the image on the same host. Set it to true.

The image_tar_file is the tar archive where the image to be scanned is stored.

The path is the directory where the tar archive file is stored.

The image_name is the name of the image to be scanned.

The image_tag is the tag name of the image to be scanned.

```
version: 2.1
orbs:
  neuvector: neuvector/neuvector-orb@1.0.2
workflows:
  scan-image:
    jobs:
      - build_image
      - neuvector/scan-image:
          requires:
            - build_image
          context: myContext
          scan_local_image: true
          image_tar_file: alpine-3.2.tar
          path: /tmp/neuvector/
          image_name: alpine
          image_tag: "3.2"
          scan_layers: false
          high_vul_to_fail: 0
          medium_vul_to_fail: 3
```

Here is a sample build job to scan the image alpine:3.12

```
jobs:
  build_image:
    docker:
      - image: docker:stable-git
    steps:
      - setup_remote_docker
      - checkout
      - run:
          name: build container
          command: |
            docker pull alpine:3.12
      - run:
          name: Save Docker image
          command: |
            rm -rf /tmp/neuvector/
            mkdir /tmp/neuvector/ -p
            docker save -o /tmp/neuvector/alpine-3.12.tar alpine:3.2
      - persist_to_workspace:
          root: /tmp/neuvector/
          paths:
            - ./
```

## Result:

### 1. Pass the criteria you set

![Pass criteria](images/pass.png?raw=true)

### 2. Fail the criteria you set

![Fail criteria](images/fail.png?raw=true)
