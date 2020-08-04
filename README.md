Lets Check how to run Jenkins-slaves in kubernetes cluster.


 For that first need to install **Kubernetes** plugin ((https://plugins.jenkins.io/kubernetes/)) for Jenkins, it helps to run agents in the Kubernetes cluster. Install the plugin from Manage Jenkins > Manage Plugins.
For each build, Jenkins agent launched a Kubernetes pod. It will always contain the default JNLP container(jenkins/jnlp-slave) that runs the Jenkins agent jar and any other 
containers you specify in the pod definition. 

Lets Check how to setting up the Jenkins agent.

### Create credentials set for the Jenkins master to access the Kubernetes cluster.  
  1. In the Jenkins UI, click the Credentials link in the left-hand navigation panel
  2. Click on the Jenkins under the Stores scoped to Jenkins.
  3. On the next page select Global Crednetails (unrestricted)
  4. Click Add CredentialsUnder Kind, specify Kubernetes Service Account
  5. Leave the scope set to Global
  6. Click OK.
