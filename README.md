this stack contains:
Mariadb galera cluster deployed on ETC cluster with docker swarm
Memcached server
web service (remember to push your own bookface image here and replace it in the docker-compose file)
the dockerfile for both memcached(no need to rebuild and push) and web (edit config.php and push) is in a separate folder in this repo. 
to scale this service use 
sudo docker service "name of service"=5 (use odd numbers here)




#install centos on wanted number of docker swarm nodes, use centos as password when prompted
"open ports 2380, 2399, 2378 and 4001

#install ETCD
sudo yum install etcd

#install some needed utilities 
sudo yum install -y yum-utils \
   device-mapper-persistent-data \
   lvm2

#add docker-ce repo

sudo yum-config-manager \
     --add-repo \
     https://download.docker.com/linux/centos/docker-ce.repo

#install docker-ce
sudo yum install docker-ce
sudo systemctl start docker #remember to start docker
sudo  docker swarm join .... #and to join the swarm


#edit etcd.conf
sudo vi /etc/etcd/etcd.conf    #its important to comment out EVERY line, or remve them all together before adding the config


ETCD_NAME=etcd1                                             #name of the given instance, change for second node
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"                  #datadir,default, no need to change
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"                 #same for all configs
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"               #same for all configs
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.10.0.98:2380"   #change to the ip of the  host (the ip of the machine the config is deployed to)!!!
ETCD_INITIAL_CLUSTER="etcd1=http://10.10.0.98:2380,etcd2=http://10.10.0.159:2380,etcd3=http://10.10.1.57:2380,etcd4=http://10.10.1.10:2380"    #add all the nodes part of the swarm
ETCD_INITIAL_CLUSTER_STATE="new"                            #no need to change                                                                                                       
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-1"                 #no need to change
ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"            #no need to change

#enable etcd

sudo systemctl enable etcd

#check etcd health
sudo etcdctl cluster-health
sudo etcdctl members 
curl -L http://127.0.0.1:2379/health


#debugging

systemctl status etcd.service
journalctl -u etcd

if encountering and error where the service failes do the following
 sudo systemctl stop etcd
 sudo rm -r /var/lib/etcd/default.etcd/member    #also if getting mismatched IDs
 sudo systemctl daemon-reload
 sudo systemctl start etcd

#docker-compose
remember to add your own IP addresses at the discovery address list

if the service of mariadb is not starting, run sudo docker service logs "name of mariadb service" if this is something to do with nt being able to lock on the voulme, run ps aux |grep mysql and kill all mysql processes and deploy the stack again


deploy the stack using

sudo docker stack deploy -c docker-compose.yml

#documentation or further help:
https://github.com/severalnines/galera-docker-mariadb
https://severalnines.com/blog/mysql-docker-deploy-homogeneous-galera-cluster-etcd
https://severalnines.com/blog/mysql-docker-swarm-mode-limitations-galera-cluster-production-setups


The docker compose file uses my own web image with a custom php config image, build your own image and push it to docker hub

docker tag "nameofimage" "dockerhub_user/nameofrepo"
docker push dockerhub_user/nameofrepo
