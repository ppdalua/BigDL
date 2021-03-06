# Jenkins Docker
Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software. Here we run Jenkins in Docker, based on Ubuntu 14.04. 

## 1. Build images
Jenkins has master nodes and slave agents. To make Jenkins well functioning, masters and slaves should be built separatedly. 
### 1.1 Build master
First, you should step into the directory jenkins-master and run
```bash
sudo docker build --rm -t bigdl/jenkins-master .
```
If proxy is needed for building the image, building arguments related to proxy should be added in the command above. For example,
```bash
sudo docker build --build-arg http_proxy=http://yourproxyhost:port \
                  --build-arg https_proxy=https://yourproxyhost:port \
                  --rm -t bigdl/jenkins-master .
```
Note that you can set any tag of the image by changine the parameter after the option -t.

There are some extra files other than Dockerfile in the directory jenkins-master.
+ jenkins.sh: launch Jenkins
+ jenkins-support: provide support of upgrading Jenkins and its plugins
+ install-plugins: resolve dependencies and download Jenkins plugins
+ init.groovy: provide initial configuration of launching Jenkins master

### 1.2 Build slaves
Building Jenkins slave is similar to building Jenkins master. First, step into the directory jenkins-slave and run
```bash
sudo docker build --rm -t bigdl/jenkins-slave .
```
If proxy is needed for building the image, the following command should be run.
```bash
sudo docker build --build-arg http_proxy=http://yourproxyhost:port \
                  --build-arg https_proxy=https://yourproxyhost:port \
                  --rm -t bigdl/jenkins-slave .
```
There are some extra files other than Dockerfile in the directory jenkins-slave.
+ pip.conf: provide configurations of pip
+ run-slave.sh: launch Jenkins slave agents
+ settings-xml: provide settings of maven
+ slave.groovy: provide initial configurations of launching Jenkins slave agents
Please modify these files to add your own pypi index, maven settings like proxies, servers and mirrors before you build the images.

## 2. Run images
### 2.1 Run master
After building images of Jenkins master and slave agents, you should run the image of Jenkins master first using
```bash
sudo docker run -itd --net=host -p 8080:8080 -p 50000:50000 bigdl/jenkins-master
```
+ -p 8080:8080: This mapping of port is used for accessing Jenkins Web UI.
+ -p 50000:50000: This mapping of port aims at attaching build slave servers through JNLP (Java Web Start), which will be used when you connect a slave agent.
+ bigdl/jenkins-master: You can change it to your tag of Jenkins master images set in build step.
If you want to use proxy when running images, two more parameters should be added in the command.
```bash
sudo docker run -itd --net=host -p 8080:8080 -p 50000:50000 \
                     --env PROXY_HOST=yourproxyhost --env PROXY_PORT=yourproxyport bigdl/jenkins-master
```

### 2.2 Run slave
Then, you can run the image of Jenkins slave using
```bash
sudo docker run -itd --net=host \
               -e jenkins_master_host=yourJenkinsmasterhost \
               -e jenkins_master_port=yourJenkinsmasterport \
               -e jenkins_slave_label=yourslavelabel \
               -e jenkins_slave_name=yourslavename \
               -e jenkins_slave_executors=numberofyourslaveexecutors \
               --memory=50g \
               bigdl/jenkins-slave
```
Note that 8080 is the default Jenkins master port. After that, you can browse to http://yourJenkinsmasterhost:8080 to check information and status of both Jenkins master and slave agents.
Note that memory should be larger than 50G.

## 3. Configure Github and Git plugins
If you want to link Jenkins with Github, you should conduct some configurations when you first start Jenkins and log in.
* In **Manage Jenkins -> Configure System**, you should add a Github server with API URL: https://api.github.com/ and set global configurations in Git plugin part as you set Git at the first time.
* In **Credentials -> Systems -> Add domains**, you should add a domain with the name api.github.com. Then you need to add credentials in the domain **api.github.com** with the kind Username and password. The username and password should correspond to those of github accounts of users.
* When creating a new job, you should modify the setting of git in **Source Code Management** part of job configuration.
