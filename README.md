
<!-- README.md is generated from README.Rmd. Please edit that file -->
Setting Up Docker-Compose for ShinyProxy
========================================

How to use
----------

**Download a template**:

``` bash
# Download a template
git clone git@github.com:kassambara/shinyproxy-config.git
cd shinyproxy-config/docker-compose-example
```

**Template folder structure**:

    docker-compose-example
    ├── docker-compose.yml
    ├── shinyapps
    │   └── euler-docker
    │       ├── Dockerfile
    │       ├── Rprofile.site
    │       └── euler
    │           ├── server.R
    │           └── ui.R
    └── shinyproxy
        ├── Dockerfile
        └── application.yml

**Content of the docker-compose.yml**:

``` yaml
version: "3.6"
services:
  shinyproxy:
    image: datanovia/shinyproxy
    container_name: dnv_shinyproxy
    restart: on-failure
    build: ./shinyproxy
    networks:
      - dnv-net
    ports:
      - 8080:8080
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./shinyproxy-logs/server:/log
      - ./shinyproxy-logs/container:/container-logs
      - ./shinyproxy/application.yml:/opt/shinyproxy/application.yml
  euler:
    image: euler-docker
    container_name: dnv_euler
    build: ./shinyapps/euler-docker
    networks:
      - dnv-net

networks:
  dnv-net:
    name: dnv-net
```

**Content of the ShinyProxy application.yaml**:

``` yaml
proxy:
  title: ShinyProxy
  port: 8080
  authentication: simple
  admin-groups: admins
  users:
  - name: jack
    password: password
    groups: admins
  - name: jeff
    password: password
  docker:
      internal-networking: true
      container-network: dnv-net
  # change this route to your work directory, use absolute path
  # work-directory: /Users/kassambara/Documents/shinyproxy-docker-compose-example
  specs:
  - id: 01_hello
    display-name: Hello Application
    description: Application which demonstrates the basics of a Shiny app
    container-cmd: ["R", "-e", "shinyproxy::run_01_hello()"]
    container-image: openanalytics/shinyproxy-demo
    container-network: "${proxy.docker.container-network}"
    # container-volumes: ["${proxy.work-directory}/apps:/apps"]
  - id: euler
    display-name: Euler's number
    container-cmd: ["R", "-e", "shiny::runApp('/root/euler')"]
    container-image: euler-docker
    container-network: "${proxy.docker.container-network}"

logging:
  file:
    /log/shinyproxy.log
```

Docker uses `networks` to enable communication between containers. For ShinyProxy to communicate properly with the Shiny App, the network specified in `docker-compose.yml` must be the same as the same as that listed in `application.yml`. (Tip, if you’re using a docker-compose file to launch the app, don’t set up the docker network manually, see [here](https://stackoverflow.com/a/54471854/8675075) for why).

Each container listed in `services`: is named in container\_name, with the Dockerfile being pointed to via. the `context` and `Dockerfile` variables. If the container was terminated with an error code, the `restart` line instructs Docker what action is to be taken . The `networks` variables lists the Docker network that the container has access to.

The network used by all applications in this `docker-compose` must match the network specified in the ShinyProxy `application.yml`.

### Building and running

``` bash
# Go into your project folder
cd shinyproxy-config/docker-compose-example
# Build all docker images
docker-compose build
# Run shinyproxy
docker-compose up -d shinyproxy
```

Visit: <http://localhost:8080>

To stop shinyproxy use `docker-compose down`.

### Other Examples of configuration

-   [Telethon Kids Institute ShinyProxy Docker Template](https://github.com/TelethonKids/deploy_shiny_app): config includes shinyproxy + nginx + influxdb + cerbot
