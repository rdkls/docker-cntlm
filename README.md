# Cntlm Docker

rdkls fork from btrepp

This is a simple dockerfile that wraps up cntlm into debian. It will set up a open proxy, that uses NTLMv2 to
authenticate upstream. The main use case is to get a simple proxy setup that plays nice with windows networks.

Ideally you can use this with redsocks+iptables, to make docker images think they have direct access, but the connections are
actually tunneled through.

container->redsocks->ctnlm->upstream

For SSH you may also be interested in [corkscrew](https://github.com/bryanpkc/corkscrew)

## Usage

Simple usage is 

`docker run --rm -p 3128:3128 rdkls/cntlm user.name domain NTVLMv2Hash upstream_proxy:port`

This will create an unathenticated proxy on the host running at 3128

By default cntlm will run with NoProxy localhost, 127.0.0.*, 10.*, 192.168.*, *.$2
If the env var NO_PROXY is set, it gets added to this list

## Generate NTLMv2 Hash

By default I've only set this up to accept Hashed passwords. cntlm supports actual passwords, but 
that is left up to you to figure out.

### With get_ntlm.sh

A script exists in the images that will help you get the hash

`docker run --rm -t -i --entrypoint="get_ntlm.sh" rdkls/cntlm user.name@domain upstream_proxy:port`

This will ask you for your password and attempt to use the upstream proxy to get to docker.io. If it
succeeds it will print a PassNTLMv2 line, use this hash above when launching the container.

### With cntlm directly

```
~/w/docker-cntlm   +  ***docker run --rm -ti --entrypoint /usr/sbin/cntlm rdkls/cntlm -u MY_USERNAME -d MY_DOMAIN -H***
cntlm: Starting cntlm version 0.92.3 for LITTLE endian

cntlm: Proxy listening on 127.0.0.1:3128

cntlm: Workstation name used: 1ed7f958a5d0

cntlm: Using following NTLM hashes: NTLMv2(1) NT(0) LM(0)

Password: ****[You get prompted for this]******
PassLM          EABA7A3C3633DE64015F4738968A4B57
PassNT          EABA7A3C3633DE64015F4738968A4B57
PassNTLMv2      EABA7A3C3633DE64015F4738968A4B57    # Only for user 'MY_USER', domain 'MY_DOMAIN'
cntlm: Terminating with 0 active threads
```

You want the 'PassNTLMv2' hash

## Contributing

This was a quick hack to get together as nothing currently existed on the registry. Feel free to fork it and send pull requests
