# Using Flux to deploy to Tanzu Platform


## Flux Native way

Flux has the ability to specify a kubeconfig in a kustomization. The Tanzu platform kubeconfigs use the `exec` function to get a token since this token expires often the command it runs will generate a new one when needed. To use this with flux two things are needed to be done. 

1. enable the flux kustomize [controller argument](https://fluxcd.io/flux/components/kustomize/options/) that allow `exec` to be run. `--insecure-kubeconfig-exec` . 
2. build a custom image for flux that has the cli tools you need to execute.

There is a security risk with enabling the exec functionality in flux. This also requires that you have full control over the flux install.

## The other way

If you have a managed provider that installs your flux controllers like Tanzu platform then you won't want to deal with hacking the above to work. This solutions uses the OOTB flux controllers and some extra tools to handle connecting to the tanzu platform. It also does not require the `--insecure-kubeconfig-exec` flag. There are multiple ways this could be implemented , this was was chosen in order to reduce custom work and re-use as much existing tooling as possible. 

## Sample implementation

This example is not meant to show how to do gitops with flux. This is setup in a very minimal so it can be easily understood.

Pre-reqs:
* TMC 
* TMC managed flux
* [externla-secrets]()


### Deploy the controller. 

This could easily be done by flux but for simplicity we are just deploying this via the command line.

1. create the secret needed for the controller. this should be your CSP token.

```bash
cat <<'EOF' >deploy/token-generator-deploy/generator.env
CSP_TOKEN=<your-token>
EOF
```

2. deploy the controller into the cluster you plan to use for gitops with flux. Ideally this is a cluster managed by TMC with flux enabled. 

```bash
kubectl apply -k deploy/token-generator-deploy              
```



### Create the kubeconfigs for UCP

We will be using external secrets operator to handle the kubeconfig creation and also secretgen controller to handle the distribution of the kubeconfigs to the correct namespaces. This could also easily be managed via flux but for simplicity we are just using the command line. Thsi example just sets up the project context but this could be done for space or cluster group the same way.

1. get the server url from the current context on the cli. this is used for our in cluster kubecofnig secret

```bash
#get the context url from your local cli

tanzu project use AMER-West
export KUBECONFIG=~/.config/tanzu/kube/config
kubectl config view --minify -o json | jq -r '.clusters[0].cluster.server'
```

2. Update the `deploy/secret-templates/project-context.yml` to use your server url.

3. create the secret store and templated secret.

```bash
kubectl apply -f deploy/secret-templates
```

### Deploy a space to TPK8s

Since the purpose of this is to deploy objects to UCP using gitops for this one we will be using flux for the deployment. 

1. create the flux kustomization and git repo to point to the sample repo and directory.





## Building the token generator

this isn't necessary unless you are doing dev.

```bash
tanzu build config --containerapp-registry ghcr.io/{contact.team}/{name}
cd token-generator
tanzu build -r container-image
```