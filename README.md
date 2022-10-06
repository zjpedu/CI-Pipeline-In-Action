##  Concourse Pipeline

### Environments

* ubuntu/centos x86_64 ✅
* linux arm64 ubuntu/centos https://github.com/zjpedu/concourse-arm64 ❌
* m1 pro apple silicon macos ❌
* windows 🤷‍♂️

### Install Docker

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
sudo service docker status 
docker --version
# sudo docker run hello-world
# exit
```


### Install Concourse CI

```shell
git clone https://github.com/concourse/concourse-docker.git
cd concourse-docker
./keys/generate
docker-compose up -d
docker-compose down
```


### Install Fly CLI

https://github.com/concourse/concourse/releases

Download the `fly`tool and decompress. Then moving the fly to `/usr/local/bin`

### Demo Show

```shell
fly -t ci login -c http://localhost:8080 -u admin -p admin
fly targets
```

Open web browser `http://localhost:8080`, user_name: admin password: admin

<img width="1916" alt="截屏2022-09-30 16 56 23" src="https://user-images.githubusercontent.com/13810907/193233277-4ee02ae3-3d8e-45a5-bc80-4c949de790b3.png">

* pipeline.yml

```shell
jobs:
  - name: job-hello-csapp
    public: true
    plan:
      - task: hello-csapp
        config:
          platform: linux
          image_resource:
            type: docker-image
            source: {
                repository: ubuntu,
                tag: 20.04
          }
          run:
            path: echo
            args: [hello, csapp!]
```

```shell
fly -t ci set-pipeline -p test_pipeline -c pipeline.yml
```
<img width="1912" alt="截屏2022-10-02 15 04 13" src="https://user-images.githubusercontent.com/13810907/193442261-0069fad0-02a9-4aa1-b5e7-05b02566f613.png">


### Reference

1. https://github.com/concourse/concourse

### Excerises

#### Setup a Concourse CI(Continuous Integration) example pipeline.

1. Refer to Quick Start https://concourse-ci.org/quick-start.html to deploy Concourse using docker-compose.
2. Setup the hello-world pipeline based on https://concourse-ci.org/tutorial-hello-world.html.
3. Change the hello-world pipeline with two new requirements:

Requirement I. Concourse will run CI jobs in a docker container. Please create a user-defined docker image and upload it to docker hub.
Then Concourse can fetch and use this container. Docker Image requirement:

1. base image: Ubuntu20.04
2. install package libleveldb-dev
3. generate a rsa key.
4. [Plus] enable coredump

下面的 pipeline.yml 会生成 coredump，放在 `/tmp` 目录下，换成其他目录都不能成功，必须要配置 privileged: true，这样才能在 docker container 中生成 coredump
```shell
jobs:
    - name: coredump
      public: true
      plan:
        - task: execute-the-tasks
          privileged: true
          config:
            platform: linux
            image_resource:
              type: docker-image
              source: {
                repository: ubuntu,
                tag: 20.04
              }
            run:
               user: root
               path: /bin/bash
               args:
                - "-e"
                - "-c"
                - |
                  set -x
                  ulimit -c unlimited
                  echo 'ulimit -c unlimited' >> ~/.bash_profile
                  echo '/tmp/core.%t.%e.%p' | tee /proc/sys/kernel/core_pattern
                  apt-get update
                  apt-get install git -y
                  apt-get install gcc -y
                  apt-get install vim -y
                  apt-get install libleveldb-dev -y
                  apt-get install openssh-client -y
                  ssh-keygen -q -t rsa -N '' -f /root/.ssh/id_rsa
                  source ~/.bash_profile
                  cat /root/.ssh/id_rsa.pub
                  git clone https://github.com/zjpedu/Computer-Systems-Labs
                  cd Computer-Systems-Labs/gdb_test
                  gcc -g -o test test3.c
                  ./test
```

Requirement II. Instead of just echo helloworld, you should write a simple Concourse CI pipeline to get a github repo, compile a C program and run the program.

Refer to Concourse documentation(https://concourse-ci.org/docs.html).

The expected jobs:

1. Fetch a github repo (https://github.com/zjpedu/Computer-Systems-Labs))
2. 完成 lab2

A: pipeline.yml

```shell
jobs:
    - name: lab2
      public: true
      plan:
        - task: execute-the-tasks
          config:
            platform: linux
            image_resource:
              type: docker-image
              source: {
                repository: ubuntu,
                tag: 20.04
              }
            run:
               path: /bin/sh
               args:
                - "-e"
                - "-c"
                - |
                  set -x
                  apt-get update
                  apt-get install gcc -y
                  apt-get install gcc-multilib -y
                  apt-get install make -y
                  apt-get install git -y
                  git clone https://github.com/Lily127Yang/Computer-Systems-Labs.git
                  cd Computer-Systems-Labs/lab2/datalab-handout
                  make
                  ./btest -T 50
                  result=`./btest -T 50 | grep "Total point" | cut -d " " -f3 | cut -d "/" -f1`
                  ddl=`date -d "2022-10-09 23:59" +%s --utc`
                  current_time=`date +%s`
                  [ $current_time -le $ddl ]
                  [ $result -ge 36 ]
```

or using the following method (3. [Plus] Using Concourse github resource instead of clone the repo manually (refer to https://github.com/concourse/git-resource))

```shell
resources:
- icon: github
  name: csapp
  source:
    uri: https://github.com/Lily127Yang/Computer-Systems-Labs.git
  type: git

jobs:
    - name: lab2
      public: true
      plan:
        - get: csapp
          trigger: true
        - task: execute-the-tasks
          config:
            inputs:
              - name: csapp
            platform: linux
            image_resource:
              type: docker-image
              source: {
                repository: ubuntu,
                tag: 20.04
              }
            run:
               path: /bin/sh
               args:
                - "-e"
                - "-c"
                - |
                  set -x
                  apt-get update
                  apt-get install gcc -y
                  apt-get install gcc-multilib -y
                  apt-get install make -y
                  apt-get install git -y
                  cd csapp/lab2/datalab-handout
                  make
                  ./btest -T 50
                  result=`./btest -T 50 | grep "Total point" | cut -d " " -f3 | cut -d "/" -f1`
                  ddl=`date -d "2022-10-09 23:59" +%s --utc`
                  current_time=`date +%s`
                  [ $current_time -le $ddl ]
                  [ $result -ge 36 ]
```



start docker container inspect the `run` command

```shell
sudo docker run -it ubuntu bash
```

### 进入到 concourse CI 中调试

```shell
fly -t ci_name login # 先完成登陆
fly targets  # 查看 targets，url 需要和下面的 build_url 拼接 返回的 url 为：http://localhost:8080
fly -t ci_name hijack -u build_url # 登陆后就可以进入到 bash 界面，使用 gdb 等调试代码 如：fly -t ci hijack -u http://localhost:8080/teams/main/pipelines/core/jobs/coredump/builds/10  这个后面的具体内容为 build_url 拼接部分 /teams/main/pipelines/core/jobs/coredump/builds/10
apt-get install gdb -y
gdb xxx coredump_file
```

## GitHub Actions

Reference by my github action workflow https://github.com/zjpedu/Computer-Systems-Labs/tree/main/.github/workflows
