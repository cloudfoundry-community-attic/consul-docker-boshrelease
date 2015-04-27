Run Consul with BOSH using Docker image
=======================================

This BOSH release contains a pre-baked Docker image for running Consul in any datacenter even if it has no internet connectivity, and will make all your security people happy. It requires no external internet like normal Docker use cases.

The repository is also a demonstration for how to package a public Docker image into a BOSH release to remove internet dependencies.

Currently v1 includes consul v0.5.0 via the `progrium/consul:latest` image from April 26, 2015. See below for how to bump to newer version of this Docker image.

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

Development
-----------

To upgrade to a newer version of `progrium/consul` Docker image, run:

```
docker pull progrium/consul
docker save progrium/consul \
  > blobs/docker-images/progrium-consul.tgz
```

You can now create development releases to test it. To include the new docker image in a new BOSH final release:

```
bosh upload blobs
git add .
git commit -m "update progrium/consul image"
bosh create release --final
```
