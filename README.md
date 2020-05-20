# Tyk Pro for Kubernetes Helm Chart - Hybrid for Istio 


This is a fork of the Tyk-Helm-Chart repository slimmed down and optimised for deploying hybrid gateways inside of Istio.


**Prerequisites**



- Redis installed in the cluster or reachable from K8s

	`helm repo add stable https://kubernetes-charts.storage.googleapis.com`
	`helm repo update`
	`kubectl create namespace tyk-redis`
	`helm install tyk-redis stable/redis -n tyk-redis`
	`(follow notes from the installation output to get connection details)`


## Install Tyk Hybrid Gateway

Before installing the Gateway ensure you have Istio installed and the default namespace enabled for Injection. mTLS can also be enabled or disabled - the operation of Tyk will be the same in both scenarios.

To install, first modify `values_hybrid.yaml` file as follows:
1. Add redis password in `redis.pass` value. It's the value of `$REDIS_PASSWORD` environment variable (the host should be `tyk-redis-master.tyk-ingress.svc.cluster.local` if you used the tyk-ingress as the namespace.
2. Add your RPC key in `tyk_k8s.org_id` value. This is the organisation ID of your Tyk Cloud Account
3. Add your API key in `tyk_k8s.dash_key` value this is the API key of a Dashboard user on your Tyk Cloud Account
4. Add your dashboard URL in `tyk_k8s.dash_url` value. If it's a Tyk SaaS account the value `https://admin.cloud.tyk.io` is already set for you.

	`helm install tyk-hybrid -f ./values_hybrid.yaml ./tyk-hybrid`
	
To check what intallation you have from helm run:
	`helm list`
	
To uninstall run:
	`helm uninstall tyk-hybrid`	

Follow the instructions in notes to install the ingress controller. Sidecar injection support is coming soon!


## Important things to remember: Nodes are Segmented

This Helm chart installs Tyk as a *segmented* Gateway service with an external load balancer, this means that the gateways that get deployed are tagged with the `ingress` tag. Tagged gateways like this will only load APIs that have also been tagged as `ingress`.

The reason gateways are sharded is so that the dashboard and the Tyk K8s controller can target different services to different gateways, i.e. services that are exposed to the internet should be routed in the `ingress` gateways, while service-mesh sidecars need to handle private service definitions which are created programatically, and should not be loaded into the public-facing gateways.

### Making an API public

You can set a tag for your exposed services in the API Designer, under the "Advanced Options" tab, the section called `Segment Tags (Node Segmentation)` allows you to add new tags. To make an API public, simply add `istio-edge` to this section, click the "Add" button, and save the API.

If you are using an ingress spec, then the Tyk k8s controller will do this for you.

### How to disable node sharding

If you are using the latest chart, you can set the `enableSharding` value in the `values.yaml` to false. This will mean all APIs are loaded on all gateways deployed. i.e. Useful if you are only deploying Edge gateways which will process all traffic.

