## Certified Kubernetes Administrator (CKA) 

```bash
    $ docker ==> $ nerdctl 
    $ docker run --name redis redis:alpine  ==> $ nerdctl run --name redis redis:alpine 
    $ docker run --name webserver -p 80:80 -d nginx ==> $ nerdctl run ---name webserver -p 80:80 -d nginx 
```

* CLI - crictl (cry control)

```bash
    $ crictl 
    $ crictl pull busybox 
    $ crictl images 
    $ crictl ps -a
    $ crictl exec -i -t xxxxxxxxxxxxxxxxx ls 
    $ crictl logs xxxxxxxxxxxxxxxxx
    $ crictl pods 
```
