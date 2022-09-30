##  搭建 Concourse Pipeline

### 环境

linux 或者 macos， windows 暂时不知道怎么弄 😄

### 安装 docker

```shell
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli docker-compose containerd.io docker-compose-plugin
sudo service docker start
sudo docker run hello-world # 进入到 docker 中
exit # 退出 docker
```


### 安装 Concourse CI

```shell
git clone https://github.com/concourse/concourse-docker.git
cd concourse-docker
./keys/generate
docker-compose up -d
```

接着用浏览器打开 `http://localhost:8080` 看部署是否成功，看到下述画面表示成功安装 Concourse CI。

<img width="1916" alt="截屏2022-09-30 16 56 23" src="https://user-images.githubusercontent.com/13810907/193233277-4ee02ae3-3d8e-45a5-bc80-4c949de790b3.png">



### 安装 Fly CLI

https://github.com/concourse/concourse/releases

下载对应的包，使得 `fly` 命令可用。解压后，

```shell
mv fly /usr/local/bin
```

### Hello World Demo

```shell
fly targets
```

```shell
fly -t ci set-pipeline -p test_pipeline -c pipeline.yml
```
<img width="1920" alt="截屏2022-09-30 17 18 50" src="https://user-images.githubusercontent.com/13810907/193237906-ac8e5795-0b3e-4896-bc13-9a673f4db079.png">



### 参考

1. https://github.com/concourse/concourse

### 练习

# Setup a Concourse CI(Continuous Integration) example pipeline.

1. Refer to Quick Start https://concourse-ci.org/quick-start.html to deploy Concourse using docker-compose.
2. Setup the hello-world pipeline based on https://concourse-ci.org/tutorial-hello-world.html.
3. Change the hello-world pipeline with two new requirements:

Requirement I. Concourse will run CI jobs in a docker container. Please create a user-defined docker image and upload it to docker hub.
Then Concourse can fetch and use this container. Docker Image requirement:
a. base image: Ubuntu20.04
b. install package libleveldb-dev
c. generate a rsa key.
d. [Plus] enable coredump

Requirement II. Instead of just echo helloworld, you should write a simple Concourse CI pipeline to get a github repo, compile a C program and run the program.
Refer to Concourse documentation(https://concourse-ci.org/docs.html).
The expected jobs:
a. Fetch a github repo (https://github.com/zhangh43/hello)
b. compile the a.cpp in repo HELLO (`g++ --std=c++17 a.cpp`)
c. run the HELLO program (`./a.out`)
d. [Plus] Using Concourse github resource instead of clone the repo manually (refer to https://github.com/concourse/git-resource)

