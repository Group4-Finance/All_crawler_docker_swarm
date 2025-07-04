# Swarm 流程
## 前置工作
必須先建立 爬蟲 以及 api 這兩個 image，後續啟動 docker swarm 的 yml 檔時會讀取對應的 image
，讓docker去執行爬蟲、發送任務以及建立api

前面的爬蟲、發送任務、mysql必須確保每個階段都能正常運作，否則到後面 swarm 一樣會出問題。

## 初始化
docker swarm init

## 創建 docker swarm network  
docker network create --scope=swarm --driver=overlay network <名稱>

Ex: docker network create --scope=swarm --driver=overlay my_swarm_network

## 拉取官方image
docker pull portainer/agent
docker pull portainer/portainer-ce:2.0.1

## 部屬portainer
docker stack deploy -c portainer.yml <名稱>

Ex: docker stack deploy -c portainer.yml por

## 創建 volume
docker volume create <名稱>

Ex: docker volume create mysql

## 啟動 yml 檔
docker stack deploy --with-registry-auth -c rabbitmq-swarm.yml rabbitmq

docker stack deploy --with-registry-auth -c mysql-swarm.yml mysql

DOCKER_IMAGE_FULL=<填入自己的image的完整名稱> docker stack deploy --with-registry-auth -c docker-compose-worker-swarm.yml crawlerworker

DOCKER_IMAGE_FULL=<填入自己image的完整名稱> docker stack deploy --with-registry-auth -c docker-compose-producer-swarm.yml producer

DOCKER_IMAGE_FULL=<填入自己 api 的 image 完整名稱> docker stack deploy --with-registry-auth -c docker-compose-api-swarm.yml api

除了portainer、mysql、rabbitmq，後面三個 yml 檔啟動完必須到 http://127.0.0.1:9000/ 
的 swarm ，點選 docker-desktop，依據 worker、producer、api 的 service 數量 增加相應的 label 數量，接著查看以上三個 yml 檔 service 底下的 constraints ，填入對應的 name 以及 value，填完按下 apply changes 。

Ex: constraints: [node.labels.api == true]  在 label 的 name 填入 api 、 value 填入 true

## 離開docker swarm
docker swarm leave --force