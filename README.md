# go-web-hello-world

### Task 0: Install a ubuntu 16.04 server 64-bit

* Create one virtual box VM, add one virtual CD-ROM and attach the ubuntu16.04 ISO.
* Config the virtual network as NAT mode and forward required ports to host machine.
  > 22->2222 for ssh
  > 80->8080 for gitlab
  > 8081/8082->8081/8082 for go app
  > 31080/31081->31080/31081 for go app in k8s
* Power on the VM and start to install ubuntu16.04
* Ubuntu16.04 installation is finished
* Succeed to login ubuntu

### Task 1: Update system

* Ssh to the VM by putty
  > login as: demo
  > demo@localhost's password:
  > Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)
  > ...
  > Last login: Sun Feb 16 17:55:45 2020 from 10.0.2.2

* Upgrade the system
  > demo@localhost:~$ sudo apt update
  > [sudo] password for demo:
  > Hit:1 http://us.archive.ubuntu.com/ubuntu xenial InRelease
  > Get:2 http://security.ubuntu.com/ubuntu xenial-security InRelease [109 kB]
  > ...
  > Get:8 http://security.ubuntu.com/ubuntu xenial-security Release.gpg [916 B]
  > Fetched 226 kB in 6min 33s (574 B/s)
  > Reading package lists... Done
  > Building dependency tree
  > Reading state information... Done
  > 143 package

  > demo@localhost:~$ apt list --upgradable
  > Listing... Done
  > amd64-microcode/xenial-updates,xenial-security 3.20191021.1+really3.20180524.1~ubuntu0.16.04.2 amd64 [upgradable from: 3.20180524.1~ubuntu0.16.04.2]
  > apparmor/xenial-updates,xenial-security 2.10.95-0ubuntu2.11 amd64 [upgradable from: 2.10.95-0ubuntu2.10]
  > ...
  > zlib1g/xenial-updates,xenial-security 1:1.2.8.dfsg-2ubuntu4.3 amd64 [upgradable from: 1:1.2.8.dfsg-2ubuntu4.1]
  > s can be upgraded. Run 'apt list --upgradable' to see them.

  > demo@localhost:~$ sudo apt upgrade -y
  > Reading package lists... Done
  > Building dependency tree
  > Reading state information... Done
  > Calculating upgrade... Done
  > The following NEW packages will be installed:
  > linux-headers-4.4.0-173 linux-headers-4.4.0-173-generic linux-image-4.4.0-173-generic linux-modules-4.4.0-173-generic linux-modules-extra-4.4.0-173-generic
  > The following packages will be upgraded:
  > amd64-microcode apparmor apport apt apt-transport-https apt-utils base-files bash bind9-host bsdutils btrfs-tools busybox-initramfs busybox-static bzip2
  > ...
  > Found linux image: /boot/vmlinuz-4.4.0-173-generic
  > Found initrd image: /boot/initrd.img-4.4.0-173-generic
  > Found linux image: /boot/vmlinuz-4.4.0-142-generic
  > Found initrd image: /boot/initrd.img-4.4.0-142-generic
  > done

* Reboot then check the kernel version
  > login as: demo
  > demo@localhost's password:  
  > Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-173-generic x86_64)
  > ...
  > 0 packages can be updated.
  > 0 updates are security updates.
  > ...

### Task 2: install gitlab-ce version in the host

* Update ubuntu source to mirrors.aliyun.com
  > vi /etc/apt/sources.list
  > deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
  > deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted

* Install curl openssh-server ca-certificates
  > demo@localhost:~$ sudo apt-get install -y curl openssh-server ca-certificates
  > Reading package lists... Done
  > Building dependency tree
  > Reading state information... Done
  > ca-certificates is already the newest version (20170717~16.04.2).
  > curl is already the newest version (7.47.0-1ubuntu2.14).
  > openssh-server is already the newest version (1:7.2p2-4ubuntu2.8).
  > 0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

