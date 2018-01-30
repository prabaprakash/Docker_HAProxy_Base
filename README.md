Haproxy for Docker Containers  

Steps:

1. Create Isolated Network for docker
`docker network create isolated_network`
2. for every container docker compose file, add the network configuration to use the `isolated_network`
```
networks:
  default:
      Use a custom driver
     external:
      name: isolated_network
```
3. for ex: containers for nginx  running on port 8080
         get the container name : `docker_container_a:8080`

4. add the subdomain config & container config in haproxy config for redirect subdomain route to `docker_container_a:8080`

```
defaults
  mode http  
  timeout connect 5s
  timeout client 120s
  timeout server 120s
frontend mywebsite
  bind *:80
  acl host_sub2 hdr(host) -i subdomain.domain.com
  use_backend sub2_nodes if host_sub2
default_backend webservers
  backend webservers
  balance roundrobin
  server web1 172.26.236.66:4445 check
  server web2 172.26.236.66:4445 check

backend sub2_nodes
  mode http
  balance roundrobin
  stick-table type ip size 200k expire 30m
  stick on src
  server node1 docker_container_a:8080 check
```
    
