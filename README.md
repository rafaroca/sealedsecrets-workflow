# sealedsecrets-workflow

Demonstrating a GitOps workflow with SealedSecrets.

## Installing a Kubernetes cluster

Install Minikube on your local host machine. See [Minikube installation instructions](https://minikube.sigs.k8s.io/docs/start/) for different operating systems.

	$ minikube start

You can start a UI to show all resources in your cluster with

	$ minikube dashboard

_Completely optional:_

You will need to enable port forwarding to reach the webserver running inside the cluster.
If you want to reach the cluster directly from your host, you need to enable an Ingress Controller on your cluster.
There is [extensive documentation](https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/) on how to do this on Minikube.

## Preparing the git repository for GitOps

- Fork this GitHub repository
- Clone the repository to your local machine
- Create a Personal Access Token on the [GitHub Developer Settings](https://github.com/settings/tokens). It needs the `write:public_key` scope to create a new public key for itself.

## Installing flux

- Install the `flux` command line tool as explained in the [flux installation documentation](https://fluxcd.io/docs/installation/#install-the-flux-cli).
- Bootstrap your local minikube cluster with `flux bootstrap github --owner=<your_user_name> --repository=sealedsecrets-workflow --path=cluster`

This will set up your minikube cluster with the configuration that is contained within the `cluster` path of your GitHub repository. Your cluster will periodically check for changes on GitHub.

## Create a sealed secret

- Install the `kubeseal` command line tool as explained in the [sealed secrets documentation](https://github.com/bitnami-labs/sealed-secrets/#homebrew).
- The following command creates a valid `htpasswd` file using the `openssl` command. The `kubectl create` command generates a Kubernetes secret out of the password. The secret is finally encrypted into a SealedSecret with the `kubeseal` command and saved into a file in the `cluster` directory.

```
	USERNAME=user \
    PASSWORD=weakpassword \
    HTPASSWD=$USERNAME:$(openssl passwd -1 $PASSWORD) \
    kubectl create secret generic nginx-htpasswd \
      --namespace default \
      --from-literal=htpasswd=$HTPASSWD \
      --output=yaml \
      --dry-run=client \
    | kubeseal -o yaml \
    > ./cluster/nginx-htpasswd-sealed.yaml
```

- Add the new to git, commit and push the changes to GitHub.
- Flux takes up the new changes from GitHub and the SealedSecrets controller decrypts the SealedSecret into a Kubernetes secret.
- The nginx pod now starts up.

## Connect to nginx

Set up a port forward with `kubectl port-forward service/nginx-svc 8888:80`.
Directing your browser to [http://localhost:8888](http://localhost:8888) asks for the login credentials.
Enter `user` as username and `weakpassword` as password to view the nginx default page.

# Note on security

The `htpasswd` secret in this example is purposefully never written to the repository. 
When using SealedSecrets, you have to take caution to never commit secrets to git repositories.
A pre-commit hook which scans for secrets can be a first barrier.
Secret detection in your CI/CD pipeline can alert you when secrets were published to your repository.

If a secret was pushed to a git repository, it must be considered compromised.
Rotate the secret and re-seal it.