* Install postfix
  > demo@localhost:~$ sudo apt-get install -y postfix
  > Reading package lists... Done
  > Building dependency tree
  > Reading state information... Done
  > postfix is already the newest version (3.1.0-3ubuntu0.3).
  > 0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

* Add the GitLab package repository
  > demo@localhost:~$ curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
  > Detected operating system as Ubuntu/xenial.
  > Checking for curl...
  > Detected curl...
  > Checking for gpg...
  > Detected gpg...
  > Running apt-get update... done.
  > Installing apt-transport-https... done.
  > Installing /etc/apt/sources.list.d/gitlab_gitlab-ce.list...done.
  > Importing packagecloud gpg key... done.
  > Running apt-get update... done.
  > 
  > The repository is setup! You can now install packages.

* Install gitlab-ce
  > demo@localhost:~$ sudo EXTERNAL_URL="http://127.0.0.1" apt-get install gitlab-ce
  > Reading package lists... Done
  > Building dependency tree
  > Reading state information... Done
  > The following NEW packages will be installed:
  >  gitlab-ce
  > ...
  > 
  >     _______ __  __          __
  >     / ____(_) /_/ /   ____ _/ /_
  >   / / __/ / __/ /   / __ `/ __ \
  >   / /_/ / / /_/ /___/ /_/ / /_/ /
  >   \____/_/\__/_____/\__,_/_.___/
  > 
  > Upgrade complete! If your GitLab server is misbehaving try running
  >   sudo gitlab-ctl restart
  > before anything else.
  > If you need to roll back to the previous version you can use the database
  > backup made during the upgrade (scroll up for the filename).

* Access it from http://127.0.0.1, set the root password, create the account

### Task 3: create a demo group/project in gitlab

* Install ssh, generate public key and copy to gitlab server.
  > demo@localhost:~$ sudo apt install ssh/xenial-updates
  > Reading package lists... Done
  > ...
  > Unpacking ssh (1:7.2p2-4ubuntu2.8) ...
  > Setting up ssh (1:7.2p2-4ubuntu2.8) ...

  > demo@localhost:~$ ssh-keygen
  > Generating public/private rsa key pair.
  > Enter file in which to save the key (/home/demo/.ssh/id_rsa):
  > ...
  > +----[SHA256]-----+

  > demo@localhost:~$ vi .ssh/id_rsa.pub

* Install golang
  > demo@localhost:~/go-web-hello-world$ sudo apt install golang
  > Reading package lists... Done
  > ...
  > Setting up golang-race-detector-runtime (2:1.6-1ubuntu4) ...
  > Processing triggers for libc-bin (2.23-0ubuntu11) ...
  
* Clone the project
  > demo@localhost:~$ git clone git@127.0.0.1:demo/go-web-hello-world.git
  > Cloning into 'go-web-hello-world'...
  > ...
  > Receiving objects: 100% (3/3), 233 bytes | 0 bytes/s, done.
  > remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
  > Checking connectivity... done.

  > demo@localhost:~$ cd go-web-hello-world/
  > demo@localhost:~/go-web-hello-world$ ls
  > README.md
  > demo@localhost:~/go-web-hello-world$ git branch
  > * master
  
* Create hello.go
  > demo@localhost:~/go-web-hello-world$ vi hello.go
  > package main
  > 
  > import (
  >    "fmt"
  >     "net/http"
  > )
  > 
  > func main() {
  >    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
  >        fmt.Fprintf(w, "Go Web Hello World!")
  >    })
  > 
  >   http.ListenAndServe(":8081", nil)
  > }

* Submit the code to gitlab
  > demo@localhost:~/go-web-hello-world$ git config --global user.email 'liyongtao2000@126.com'
  > demo@localhost:~/go-web-hello-world$ git config --global user.name 'Yongtao Li'
  > demo@localhost:~/go-web-hello-world$ git commit hello.go -s
  > [master dda0cd9] hello world sample code
  >  1 file changed, 14 insertions(+)
  >  create mode 100644 hello.go
  > demo@localhost:~/go-web-hello-world$ git push origin master
  > Total 0 (delta 0), reused 0 (delta 0)
  > To git@127.0.0.1:demo/go-web-hello-world.git
  >  43b0c65..dda0cd9  master -> master

* Source code at gitlab

### Task 4: build the app and expose ($ go run) the service to 8081 port

* Run the app
  > demo@localhost:~/go-web-hello-world$ go run hello.go
  >
  
* Output on http://127.0.0.1:8081
 
### Task 5: install docker
 
  > demo@localhost:~$ sudo apt-get install apt-transport-https ca-certificates gnupg-agent curl software-properties-common
  > [sudo] password for demo:
  > Reading package lists... Done
  > ...
  > Setting up gnupg-agent (2.1.11-6ubuntu2.1) ...
  > Processing triggers for libc-bin (2.23-0ubuntu11) ...

  > demo@localhost:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  > OK

  > demo@localhost:~$ sudo apt-key fingerprint 0EBFCD88
  > pub   4096R/0EBFCD88 2017-02-22
  >       Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
  > uid                  Docker Release (CE deb) <docker@docker.com>
  > sub   4096R/F273FCD8 2017-02-22

  > demo@localhost:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

  > demo@localhost:~/go-web-hello-world$ sudo apt-get update
  > Hit:1 http://mirrors.aliyun.com/ubuntu xenial InRelease
  > Hit:2 http://mirrors.aliyun.com/ubuntu xenial-updates InRelease
  > ...
  > Reading package lists... Done

  > demo@localhost:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io
  > Reading package lists... Done
  > ...
  > The following NEW packages will be installed:
  > aufs-tools cgroupfs-mount containerd.io docker-ce docker-ce-cli libltdl7 pigz
  > ...
  > Processing triggers for systemd (229-4ubuntu21.27) ...

  > demo@localhost:~$ sudo docker run hello-world
  > Unable to find image 'hello-world:latest' locally
  > ...
  > Hello from Docker!
  > This message shows that your installation appears to be working correctly.

### Task 6: run the app in container

* Pull golang image
  > demo@localhost:~$ sudo docker pull golang
  > Using default tag: latest
  > latest: Pulling from library/golang
  > dc65f448a2e2: Pull complete
  > 346ffb2b67d7: Pull complete
  > dea4ecac934f: Pull complete
  > 8ac92ddf84b3: Pull complete
  > 7ca605383307: Pull complete
  > f47e6cebc512: Pull complete
  > 530350156010: Pull complete
  > Digest: sha256:9295ba678e3764d79ac0aeabdbcf281a91933c81c8de29387d8a2f557e256cdb
  > Status: Downloaded newer image for golang:latest
  > docker.io/library/golang:latest

  > demo@localhost:~$ sudo docker images
  > REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  > golang              latest              297e5bf50f50        3 days ago          803MB
  > hello-world         latest              fce289e99eb9        13 months ago       1.84kB

* Update the port to 8082 in hello.go
  > demo@localhost:~/go-web-hello-world$ vi hello.go
  > http.ListenAndServe(":8082", nil)
  
* Create the Dockerfile
  > demo@localhost:~/go-web-hello-world$ vi Dockerfile
  > FROM golang
  > 
  > COPY ./hello.go /go/src/hello.go

* Build  with the dockerfile
  > demo@localhost:~/go-web-hello-world$ sudo docker build .
  > Sending build context to Docker daemon  56.32kB
  > Step 1/2 : FROM golang
  > ---> 297e5bf50f50
  > Step 2/2 : COPY ./hello.go /go/src/hello.go
  > ---> 0c5783f9406e
  > Successfully built 0c5783f9406e
  
* Run the docker
  > demo@localhost:~/go-web-hello-world$ sudo docker run -it -p 8082:8082 0c5783f9406e go run /go/src/hello.go
 
* docker ps in another shell
  > demo@localhost:~$ sudo docker ps
  > [sudo] password for demo:
  > CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
  > 24394ed8f610        0c5783f9406e        "go run /go/src/hell…"   39 seconds ago      Up 38 seconds       0.0.0.0:8082->8082/tcp   intelligent_kare
  
* Access http://127.0.0.1:8082
 
* Checkin Dockfile
  > demo@localhost:~/go-web-hello-world$ git add Dockerfile
   
  > demo@localhost:~/go-web-hello-world$ git commit Dockerfile -s
  > [master 3a8dae7] Create Dockerfile for go app
  >  1 file changed, 4 insertions(+)
  >  create mode 100644 Dockerfile
  
  > demo@localhost:~/go-web-hello-world$ git push origin master
  > Counting objects: 3, done.
  > Compressing objects: 100% (2/2), done.
  > Writing objects: 100% (3/3), 385 bytes | 0 bytes/s, done.
  > Total 3 (delta 0), reused 0 (delta 0)
  > To git@127.0.0.1:demo/go-web-hello-world.git
  >   dda0cd9..3a8dae7  master -> master

### Task 7: push image to dockerhub

  > demo@localhost:~/go-web-hello-world$ sudo docker login
  > Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
  > Username: lyt2000
  > Password:
  > WARNING! Your password will be stored unencrypted in /home/demo/.docker/config.json.
  > Configure a credential helper to remove this warning. See
  > https://docs.docker.com/engine/reference/commandline/login/#credentials-store
  > 
  > Login Succeeded
  
  > demo@localhost:~/go-web-hello-world$ sudo docker tag 0c5783f9406e lyt2000/go-web-hello-world:v0.1
  
  > demo@localhost:~/go-web-hello-world$ sudo docker push lyt2000/go-web-hello-world:v0.1
  > The push refers to repository [docker.io/lyt2000/go-web-hello-world]
  > 22eb9a1c152a: Layer already exists
  > cae11887bc90: Mounted from library/golang
  > 729c3ac48990: Layer already exists
  > 8378cd889737: Layer already exists
  > 5c813a85f7f0: Layer already exists
  > bdca38f94ff0: Layer already exists
  > faac394a1ad3: Layer already exists
  > ce8168f12337: Layer already exists
  > v0.1: digest: sha256:f41779a84231250bcc49299caedb9ffa93b883d9fc2d47f321c0577533b09e87 size: 2002
  
* Output on docker hub
 
### Task 9: install a single node Kubernetes cluster using kubeadm
 
  > demo@localhost:~$ sudo systemctl stop ufw
  > demo@localhost:~$ sudo systemctl disable ufw
  > demo@localhost:~$ sudo setenforce 0
  > demo@localhost:~$ sudo swapoff -a
 
  > demo@localhost:~$ sudo vi  /etc/docker/daemon.json
  > {
  >    "registry-mirrors": ["https://alzgoonw.mirror.aliyuncs.com"],
  >    "live-restore": true,
  >    "exec-opts": [ "native.cgroupdriver=systemd" ]
  > }
 
  > demo@localhost:~$ sudo systemctl daemon-reload
  > demo@localhost:~$ sudo systemctl restart docker

  > demo@localhost:~$ sudo apt-get install -y apt-transport-https curl
  > Reading package lists... Done
  > ...
  > 0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.

  > demo@localhost:~$ curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
  > OK

  > demo@localhost:~$ vi /etc/apt/sources.list.d/kubernetes.list
  > deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main

  > demo@localhost:~$ sudo apt-get install -y kubelet kubeadm kubectl
  > Reading package lists... Done
  > Building dependency tree
  > ...
  > Setting up kubelet (1.17.3-00) ...
  > Setting up kubectl (1.17.3-00) ...
  > Setting up kubeadm (1.17.3-00) ...
  > Processing triggers for ureadahead (0.100.0-19.1) ...
  > Processing triggers for systemd (229-4ubuntu21.27) ...

  > demo@localhost:~$ sudo apt-mark hold kubelet kubeadm kubectl
  > kubelet set on hold.
  > kubeadm set on hold.
  > kubectl set on hold.
  > demo@localhost:~$ sudo systemctl daemon-reload
  > demo@localhost:~$ sudo systemctl restart kubelet

  > demo@localhost:~$ sudo kubeadm config images list
  > k8s.gcr.io/kube-apiserver:v1.17.3
  > k8s.gcr.io/kube-controller-manager:v1.17.3
  > k8s.gcr.io/kube-scheduler:v1.17.3
  > k8s.gcr.io/kube-proxy:v1.17.3
  > k8s.gcr.io/pause:3.1
  > k8s.gcr.io/etcd:3.4.3-0
  > k8s.gcr.io/coredns:1.6.5

  > demo@localhost:~$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.17.3
  > demo@localhost:~$ sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.17.3 k8s.gcr.io/kube-apiserver:v1.17.3
  > demo@localhost:~$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.17.3
  > demo@localhost:~$ sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.17.3 k8s.gcr.io/kube-controller-manager:v1.17.3
  > demo@localhost:~$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.17.3
  > demo@localhost:~$ sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.17.3 k8s.gcr.io/kube-scheduler:v1.17.3
  > demo@localhost:~$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.3
  > demo@localhost:~$ sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.17.3 k8s.gcr.io/kube-proxy:v1.17.3
  > demo@localhost:~$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
  > demo@localhost:~$ sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
  > demo@localhost:~$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0
  > demo@localhost:~$ sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0
  > demo@localhost:~$ sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.5
  > demo@localhost:~$ sudo docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.5 k8s.gcr.io/coredns:1.6.5
  > demo@localhost:~$ sudo docker images |grep k8s.gcr.io
  > k8s.gcr.io/kube-proxy                                                         v1.17.3             ae853e93800d        6 days ago          116MB
  > k8s.gcr.io/kube-controller-manager                                            v1.17.3             b0f1517c1f4b        6 days ago          161MB
  > k8s.gcr.io/kube-apiserver                                                     v1.17.3             90d27391b780        6 days ago          171MB
  > k8s.gcr.io/kube-scheduler                                                     v1.17.3             d109c0821a2b        6 days ago          94.4MB
  > k8s.gcr.io/coredns                                                            1.6.5               70f311871ae1        3 months ago        41.6MB
  > k8s.gcr.io/etcd                                                               3.4.3-0             303ce5db0e90        3 months ago        288MB
  > k8s.gcr.io/pause                                                              3.1                 da86e6ba6ca1        2 years ago         742kB

  > demo@localhost:~$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16
  > W0218 12:52:54.998807    7885 version.go:101] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt:    net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
  > W0218 12:52:54.999525    7885 version.go:102] falling back to the local client version: v1.17.3
  > W0218 12:52:55.004863    7885 validation.go:28] Cannot validate kube-proxy config - no validator is available
  > W0218 12:52:55.006277    7885 validation.go:28] Cannot validate kubelet config - no validator is available
  > [init] Using Kubernetes version: v1.17.3
  > [preflight] Running pre-flight checks
  > [preflight] Pulling images required for setting up a Kubernetes cluster
  > [preflight] This might take a minute or two, depending on the speed of your internet connection
  > [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
  > [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
  > [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
  > [kubelet-start] Starting the kubelet
  > [certs] Using certificateDir folder "/etc/kubernetes/pki"
  > [certs] Generating "ca" certificate and key
  > [certs] Generating "apiserver" certificate and key
  > [certs] apiserver serving cert is signed for DNS names [localhost kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.2.15]
  > [certs] Generating "apiserver-kubelet-client" certificate and key
  > ...
  > [bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
  > [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
  > [bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
  > [bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
  > [bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
  > [kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
  > [addons] Applied essential addon: CoreDNS
  > [addons] Applied essential addon: kube-proxy
  > 
  > Your Kubernetes control-plane has initialized successfully!
  > ...
  > You should now deploy a pod network to the cluster.
  > Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  > https://kubernetes.io/docs/concepts/cluster-administration/addons/
  > 
  > Then you can join any number of worker nodes by running the following on each as root:
  > 
  > kubeadm join 10.0.2.15:6443 --token w8k32d.b2u44oyvh9gzbm9x \
  >    --discovery-token-ca-cert-hash sha256:df948961e1b065223ce76a4a3cea2da608f81fce32171d6124a3f7bcd6306afa
    
  > demo@localhost:~$ mkdir -p $HOME/.kube
  > demo@localhost:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  > demo@localhost:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
  > demo@localhost:~$ export KUBECONFIG=/etc/kubernetes/admin.conf

  > demo@localhost:~$ sudo kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
  > configmap/calico-config created
  > customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
  > customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
  > customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
  > ...
  > clusterrole.rbac.authorization.k8s.io/calico-node created
  > clusterrolebinding.rbac.authorization.k8s.io/calico-node created
  > daemonset.apps/calico-node created
  > serviceaccount/calico-node created
  > deployment.apps/calico-kube-controllers created
  > serviceaccount/calico-kube-controllers created

  > demo@localhost:~$ sudo kubectl taint nodes --all node-role.kubernetes.io/master-
  > node/localhost untainted

  > demo@localhost:~$ sudo kubeadm join 10.0.2.15:6443 --token w8k32d.b2u44oyvh9gzbm9x \
  >     --discovery-token-ca-cert-hash sha256:df948961e1b065223ce76a4a3cea2da608f81fce32171d6124a3f7bcd6306afa
  > W0218 12:59:04.832387   13581 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
  > [preflight] Running pre-flight checks
  > error execution phase preflight: [preflight] Some fatal errors occurred:
  >        [ERROR DirAvailable--etc-kubernetes-manifests]: /etc/kubernetes/manifests is not empty
  >         [ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
  >         [ERROR Port-10250]: Port 10250 is in use
  >         [ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
  > [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
  > To see the stack trace of this error execute with --v=5 or higher

### Task 10: deploy the hello world container

  > demo@localhost:~$ cat go-web-hello-world_deployment.yaml
  > apiVersion: apps/v1
  > kind: Deployment
  > metadata:
  >  name: go-web-hello-world_deployment
  > spec:
  >   replicas: 1
  >   selector:
  >     matchLabels:
  >       app: go-web-hello-world
  >   template:
  >      metadata:
  >       labels:
  >         app: go-web-hello-world
  >     spec:
  >       containers:
  >       - name: my-demo
  >         image: lyt2000/go-web-hello-world:v0.1
  >         ports:
  >         - containerPort: 31080

  > demo@localhost:~$ sudo kubectl create -f go-web-hello-world_deployment.yaml --record

### Task 11: install kubernetes dashboard

  > demo@localhost:~$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml

### Task 12: generate token for dashboard login in task 11

  > demo@localhost:~$ vi dashboard-adminuser.yaml
  > apiVersion: v1
  > kind: ServiceAccount
  > metadata:
  >   name: admin-user
  >   namespace: kubernetes-dashboard
  >   
  > apiVersion: rbac.authorization.k8s.io/v1
  > kind: ClusterRoleBinding
  > metadata:
  >   name: admin-user
  > roleRef:
  >   apiGroup: rbac.authorization.k8s.io
  >   kind: ClusterRole
  >   name: cluster-admin
  > subjects:
  > - kind: ServiceAccount
  >   name: admin-user
  >   namespace: kubernetes-dashboard
  
  > demo@localhost:~$ sudo kubectl apply -f dashboard-adminuser.yaml
  > demo@localhost:~$ sudo kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

* Copy the token and paste it into Enter token field on login screen
