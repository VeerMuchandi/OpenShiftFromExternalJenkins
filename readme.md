## Deploying to OpenShift from External Jenkins

OpenShift provides a Jenkins Container as the CICD tool to run on an OpenShift cluster. You can use this tool to set up your CICD pipelines in order to deploy applications to an OpenShift cluster or elsewhere. However, we do come across cases where there is desire to run Jenkins outside OpenShift. Perhaps some organizations have invested in CICD infrastructure that they want to reuse. Some organzations want to stick to their current infrastructure and eventually move to Jenkins running on OpenShift in the future. 

There are a few ways to use such external Jenkins to deploy applications to OpenShift:
1. You can install `oc` command line tool on the external Jenkins server and use `oc` mechanism to deploy applications to OpenShift
2. You can use REST APIs to interact with OpenShift
3. You can use OpenShift plugin to deploy applications to OpenShift

In this article, we will focus on the third option.

**Step 1: Install Pipeline Plugin** 

On your Jenkins console, go to `Manage Jenkins` option and `Manage Plugins`.

Search for [OpenShift Pipeline Jenkins Plugin](https://wiki.jenkins-ci.org/display/JENKINS/OpenShift+Pipeline+Plugin) in the `Available` tab and install it.

**Note:** Make sure that the Jenkins [Pipeline](https://wiki.jenkins-ci.org/display/JENKINS/Pipeline+Plugin) plugin, is installed. 

Now we are ready to set up a pipeline.

**Step 2: Setup a pipeline**

Now go to Jenkins Home and add a `New Item` on the left menu.

Input a value for the field `Enter an item name` and choose `Pipeline` and press `OK`

On the next page	

* (Optionally,) add a `Description`		
* Check the box `This project is parameterized`. It will display the `Add Parameter` button. Add the following parameters	
	* String parameter named `KUBERNETES_SERVICE_HOST` and set the Default Valu e as your openshift host url.
	* Boolean parameter named `SKIP_TLS` checked by default
	* Password parameter named `AUTH_TOKEN` set to your token value. You can login to the same openshift server using `oc login` on your workstation and run `oc whoami -t` to get the token value to use here

* Scroll down to the end and add the pipeline script in the `Pipeline` section. For this exercise, we will use the following example script

```
node('') {
stage 'buildInDevelopment'
openshiftBuild(namespace: 'development', buildConfig: 'myapp', showBuildLogs: 'true')
stage 'deployInDevelopment'
openshiftDeploy(namespace: 'development', deploymentConfig: 'myapp')
openshiftScale(namespace: 'development', deploymentConfig: 'myapp',replicaCount: '2')
}
```
This script has two stages. In the `buildInDevelopment` stage, it starts a build by invoking the build configuration named `myapp` in the project named `development` using the `openshiftBuild` step.

In the `deployInDevelopment` stage, it deploys using the deployment configuration named `myapp` using the `openshiftDeploy` step. Then scales up the application to 2 replicas using the `openshiftScale` step. 

The documentation about these steps from the OpenShift Jenkins Pipelines plugin is available [here](https://github.com/jenkinsci/openshift-pipeline-plugin#jenkins-build-steps). Please explore other possibilities.

**Step 3: Create the build and deployment configurations**

Since we are trying to use build and deployment configurations in the project named `development` from the pipeline, we will have to first create those.

* Create a project named `development` from web console
* Add to project an application (Example: using PHP latest)
* Name the application as `myapp`
* Provide a git url for source code (Example : [https://github.com/openshift/cakephp-ex.git](https://github.com/openshift/cakephp-ex.git))	
* Click on `advanced options`
* **Uncheck** the three options under `Build Configuration`
	* `Configure a webhook build trigger`
	* `Automatically build a new image when the builder image changes`
	* `Launch the first build when the build configuration is created`
* **Uncheck** the two `Autodeploy` options under `Deployment Configuration` 
	* `New image is available`
	* `Deployment configuration changes`

**Step 4: Start pipeline**

Now go back to Jenkins and start pipeline.
Observe build, deployment and scale up on the OpenShift Webconsole while the pipeline stages are executed in the Jenkins console.


###Summary
In this article we learnt to set up a external jenkins to talk to an OpenShift cluster using Jenkins Pipeline plugin and use it.
