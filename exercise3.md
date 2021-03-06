# Exercise 3: Building your own chart

Now that we know how to install/upgrade/delete and rollback a release from a 3rd party chart, it is time to build our own.

## The helm-echoserver chart

We will build a simple helm chart for a [helm-echoserver](https://github.com/Praqma/helm-echoserver). The chart will deploy a helm-echoserver container and perform a small post-installation task. This task is to wait for a configurable amount of time and then make a POST request to the helm-echoserver with a message to be stored in the file system of the helm-echoserver container.

A set of static kubernetes manifests are provided for guidance in [helm-echoserver](helm-echoserver) directory.

---

> Whenever the text below mentions the `provided` files, it refers to the files provided in helm-echoserver/ in this repo.

> You can run `helm lint ` after each step below to detect some of the common syntax errors.


## Create a new helm chart


1. Start by creating a new chart named `echoserver` by running `helm create echoserver`. This will create some default templates. 
2. Now, in the generated chart, edit the `Chart.yaml`file. Hint: check [here](https://docs.helm.sh/developing_charts/#the-chart-yaml-file) for the possible options.
3. Render the output from the chart templates by running `helm install echoserver --dry-run --debug --tiller-namespace <your-namespace>`


### Customizing the new chart

Now, we want to customize the default chart to our needs. 

1. Replace the values file in the new chart with the [provided](helm-echoserver/values.yaml) one. Now you have your input parameters.

#### Customizing the deployment
We want to customize the deployment template in the chart so that it can render deployment manifest similar to the provided one.

1. Inspect the name of the deployment. Try to understand how it's generated.

2. Similarly, inspect the labels. Which built-in helm objects are used?

3. On line 11 in deployment.yaml, `replicaCount` is read from input. Make it such that it defaults to 2 replicas when there is no user input for that value. **Hint**: use pipes and check the [Go Sprig functions](http://masterminds.github.io/sprig/)

4. Replace the hard-coded container port in line 28 with a Go template reading input from values.

5. In the `container` section, add environment variables such that an infinite number of environment variables can be defined by the user in the values file and get rendered in the deployment manifest. In addition to the user-provided variables a set of variables have to be added as well. Copy these from the provided [deployment.yaml](helm-echoserver/deployment.yaml). **Hint**: Use a loop!

6. Replace the hard-coded values of the environment variables [__HELM_CHART_NAME__, __HELM_CHART_VERSION__, __HELM_RELEASE__] in deployment.yaml with values from the helm built-in objects.


#### Customizing service.yaml

1. Similar to the deployment, replace hard-coded values with dynamic ones and follow the comments in the provided service.yaml file.

2. Make sure the service is created only if the service section in the values file is not empty. 

#### Customizing ingress.yaml

1. Again, follow the instructions in the comments in the provided ingress.yaml to make it dynamic.

#### Customizing  post_install_job.yaml

This job is intended to be run each time helm-echoserver is deployed/upgraded. It simply sends a curl request posting a text message.

1. As before, follow the comments in the provided `post_install_job.yaml` to make the job dynamic.

2. Now, let's make this a Helm life cycle hook. Check [life cycle hooks](https://helm.sh/docs/topics/charts_hooks/#writing-a-hook)


### Adding templates/NOTES.txt

1. Add some message to the user with dynamic content. Example: how the user can access the app in the browser based on the service configurations they choose. **Hint**: get inspired from the [Jenkins chart](https://github.com/kubernetes/charts/blob/master/stable/jenkins/templates/NOTES.txt)

----

## Building the chart

```
helm package echoserver/.
```
If no errors are found, the above command generates the compressed chart `<chart-name>-<chart-version>.tgz`

3. Horray! Our chart is ready. Let's install :

```
helm install --name echoserver echoserver-0.1.0.tgz --set post_install.delay=50,post_install.message="<your custom message here>" --tiller-namespace <your-namespace> --wait
```

> The `--wait` forces any post-install hooks to wait for all other resources to be in ready state.
> The `post_install.delay=50` defines how long the post-install pod will wait (in seconds) before sending the message.

If all goes well, the last command results in a list of resources created by the chart and instructions to access the helm-echoserver (these instructions come from echoserver/templates/NOTES.txt).

## Clean up

Before we move to the next exercise, let's clean our cluster by deleting all the releases we installed so far:

```
helm delete --purge $(helm list --all -q)
```