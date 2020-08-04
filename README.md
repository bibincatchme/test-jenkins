Lets Check how to run Jenkins-slaves in kubernetes cluster.


 For that first need to install **Kubernetes** plugin ((https://plugins.jenkins.io/kubernetes/)) for Jenkins, it helps to run agents in the Kubernetes cluster. Install the plugin from Manage Jenkins > Manage Plugins.
For each build, Jenkins agent launched a Kubernetes pod. It will always contain the default JNLP container(jenkins/jnlp-slave) that runs the Jenkins agent jar and any other 
containers you specify in the pod definition. 

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
  1. In the Jenkins UI, go to Manage Jenkins → Configure System -> Cloud -> Kubernetes
  2. Update the following parameters:
     * Name: <your choice> - This defaults to kubernetes
     * Kubernetes URL: https://kubernetes.default - This was automatically configured from the service account. 
       kubectl cluster-info | grep master
     * Kubernetes Namespace: default - Unless you are running your master in another namespace
     * Credentials: Select the Kubernetes Service Account credentials you created in the previous step
     * Jenkins URL: http://<your_jenkins_hostname>
     * Jenkins tunnel: <jenkins_service>:50000 - This is the port that is used to communicate with an agent.
