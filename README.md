Lets Check how to run Jenkins-slaves in kubernetes cluster.


 For that first need to install **Kubernetes** plugin ((https://plugins.jenkins.io/kubernetes/)) for Jenkins, it helps to run agents in the Kubernetes cluster. Install the plugin from Manage Jenkins -> Manage Plugins.

Lets Check how to setting up the Jenkins agent.

**Create credentials set for the Jenkins master to access the Kubernetes cluster.**
  1. In the Jenkins UI, click the Credentials link in the left-hand navigation panel
  2. Click on the Jenkins under the Stores scoped to Jenkins.
  3. On the next page select Global Crednetails (unrestricted)
  4. Click Add CredentialsUnder Kind, specify Kubernetes Service Account
  5. Leave the scope set to Global
  6. Click OK.

This allows the Jenkins master to use a Kubernetes service account to access the Kubernetes API.

**Create a cloud configuration on the Jenkins Master.**
  1. In the Jenkins UI, go to Manage Jenkins -> Configure System -> Cloud -> Kubernetes
  2. Update the following parameters:
     * Name: <your choice> - This defaults to kubernetes
     * Kubernetes URL: https://kubernetes.default - This was automatically configured from the service account.
       `kubectl cluster-info | grep master`
     * Kubernetes Namespace: default - Unless you are running your master in another namespace
     * Credentials: Select the Kubernetes Service Account credentials you created in the previous step
     * Jenkins URL: http://<your_jenkins_hostname>
     * Jenkins tunnel: <jenkins_service>:50000 - This is the port that is used to communicate with an agent.
Refer the Image:
To test this connection is successful you can use the Test Connection button to ensure there is proper communication from Jenkins to the Kubernetes cluster. These parameters are using to launch an agent in the K8s cluster. You can certainly modify other parameters to tweak your environment.
 
Now that you’ve configured your Jenkins master so that it can access your K8s cluster, it’s time to define some pods. A pod is the basic building block of Kubernetes and consists of one or more containers with shared network and storage. Each Jenkins agent is launched as a Kubernetes pod. It will always contain the default JNLP container that runs the Jenkins agent jar and any other containers you specify in the pod definition. If you have some custom requirements for your slaves, you can create a custom docker image.

There are at least two ways to configure pod templates – in the **Jenkins UI** and in your **pipeline script**.

**Configure a Pod Template in the Jenkins UI**
 Kubernetes Pod Template section, we need to configure the image that will be used to spin up the agent pod.
 1. In the Jenkins UI, go to Manage Jenkins → Configure System -> Cloud -> Kubernetes ->  
 2. Click the Add Pod Template button and select Kubernetes Pod Template
 3. Enter values for the following parameters:
   * Name: <your choice>
   * Namespace: default - unless you configured a different namespace in the previous step
   * Labels: <your choice> - this will be used to identify the agent pod from your Jenkinsfiles
   * Usage: Select "Use this node as much as possible" if you would like for this pod to be your default node when no node is specified. Select "Only build jobs with label matching expressions matching this node" to use this pod only when its label is specified in the pipeline script
   * Containers: The containers you want to launch inside this pod. 
     * Containers -> Add Containers -> Containers Template
     * Name: jnlp
     * Docker Image: jenkins/agent  it will be used as a reference to spin up a new Jenkins slave
     * Command to run :
     * Arguments to pass to the command:
     * Allocate pseudo-TTY: yes
   *
 
 
