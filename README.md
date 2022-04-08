# sealedsecrets-workflow

Showing how to work with SealedSecrets in a GitOps environment.

## Installing a Kubernetes cluster

Install Minikube on your local host machine. See [Minikube installation instructions](https://minikube.sigs.k8s.io/docs/start/) for different operating systems.

	$ minikube start

You can start a UI to show all resources in your cluster with

	$ minikube dashboard

_Completely optional:_

You will need to enable port forwarding to reach the webserver running inside the cluster.
If you want to reach the cluster directly from your host, you need to enable an Ingress Controller on your cluster.
There is an [extensive documentation](https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/) on how to do this on Minikube.

## Installing flux

Install the flux command line tool as explained in the [flux installation documentation](https://fluxcd.io/docs/installation/#install-the-flux-cli).

Clone this GitHub Repository, so that the `flux bootstrap` command may write to it.

