##  搭建 Concourse Pipeline

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

```
```

### Hello World Demo

```shell
fly -t tutorial login -c http://localhost:8080 -u test -p test
fly targets
```

接着我们运行一个 demo

```shell
git clone https://github.com/starkandwayne/concourse-tutorial.git
cd concourse-tutorial/tutorials/basic/task-hello-world
fly -t tutorial execute -c task_hello_world.yml
```
<img width="919" alt="截屏2022-09-30 15 48 15" src="https://user-images.githubusercontent.com/13810907/193219327-cb6945d0-39e7-42a9-ab6e-a2f3ea1fbcbd.png">


可以看到，Concourse 收到这个 task 之后，下载了一个 Busybox 的 Docker 镜像，然后执行了 echo hello world 这条命令。那么，Concourse 是怎么知道要如何执行一个 task 呢？这就得从上面运行的 task_hello_world.yml 说起了。

```shell
fly -t tutorial set-pipeline -c task_hello_world.yml -p hello-world
```
