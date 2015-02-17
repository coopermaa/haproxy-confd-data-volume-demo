# docker-confd

Dockerfile for confd. This repo illustrates how to load balance web containers in CoreOS using HAProxy and confd.

## To Build image

```
$ docker build -t <user name>/confd .
```

## Usage

You must pass the etcd peer address with the -e flag in docker run so that confd can connect to your etcd cluster:

```
$ docker run --name confd.service -e "ETCD=http://${COREOS_PRIVATE_IPV4}:4001" \
  -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) \
  coopermaa/confd'
```

When confd updates the config, it will send a HUP signal to HAProxy. The /var/run/docker.sock is shared with
 the confd container so that it can use the docker API to do that.

By default, the container will write configs to /usr/local/etc/haproxy/haproxy.cfg, you can use a shared data volume like so:

```
$ docker run --name confd.service -e "ETCD=http://${COREOS_PRIVATE_IPV4}:4001" \
  -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) \
  -v /usr/local/etc/haproxy  coopermaa/confd'
$ docker run --name %n -p 80:80 -p 1936:1936 --volumes-from confd.service haproxy:1.5.10
```

## Demo

See [haproxy-confd-data-volume-demo](https://github.com/coopermaa/haproxy-confd-data-volume-demo) for a demonstration.