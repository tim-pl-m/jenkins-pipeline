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
git clone git://github.com/dockersamples/docker-swarm-visualizer
cd docker-swarm-visualizer
# create a swarm
docker swarm init
docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer

```
wip:
init a swarm: wip
# install docker-compose
apt install docker-compose -y
docker-compose up -d

sources:
https://github.com/dockersamples/docker-swarm-visualizer
http://jpetazzo.github.io/orchestration-workshop/#1