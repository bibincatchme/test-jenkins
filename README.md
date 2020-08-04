## How to run Jenkins-slaves pods in kubernetes cluster.


For that first need to install **Kubernetes** plugin ((https://plugins.jenkins.io/kubernetes/)) for Jenkins, it helps to run agents in the Kubernetes cluster. Install the plugin from Manage Jenkins -> Manage Plugins.

Lets Check how to setting up the Jenkins agent.

**Create Secret credentials set for the Jenkins master to access the Kubernetes cluster.**
  1. In the Jenkins UI, click the Credentials link in the left-hand navigation panel
  2. Click on the Jenkins under the Stores scoped to Jenkins.
  3. On the next page select Global Crednetails (unrestricted)
  4. Click Add Credentials Under Kind, specify Kubernetes Service Account
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
  
Refer the Image: cloud-configuration
 
To test this connection is successful you can use the Test Connection button to ensure there is proper communication from Jenkins to the Kubernetes cluster. These parameters are using to launch an agent in the K8s cluster. You can certainly modify other parameters to tweak your environment.
 
Now that you’ve configured your Jenkins master so that it can access your K8s cluster, it’s time to define some pods. A pod is the basic building block of Kubernetes and consists of one or more containers with shared network and storage. Each Jenkins agent is launched as a Kubernetes pod. It will always contain the default JNLP container that runs the Jenkins agent jar and any other containers you specify in the pod definition. If you have some custom requirements for your slaves, you can create a custom docker image.

There are at least two ways to configure pod templates – in the **Jenkins UI** and in your **Pipeline script**.

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
      * Name: jnlp - because Jenkins always creates a container called jnlp and executes jobs.
      * Docker Image: jenkins/agent  it will be used as a reference to spin up a new Jenkins slave
      * Command to run :
      * Arguments to pass to the command:
      * Allocate pseudo-TTY: yes
 
 Refer the Image: pod-template-configuration
 
 Create a Jenkins pieline and build with the below code, it will create the slave pod and echo hello

 node('jenkins-slave') {
    
     stage('Run shell') {
        echo "Hello World"
    }
}
 
 It will create the slave pod with prefix ‘jenkins-slave’. Once the build complated it terminate the slave pods.
 You can see the echo "Hello World" in the build output.
 
**Configure a Pod Template with Pipeline script** 

Any of the configuration parameters available in the UI or in the YAML definition can be added to the podTemplate and containerTemplate sections.
Nodes can be defined in a pipeline and then used, however, default execution always goes to the jnlp container. You will need to specify the container you want to execute your task in. 
The default jnlp agent image used can be customized by adding it to the template

Please note the POD_LABEL is a new feature to automatically label the generated pod in versions 1.17.0 or higher, older versions of the Kubernetes Plugin will need to manually label the podTemplate

This will run in jnlp container
```
podTemplate {
    node(POD_LABEL) {
        stage('Run shell') {
            sh 'echo hello world'
        }
    }
}
```
In the example below, I’ve defined a pod with two container templates. Ports in each container can be accessed as in any Kubernetes pod, by using localhost.
The container statement allows to execute commands directly into each container.

```
podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'git', image: 'alpine/git', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]
  ) {
    node('mypod') {
        stage('Clone repository') {
            container('git') {
                sh 'git --version'
            }
        } 
        stage('Check running containers') {
            container('docker') {
                sh 'docker --version'
            }
        }
      }
    
    }
```

Output of Jenkins UI Method
```
Started by user bibin
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Still waiting to schedule task
‘Jenkins’ doesn’t have label ‘jenkins-slave’
Agent jenkins-slave-380nq is provisioned from template jenkins-slave
---
apiVersion: "v1"
kind: "Pod"
metadata:
  labels:
    jenkins: "slave"
    jenkins/label: "xjenkins-slave"
  name: "jenkins-slave-380nq"
spec:
  containers:
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_TUNNEL"
      value: "jenkins5000:50000"
    - name: "JENKINS_AGENT_NAME"
      value: "jenkins-slave-380nq"
    - name: "JENKINS_NAME"
      value: "jenkins-slave-380nq"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://54.91.140.176:32535/"
    image: "jenkins/jnlp-slave"
    imagePullPolicy: "IfNotPresent"
    name: "jnlp"
    resources:
      limits: {}
      requests: {}
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"
  hostNetwork: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on jenkins-slave-380nq in /home/jenkins/agent/workspace/test-jnllp
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Run shell)
[Pipeline] echo
Hello World
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```


```
Started by user bibin
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Still waiting to schedule task
‘Jenkins’ doesn’t have label ‘test_2-2wg9h’
Created Pod: jenkins/test-2-2wg9h-t0fbr-nj3j5
[Normal][jenkins/test-2-2wg9h-t0fbr-nj3j5][Scheduled] Successfully assigned jenkins/test-2-2wg9h-t0fbr-nj3j5 to ip-192-168-111-104.ec2.internal
[Normal][jenkins/test-2-2wg9h-t0fbr-nj3j5][Pulling] Pulling image "jenkins/inbound-agent:4.3-4"
[Normal][jenkins/test-2-2wg9h-t0fbr-nj3j5][Pulled] Successfully pulled image "jenkins/inbound-agent:4.3-4"
[Normal][jenkins/test-2-2wg9h-t0fbr-nj3j5][Created] Created container jnlp
[Normal][jenkins/test-2-2wg9h-t0fbr-nj3j5][Started] Started container jnlp
Agent test-2-2wg9h-t0fbr-nj3j5 is provisioned from template test_2-2wg9h-t0fbr
---
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://54.91.140.176:32535/job/test/2/"
    runUrl: "job/test/2/"
  labels:
    jenkins: "slave"
    jenkins/label: "test_2-2wg9h"
  name: "test-2-2wg9h-t0fbr-nj3j5"
spec:
  containers:
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_TUNNEL"
      value: "jenkins5000:50000"
    - name: "JENKINS_AGENT_NAME"
      value: "test-2-2wg9h-t0fbr-nj3j5"
    - name: "JENKINS_NAME"
      value: "test-2-2wg9h-t0fbr-nj3j5"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://54.91.140.176:32535/"
    image: "jenkins/inbound-agent:4.3-4"
    name: "jnlp"
    resources:
      requests:
        cpu: "100m"
        memory: "256Mi"
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on test-2-2wg9h-t0fbr-nj3j5 in /home/jenkins/agent/workspace/test
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Run shell)
[Pipeline] sh
+ echo hello world
hello world
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS
```
