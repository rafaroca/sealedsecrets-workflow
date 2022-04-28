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
There is [extensive documentation](https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/) on how to do this on Minikube.

## Installing flux

Install the `flux` command line tool as explained in the [flux installation documentation](https://fluxcd.io/docs/installation/#install-the-flux-cli).

- Clone this GitHub Repository
- Create a Personal Access Token on the [GitHub Developer Settings](https://github.com/settings/tokens). It needs the `write:public_key` scope to create a new key for itself.
- Install flux with `flux bootstrap github --owner=<your_user_name> --repository=sealedsecrets-workflow --path=cluster`

## Create a sealed secret

Install the `kubeseal` command line tool as explained in the [sealed secrets documentation](https://github.com/bitnami-labs/sealed-secrets/#homebrew).

- Seal the htpasswd secret with `kubeseal -o yaml <nginx-htpasswd.yaml >cluster/nginx-htpasswd-sealed.yaml` 
- Commit the new file to git and push the changes to GitHub.
- Flux takes up the new changes from GitHub and the SealedSecrets controller decrypts the SealedSecret into a Kubernetes secret.
- The nginx pod now starts up

## Connect to nginx

Set up a port forward with `kubectl port-forward service/nginx-svc 8888:80`.
Directing your browser to [http://localhost:8888](http://localhost:8888) asks for the login credentials.
Enter `user1` as username and as password to view the nginx default page.

# Note on security

The `nginx-htpasswd.yaml` file is committed to the repository for demonstration only.
A real usage would only ever contain the sealed `nginx-htpasswd-sealed.yaml` file.
If a secret was ever pushed to a git repository, it must be considered as compromised.
Rotate the secret and re-seal it.

When using SealedSecrets, you have to take caution to never commit secrets to git repositories.
A pre-commit hook which scans for secrets can be a first barrier.
Also use secret detection in your CI/CD pipeline.

