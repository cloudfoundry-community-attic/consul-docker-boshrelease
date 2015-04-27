Run Consul with BOSH using Docker image
=======================================

This BOSH release contains a pre-baked Docker image for running Consul in any datacenter even if it has no internet connectivity, and will make all your security people happy. It requires no external internet like normal Docker use cases.

The repository is also a demonstration for how to package a public Docker image into a BOSH release to remove internet dependencies.

Usage
-----

To use this BOSH release, first upload it to your bosh and the `docker` release

```
bosh upload release https://bosh.io/releases/cloudfoundry-community/consul-docker
bosh upload release https://bosh.io/d/github.com/cf-platform-eng/docker-boshrelease
```

For [bosh-lite](https://github.com/cloudfoundry/bosh-lite), you can quickly create a deployment manifest & deploy a cluster:

```
templates/make_manifest warden
bosh -n deploy
```

For AWS EC2, create a single VM:

```
templates/make_manifest aws-ec2
bosh -n deploy
```

### Manual manifest setup

To include this BOSH release in a Docker VM managed by BOSH you need the following job templates in the following order:

```yaml
jobs:
- name: consul_docker_z1
  templates:
    # run docker daemon
    - {name: docker, release: docker}
    # warm docker image cache from bosh package
    - {name: consul_docker_image, release: consul-docker}
    # run containers (see properties.containers)
    - {name: containers, release: docker}
```

You will need both `consul-docker` and `docker` releases:

```yaml
releases:
- name: docker
  version: latest
- name: consul-docker
  version: latest
```

### Override security groups

Ensure that the security group (`consul` by default, see `templates/stub.yml`) opens the ports you need for clients to connect, such as 8500 and 8600.
