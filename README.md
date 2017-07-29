# Alternate NGINX controller for GKE


Why do you need this addition controller.
The default GKE deployed in the Goggle kubernetes cluster does not support the stanadrd NGINX features like redirection and basic authentication.


## Setup

First, create a loadbalancer Service and wait for it to acquire an IP

```console
$ kubectl create -f nginx-lb-svc.yaml
service "nginx-ingress-lb" created

$ kubectl get svc nginx-ingress-lb
NAME               CLUSTER-IP     EXTERNAL-IP       PORT(S)                      AGE
nginx-ingress-lb   x.x.x.x        x.x.x.x          
```

Create the nginx-controller

```console
$ kubectl create -f nginx-controller-deamonset.yaml
deployment "nginx-ingress-controller" created
```

Promote the allocated IP to a static IP.

```console
$ kubectl patch svc nginx-ingress-lb -p '{"spec": {"loadBalancerIP": "x.x.x.x"}}'
"nginx-ingress-lb" patched
```
```console
$ gcloud compute addresses create nginx-ingress-lb --addresses 104.154.109.191 --region us-central1
Created [https://www.googleapis.com/compute/v1/projects/kubernetesdev/regions/us-central1/addresses/nginx-ingress-lb].
```
## Configure Ingress to use this nginx instead of GCE

Every Ingress created with the `ingress.class` annotation set to
`nginx` will use this controller. Changing this to 'gce' will use the GCE controller
