Lets Check how to run Jenkins-slaves in kubernetes cluster.


  **Kubernetes** plugin ((https://plugins.jenkins.io/kubernetes/)) for Jenkins helps to run agents in the Kubernetes cluster. Install the plugin from Manage Jenkins > Manage Plugins.
For each build, Jenkins agent launched a Kubernetes pod. It will always contain the default JNLP container(jenkins/jnlp-slave) that runs the Jenkins agent jar and any other 
containers you specify in the pod definition. 

Lets Check how to setting up the Jenkins agent.

1. Need to create credentials set for the Jenkins master to access the Kubernetes cluster.  
