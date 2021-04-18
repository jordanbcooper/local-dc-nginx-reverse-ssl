# local-dc-nginx-reverse-ssl

based on https://github.com/akullpp/multiple-react-nginx, but with SSL and reverse proxy.


Follow ssl.md instructions first

`docker network create local` (or whatever you want to call it, just change the network in the docker-compose.yml)

configure gateway/nginx.conf and Dockerfile for your own code and routes (just copy existing server and update for your app, also modify main docker-compose.yml)

Once all configured, docker-compose up --build in main directory.


Add 

127.0.0.1 domain.com subdomain.domain.com

to your hosts file
