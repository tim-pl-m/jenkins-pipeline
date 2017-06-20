# jenkins-pipeline
ec2 without ecs

setup: win8.1, docker-tool-box, babun on windows/macOS
prerequisites: an aws-node with docker installed on the node and a running registry on it
(created with docker-machine, node should be running)

```shell
# connect to the node:
docker-machine ssh <your node>
# get admin
sudo su
# add a visualizer
# TODO evtl. clone not needed
git clone git://github.com/dockersamples/docker-swarm-visualizer
cd docker-swarm-visualizer
# create a swarm
docker swarm init
docker service create \
  --name=viz \
  --publish=8081:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer

docker service list

docker service create \
       --name jenkins-master \
       --mount type=bind,target=/var/run/docker.sock,src=/var/run/docker.sock \
       --publish 50000:50000 \
       --publish 8080:8080 \
       timplm/jenkins-docker

# if a plain jenkins:
docker exec -it <c-id> bash
nano /var/jenkins_home/secrets/initialAdminPassword

# default from docker-hub is docker:jenkins


```
wip:
init a swarm: wip
# install docker-compose
apt install docker-compose -y
docker-compose up -d

sources:
https://github.com/dockersamples/docker-swarm-visualizer
http://jpetazzo.github.io/orchestration-workshop/#1
http://www.catosplace.net/blog/2015/02/11/running-jenkins-in-docker-containers/
https://forums.docker.com/t/docker-swarm-mode-and-local-images/22894/4