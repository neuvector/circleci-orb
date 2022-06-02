# NeuVector Orb for CircleCI
**CircleCI Orb Registry:** https://circleci.com/orbs/registry/orb/neuvector/neuvector-orb

This orb provides NeuVector vulnerability scanning to your CircleCI workflows.

NeuVector supports the local image scan and the registry scan.

## Setup:

### 1. Create a context in your CircleCI app.

![Set context](images/context.png?raw=true)

### 2. Add environment variables `controller_ip`, `controller_port`, `controller_username`, `controller_password`, `nv_registry_username`, `nv_registry_password` to the context.

![Set env](images/env.png?raw=true)

### 3. Add the NeuVector orb to your current build config.yml

```
orbs:
  neuvector: neuvector/neuvector-orb@x.y.z
  (x.y.z is the orb version number.)
```

Add the job "neuvector/scan-image" to the circleCI workflow.

Usage examples:

#### a. Scan the image at a public registry

The registry_url is url of the public registry.

The job "neuvector/scan-image" will fail when the number of high or medium vulnerability found in your image exceeds the criteria.

```
version: 2.1
orbs:
  neuvector: neuvector/neuvector-orb@1.1.0
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

#### b. Scan the image at the private registry

Add variables "registry_username" and "registry_password" to the project
![Set env](images/env2.png?raw=true)

The registry_url is the url of the private registry. 

The registry_username is the login user of your private registry. 

The registry_password is the login password of your private registry.

The job "neuvector/scan-image" will fail when the number of high or medium vulnerability found in your image exceeds the criteria.

```
version: 2.1
orbs:
  neuvector: neuvector/neuvector-orb@1.1.0
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

#### c. Scan the local image built in the CircleCI job "build_image"

To scan the image on the same host, you need to set scan_local_image as true.

The image needs to be saved as a tar archive file and set the image_tar_file.

The path is the directory where the tar archive file is stored.

The image_name is the name of the to-be-scanned image.

The image_tag is the tag of the to-be-scanned image.

```
version: 2.1
orbs:
  neuvector: neuvector/neuvector-orb@1.1.0
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

## Sample Results:

### 1. Passes the criteria configured

![Pass criteria](images/pass.png?raw=true)

### 2. Fails the criteria configured

![Fail criteria](images/fail.png?raw=true)
