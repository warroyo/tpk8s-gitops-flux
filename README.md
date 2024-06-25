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
* [secretgen controller](https://github.com/carvel-dev/secretgen-controller), this is installed by TMC automatically.  


### Deploy the controller. 

### Create the kubeconfigs for UCP

We will be using secretgen controller to handle the kubeconfig creation and also the distribution of the kubecofnigs to the correct namespaces



### Deploy a space to TPK8s




## Building the token generator

this isn't necessary unless you are doing dev.

```bash
tanzu build config --containerapp-registry ghcr.io/{contact.team}/{name}
cd token-generator
tanzu build -r container-image
```