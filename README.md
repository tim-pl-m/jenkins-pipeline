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

# default login from docker-hub(timplm/jenkins-docker) is docker:jenkins

# problem: data cant be push and pulled
# solution: persistent data in volume?

# v1
# create a data-volume for the config
docker create -v /var/jenkins_home --name jenkins-dv jenkins
# bind the volume-container to the jenkins-container
docker run -d -p 8080:8080 --volumes-from jenkins-dv --name jenkins-master jenkins

docker exec -it jenkins bash
# now u can stop and start the jenkins
docker stop jenkins
docker run -d -p 8080:8080 --volumes-from jenkins-dv --name jenkins jenkins

# v2
docker build -t jenkinsdata .
docker run --name=jenkins-data jenkinsdata

docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master --volumes-from=jenkins-data -d jenkins
docker exec -it jenkins-master bash
nano /var/jenkins_home/secrets/initialAdminPassword

# v3
# backup your jenkins_home
docker exec -it jenkins-master bash

cd /tmp
git clone https://github.com/tim-pl-m/jenkins-pipeline.git
cp -r /var/jenkins_home/ /tmp/jenkins-pipeline/
cd jenkins-pipeline
RUN git config --global user.email "jenkins@container" &&\
    git config --global user.name  "jenkins"
git add .
git commit -m "config change"
<login to your github>
# backup done
# now stop and remove the image
wip

# and restore your config
# with dockerfile
git clone https://github.com/tim-pl-m/jenkins-pipeline.git
cd jenkins-pipeline/jenkins-image/
docker build -t jenkinsconfig .
# docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master -d jenkinsconfig
docker run \
    -d \
    --name jenkins-master \
    -p 8080:8080 \
    -p 50000:50000 \
    -v $(which docker):/usr/bin/docker \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /root/workspace \
    jenkinsconfig &&\
    docker logs -f jenkins-master;

# manually
docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master1 -d jenkins
docker exec -it jenkins-master1 bash
git clone https://github.com/tim-pl-m/jenkins-pipeline.git
cd /tmp
git clone https://github.com/tim-pl-m/jenkins-pipeline.git
cp -r /tmp/jenkins-pipeline/jenkins_home /var/jenkins_home/
# TODO fix

# TODO obsolete?
# push it to lokal registry
docker tag jenkins localhost:5000/jenkins
curl 127.0.0.1:5000/v2/_catalog
docker push localhost:5000/jenkins
# push it to docker-hub
docker tag jenkins timplm/jenkins-docker:addedUser
docker push timplm/jenkins-docker:addedUser

# v4
# log on your node
git clone https://github.com/tim-pl-m/jenkins-in-docker.git
cd jenkins-in-docker
cd keys
ssh-keygen -t rsa -b 4096 -C "my@gmail.com"
# use as file: jenkins.config.id_rsa
# press enter 2 times
# create an empty github repo for the config i.e.
# Add created key to GitHub (repository → Settings → Deploy keys) with write access
vi jenkins.config.id_rsa.pub

ssh-keygen -t rsa -b 4096 -C "my@gmail.com"
# jenkins.application.id_rsa
# add it to app-repo
# wip

cd ..
cd images/master
bin/image-build.sh
# a new image "jenkins-master" was created
docker images
# run it
bin/container-run.sh
# see log for init pw
# go tohttp://localhost:8080/configure and "SCM Sync configuration" section
# include separatly:
org.jenkinsci.plugins.workflow.flow.FlowExecutionList.xml
com.nirima.jenkins.plugins.docker.DockerPluginConfiguration.xml
nodes/*/config.xml
jobs/**/nextBuildNumber
secrets/jenkins.slaves.JnlpSlaveAgentProtocol.secret
secrets/master.key
# specifiy the git-repo above, i.e.:
git@github.com:tim-pl-m/public-jenkins-config.git
# a commit should appear in your config-repo

# now lets try failover:
docker stop jenkins-master
docker rm -f jenkins-master
docker rmi -f jenkins-master

cd images/master
./bin/image-build.sh
./bin/container-run.sh
# if you lost your credentials change
<useSecurity>true</useSecurity>
# in your config.xml to false, enable security and press sign up on top right corner.
# if you didnt lost your credentials:
# log in with docker:jenkins
#TODO check if password wasnt changed

# TODO add script at root levelto start jenkins after instance restart/add jenkins as docker-service:
sh jenkins-in-docker/images/master/bin/container-rerun.sh


```
wip:
# install docker-compose
apt install docker-compose -y
docker-compose up -d










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