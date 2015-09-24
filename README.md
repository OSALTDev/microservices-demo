# microservices-demo
A very simple Go-Redis app to demo discovery of multiple services behind a haproxy load balanced (using the interlock plugin system)

### Pre-requisites
1. Ensure Docker Swarm is working. An effective demo would require at least 3 active nodes in the swarm.
2. Docker version > 1.8
3. Set the `DOCKER_HOST` environment variable to the Docker Swarm's tcp endpoint. Do not use localhost, even if you are on the Docker Swarm manager / master. Example: `export DOCKER_HOST=tcp://10.0.0.6:9999`.
4. `docker info` should show the nodes added to the cluster.

### Steps
1. On any node that is part of the cluster, run a container from the ehazlett/interlock image using the command below. We want to use the haproxy plugin for this lab.
  ```
  docker run -p 80:80 --name lb0 -d ehazlett/interlock --swarm-url $DOCKER_HOST --plugin haproxy start
  ```
2. Due to the cluster scheduling, the haproxy container may actually be running on a different host than the one where the above command was run. Use `docker ps | grep lb0` to identify the host it is running on.
  - Alternatively, specify a filter (ie., affinity:nodename or constraint:container) to restrict the container to a specific docker host.
3. Clone this repo to a local folder. `git clone https://github.com/anokun7/microservices-demo.git`
4. `cd microservices-demo`
5. Use docker-compose to build and run the web app containers. `docker-compose up -d`

  ```
  vagrant@ubuntu5:/vagrant/microservices-demo$ docker-compose up -d 
  Creating microservicesdemo_dbdata_1...
  Creating microservicesdemo_db_1...
  Creating microservicesdemo_web_1...
  ```
6. Every container started in a swarm cluster gets registered to the ha-proxy as a backend as long as the container has an exposed port and a hostname.
  - The hostname for the `web` container is configured in the `docker-compose.yml` using the `INTERLOCK_DATA` environment variable.
7. Let's again use docker-compose to scale up the number of web containers to 9. Each of these 9 web containers will also get registered to the same backend in the HA Proxy config.
 
  ```
  vagrant@ubuntu5:/vagrant/microservices-demo$ docker-compose scale web=9
  Creating microservicesdemo_web_2...
  Creating microservicesdemo_web_3...
  Creating microservicesdemo_web_4...
  Creating microservicesdemo_web_5...
  Creating microservicesdemo_web_6...
  Creating microservicesdemo_web_7...
  Creating microservicesdemo_web_8...
  Creating microservicesdemo_web_9...
  Starting microservicesdemo_web_2...
  Starting microservicesdemo_web_3...
  Starting microservicesdemo_web_4...
  Starting microservicesdemo_web_5...
  Starting microservicesdemo_web_6...
  Starting microservicesdemo_web_7...
  Starting microservicesdemo_web_8...
  Starting microservicesdemo_web_9...
  ```
8. Ensure DNS is setup (or add entries to `/etc/hosts` file) to resolve the host where the `lb0` container is running.
9. Browse to the URL: `http://[host-ip-running-lb0]/demo`
10. Every time a container responds to the HTTP request, it should get its counter incremented. The counter is being stored (and retrieved) from a REDIS backend database.
