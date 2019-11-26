# SimpleLB

Simple LB is the simplest Load Balancer ever created.

It uses RoundRobin algorithm to send requests into set of backends and support
retries too.

It also performs active cleaning and passive recovery for unhealthy backends.

Since its simple it assume if / is reachable for any host its available

# How to use
```bash
Usage:
  -backends string
        Load balanced backends, use commas to separate
  -port int
        Port to serve (default 3030)
```

Example:

To add followings as load balanced backends
- http://localhost:3031
- http://localhost:3032
- http://localhost:3033
- http://localhost:3034
```bash
simple-lb.exe --backends=http://localhost:3031,http://localhost:3032,http://localhost:3033,http://localhost:3034
```


# docker version diff
default is: 1.13.1
but docker-ce is 19.03

# install process
1, yum erase docker docker-client docker-common
2, install latest version docker
[root@our-server simplelb]# curl -fsSL https://get.docker.com/ | sh
# Executing docker install script, commit: f45d7c11389849ff46a6b4d94e0dd1ffebca32c1
+ sh -c 'yum install -y -q yum-utils'
+ sh -c 'yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo'
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
+ '[' stable '!=' stable ']'
+ sh -c 'yum makecache'
base                                                                                  | 3.6 kB  00:00:00     
docker-ce-stable                                                                      | 3.5 kB  00:00:00     
epel                                                                                  | 5.4 kB  00:00:00     
extras                                                                                | 2.9 kB  00:00:00     
mongodb-org-3.6                                                                       | 2.5 kB  00:00:00     
remi-safe                                                                             | 3.0 kB  00:00:00     
updates                                                                               | 2.9 kB  00:00:00     
(1/15): base/7/x86_64/other_db                                                        | 2.6 MB  00:00:00     
(2/15): docker-ce-stable/x86_64/filelists_db                                          |  18 kB  00:00:00     
(3/15): docker-ce-stable/x86_64/primary_db                                            |  37 kB  00:00:00     
(4/15): epel/x86_64/prestodelta                                                       | 6.0 kB  00:00:00     
(5/15): epel/x86_64/filelists_db                                                      |  12 MB  00:00:00     
(6/15): epel/x86_64/other_db                                                          | 3.3 MB  00:00:00     
(7/15): docker-ce-stable/x86_64/other_db                                              | 111 kB  00:00:00     
(8/15): extras/7/x86_64/other_db                                                      | 100 kB  00:00:00     
(9/15): docker-ce-stable/x86_64/updateinfo                                            |   55 B  00:00:00     
(10/15): epel/x86_64/updateinfo_zck                                                   | 1.5 MB  00:00:00     
(11/15): updates/7/x86_64/other_db                                                    | 267 kB  00:00:00     
(12/15): updates/7/x86_64/filelists_db                                                | 2.7 MB  00:00:00     
(13/15): mongodb-org-3.6/7/other_db                                                   | 7.6 kB  00:00:02     
(14/15): remi-safe/other_db                                                           | 435 kB  00:00:02     
(15/15): remi-safe/filelists_db                                                       | 1.3 MB  00:00:38     
Metadata Cache Created
+ '[' -n '' ']'
+ sh -c 'yum install -y -q docker-ce'
warning: /var/cache/yum/x86_64/7/docker-ce-stable/packages/containerd.io-1.2.10-3.2.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
Public key for containerd.io-1.2.10-3.2.el7.x86_64.rpm is not installed
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060a 61c5 1b55 8a7f 742b 77aa c52f eb6b 621e 9f35
 From       : https://download.docker.com/linux/centos/gpg
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.

3, check new version
[root@our-server simplelb]# docker --version
Docker version 19.03.5, build 633a0ea


4, restart docker
service docker restart

5, docker-compose up -d
[root@our-server simplelb]# docker-compose up -d
Building front
Step 1/9 : FROM golang:1.13 AS builder
1.13: Pulling from library/golang
Digest: sha256:f9de064473fb30c66bc0d2ddf2cf9a4a9bd38cbd2c5e59ce3bdf8af7b8747a57
Status: Downloaded newer image for golang:1.13
 ---> a2e245db8bd3
Step 2/9 : WORKDIR /app
 ---> Using cache
 ---> f316ef0346ad
Step 3/9 : COPY main.go go.mod ./
 ---> b5cb4cb1401c
Step 4/9 : RUN CGO_ENABLED=0 GOOS=linux go build -o lb .
 ---> Running in b92f4c13b511
Removing intermediate container b92f4c13b511
 ---> a9ebe62ea9c5
Step 5/9 : FROM alpine:latest
latest: Pulling from library/alpine
89d9c30c1d48: Pull complete
Digest: sha256:c19173c5ada610a5989151111163d28a67368362762534d8a8121ce95cf2bd5a
Status: Downloaded newer image for alpine:latest
 ---> 965ea09ff2eb
Step 6/9 : RUN apk --no-cache add ca-certificates
 ---> Running in d5921ea2b83e
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/community/x86_64/APKINDEX.tar.gz
(1/1) Installing ca-certificates (20190108-r0)
Executing busybox-1.30.1-r2.trigger
Executing ca-certificates-20190108-r0.trigger
OK: 6 MiB in 15 packages
Removing intermediate container d5921ea2b83e
 ---> f6b7b72bc71c
Step 7/9 : WORKDIR /root
 ---> Running in e2c89ca30647
Removing intermediate container e2c89ca30647
 ---> 2b501ac1e7f7
Step 8/9 : COPY --from=builder /app/lb .
 ---> 6cfd0439341f
Step 9/9 : ENTRYPOINT [ "/root/lb" ]
 ---> Running in b3f661f29164
Removing intermediate container b3f661f29164
 ---> f8344d8e8c8c
Successfully built f8344d8e8c8c
Successfully tagged simplelb_front:latest
WARNING: Image for service front was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Pulling web1 (strm/helloworld-http:latest)...
latest: Pulling from strm/helloworld-http
85b1f47fba49: Pull complete
8f137bf105ee: Pull complete
8fb76b9403d9: Pull complete
b53610b50bff: Pull complete
Digest: sha256:bd44b0ca80c26b5eba984bf498a9c3bab0eb1c59d30d8df3cb2c073937ee4e45
Status: Downloaded newer image for strm/helloworld-http:latest
Creating load-balancer ... done
Creating simplelb_web1_1  ... done
Creating simplelb_web2_1  ... done
Creating simplelb_web3_1  ... done

6, test method
[root@our-server simplelb]# curl localhost:3030
<html><head><title>HTTP Hello World</title></head><body><h1>Hello from e82deedcf4a3</h1></body></html
[root@our-server simplelb]# curl localhost:3030
<html><head><title>HTTP Hello World</title></head><body><h1>Hello from 7c450c2167d1</h1></body></html
[root@our-server simplelb]# curl localhost:3030
<html><head><title>HTTP Hello World</title></head><body><h1>Hello from 9c1b828dddc2</h1></body></html

we can see, response from three different backend

7, stop all containers
[root@our-server simplelb]# docker-compose kill
Killing simplelb_web2_1 ... done
Killing simplelb_web1_1 ... done
Killing load-balancer   ... done
Killing simplelb_web3_1 ... done

