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
(TODO check why needed?!?!
docker swarm init --advertise-addr 192.168.99.100
)
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

(docker exec -it <c-id> bash)
docker exec -it
TODO test if nano is installed in the image
cat /var/jenkins_home/secrets/initialAdminPassword

to stop it:
docker service rm jenkins-master


sources:

pipeline:
https://jenkins.io/doc/book/pipeline/syntax/

jenkins-config:
https://antonfisher.com/posts/2017/01/16/run-jenkins-in-docker-container-with-persistent-configuration/
https://gist.github.com/cenkalti/5089392

backround info:
https://engineering.riotgames.com/news/docker-jenkins-data-persists
https://github.com/dockersamples/docker-swarm-visualizer
http://jpetazzo.github.io/orchestration-workshop/#1
http://www.catosplace.net/blog/2015/02/11/running-jenkins-in-docker-containers/
https://forums.docker.com/t/docker-swarm-mode-and-local-images/22894/4