# Setting Up a Secure Development Proxy

Following this guide will give you a nice local development environment with a 
reverse proxy that will forward traffic to locally running Docker containers. 
The proxy supports HTTPS through self-signed certificates, which are added to 
the trust store, so browsers will trust them.

Custom domain names are supported and resolve automatically to the reverse proxy 
by using `dnsmasq`.

This guide has only been tested on OS X. The tools used below are also available 
on Linux, so theoretically this guide should work on Linux too, with some minor 
modifications.

## Basic steps

**Install dnsmasq**

```
$ brew install dnsmasq
```

**Configure and start dnsmasq**

```
$ echo "address=/.test/127.0.0.1" | sudo tee -a $(brew --prefix)/etc/dnsmasq.conf
$ sudo mkdir -p /etc/resolver
$ echo 'nameserver 127.0.0.1' | sudo tee /etc/resolver/test
$ # doesn't work without sudo
$ sudo brew services restart dnsmasq
```

**Install mkcert**

```
$ brew install mkcert
```

**Install a root certificate**

We'll install our own certificate authority and add it to the system trust 
store and the trust stores of installed browsers (where necessary).

```
$ mkcert -install
```

Any certificates we create, use the CA we've just installed, so all certificates 
will be trusted by the browser and other tools.

**Creating certificates**

You can now create certificates with `mkcert` or by using the `mk-cert` script 
located in the `bin` directory. The latter has a couple of advantages:

- It will rename the generated certificates to the right format for nginx-proxy (see below)
- It will put the generated certificates in the `certs` directory, which is 
mounted as a volume for nginx-proxy, which will automatically pick up any new certificates 
you create

`mk-cert` can be called from any directory, so I recommend you add it to your 
`$PATH` for ease of use.

**Start the proxy**

```
$ docker-compose up -d
```

This will start [nginx-proxy](https://github.com/jwilder/nginx-proxy). You 
can automatically proxy traffic for any domain you want to any Docker 
container you start by setting the `VIRTUAL_HOST` environment variable on 
that container to the domain you want it to respond to.

nginx-proxy will run on its own network, called `proxy`, so you'll have to 
start your containers on that network as well.

## Example usage

The `docker-compose-sample-services.yml` will start 2 Apache web servers 
which serve the default web page. They will be connected to the `proxy` 
network. They have the following `VIRTUAL_HOST` vars defined: 
`server1.test` and `server2.test`.

**Create certificates for the example**

```
$ bin/mk-cert server1.test
$ bin/mk-cert server2.test
```

**Start the Apache containers**

```
$ docker-compose -f docker-compose-sample.services.yml up
```

You now should be able to browse to https://server1.test and https://server2.test


