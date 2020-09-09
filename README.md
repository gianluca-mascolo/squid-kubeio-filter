# squid-kubeio-filter

## Goal
I'm preparing for Kubernetes certification. Since the only website allowed during the exam is the official [kubernetes.io](https://kubernetes.io/docs/home/) documentation, I wrote a squid proxy configuration that allow me to stuck on that site while study.

## Requirements

This PoC is meant to be run on minikube on your computer, so you must have it installed and running. Just apply the yaml file with
```
kubectl apply -f squid.yaml
```
It use the following images on Docker Hub
- [b4tman/squid](https://hub.docker.com/r/b4tman/squid) ([source](https://github.com/b4tman/docker-squid))
- [frapsoft/openssl](https://hub.docker.com/r/frapsoft/openssl) ([source](https://github.com/ellerbrock/openssl-docker))

## Howto

After resource creation on Kubernetes, you need to configure your browser to use a proxy with a custom certificate to access the Internet.
### SSL Certificate
You can extract the custom certificate from the running squid with (example)
```
]$ kubectl get pods -l app=squid
NAME                     READY   STATUS    RESTARTS   AGE
squid-764554f67f-9hcrl   1/1     Running   0          36m
]$ kubectl cp squid-764554f67f-9hcrl:/etc/squid/cert/squid-ca-cert.pem /tmp/squid-ca-cert.pem
```
### Proxy Address
The proxy is reachable from your computer at _MinikubeIp_:_NodePort_. Example:
```
]$ kubectl get svc -l app=squid
NAME    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
squid   NodePort   10.98.172.39   <none>        3128:32347/TCP   54m
]$ minikube ip
192.168.39.230
```
HTTP proxy address in the example is 192.168.39.230:32347
### Browser configuration
- Configure your HTTP and HTTPS proxy to _MinikubeIp_:_NodePort_ (do not forget HTTPS or the filter won't work)
- Load the squid-ca-cert.pem in your certification authorities list
- Extra Tip: you can use a custom profile in your browser only for that, e.g. with `firefox -P`

## Notes

- You can monitor squid logs with `kubectl logs -f -l app=squid`
- URL filter is customized to include resources coming from external sites into kubernetes.io site. This may change in future.
- This configuration allow access to https://kubernetes.io/docs/ and https://kubernetes.io/search/ only. Other sections like blog are forbidden.
