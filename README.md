# local-dc-nginx-reverse-ssl

based on https://github.com/akullpp/multiple-react-nginx, but with SSL and reverse proxy.


Follow ssl.md instructions first

configure gateway/nginx.conf and Dockerfile for your own code and routes (just copy existing server and update for your app, also modify main docker-compose.yml)

Once all configured, docker-compose up --build in main directory.
