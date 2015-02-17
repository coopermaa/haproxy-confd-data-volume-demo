# haproxy-confd-data-volume-demo

This repo illustrates how to load balance web containers in CoreOS using HAProxy and confd.

Compared to [haproxy-conf-demo](https://github.com/coopermaa/haproxy-confd-demo.git), the differences are:

  * confd runs in its own container, writes configs to a shared data volume
  * HAProxy runs in its own container, reading config from the shared data volume
  * both containers constrained to run on the same machine
  * when confd updates the config, it will send a HUP signal to HAProxy. The /var/run/docker.sock is shared with
    the confd container so that it can use the docker API to do that.

## Quickstart

### Clone this repo

```
$ git clone https://github.com/coopermaa/haproxy-confd-data-volume-demo.git
$ cd haproxy-conf-data-volume-demo
```

### Deploy web backend containers

Create two instances of nginx services and sidekick containers. These two nginx services will serve as our web backend:

```
$ fleetctl start nginx@1.service
$ fleetctl start nginx-discovery@1.service
$ fleetctl start nginx@2.service
$ fleetctl start nginx-discovery@2.service
```

### Deploy the confd container:

```
$ fleetctl start confd.service
```

### Deploy the load balancer container

```
$ fleetctl start haproxy.service
```

This will run HAProxy service at port 80 and port 1936 for statistics.

### Special Note

The HAProxy service must be the last one to be deployed. If we deploy haproxy.service before confd.service, it will
 wait until confd.service the the command prompt will not return immediately.

$ fleetctl list-units
UNIT                            MACHINE                         ACTIVE          SUB
confd.service                   f564b0fc.../172.17.8.101        activating      start-pre

If the confd.service is activating and start-pre it means that docker pull from the `ExecStartPre` is still running,
 you have to wait until it's running then start the haproxy.service.

### Test the application

Now, our web backend is load balanced. We can visit them via the load balancer.
First, find out where our haproxy.service is deployed by running `fleetctl list-units`:

```
$ fleetctl list-units
UNIT                            MACHINE                         ACTIVE  SUB
confd.service                   beb76d18.../172.17.8.102        active  running
haproxy.service                 beb76d18.../172.17.8.102        active  running
nginx-discovery@1.service       f315121c.../172.17.8.103        active  running
nginx-discovery@2.service       f564b0fc.../172.17.8.101        active  running
nginx@1.service                 f315121c.../172.17.8.103        active  running
nginx@2.service                 f564b0fc.../172.17.8.101        active  running
```

Take a note of the IP Address of the machine where haproxy is deployed. In my case, it's '172.17.8.102'. 
Now make some requests:

```
$ for i in {1..100}; do curl -s http://172.17.8.102 > /dev/null; done
```

and visit http://172.17.8.102:1936 to view the HAProxy statistics.