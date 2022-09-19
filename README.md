# traefik

How To Use Traefik v2 as a Reverse Proxy for Docker Containers on Ubuntu 20.04

https://www.digitalocean.com/community/tutorials/how-to-use-traefik-v2-as-a-reverse-proxy-for-docker-containers-on-ubuntu-20-04

RUN WITH:

    docker run -d --restart=always \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v $PWD/traefik.toml:/traefik.toml \
      -v $PWD/traefik_dynamic.toml:/traefik_dynamic.toml \
      -v $PWD/acme.json:/acme.json \
      -p 80:80 \
      -p 443:443 \
      --network web \
      --name traefik2022 \
      traefik:v2.2
