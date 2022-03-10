# kubernetes-jenkins-deployment
This repo is usded to deploy using kubernetes plugin on jenkins
The minikube is on aws account


Reference:
https://geekdudes.wordpress.com/2020/01/03/minikube-configure-jenkins-kubernetes-plugin/


kubectl apply account.yml
kubectl get secrets
OUTPUT
NAME                  TYPE                                  DATA   AGE
default-token-5nx5s   kubernetes.io/service-account-token   3      6h39m
jenkins-token-7l68m   kubernetes.io/service-account-token   3      19m

kubectl describe secret/jenkins-token-7l68m
OUTPUT
Name:         jenkins-token-7l68m
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: jenkins
              kubernetes.io/service-account.uid: a0cc03af-61c3-49b2-a5fe-cfe00c90e34a

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1111 bytes
namespace:  7 bytes
token:      The Token was removed from here for security purposes

cd .kube/
cat config
OUTPUT
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/ubuntu/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 22 Feb 2022 15:20:15 UTC
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: cluster_info
    server: https://ip-masked-here:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Tue, 22 Feb 2022 15:20:15 UTC
        provider: minikube.sigs.k8s.io
        version: v1.25.1
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/ubuntu/.minikube/profiles/minikube/client.crt
    client-key: /home/ubuntu/.minikube/profiles/minikube/client.key
    
We’ll need server and client-certificate value


server: https://ip-masked-here:8443
client-certificate: /home/ubuntu/.minikube/profiles/minikube/client.crt
client-key: /home/ubuntu/.minikube/profiles/minikube/client.key
certificate-authority: /home/ubuntu/.minikube/ca.crt


In Jenkins click on Manage Jenkins – Manage Credentails -> Stores scoped to Jenkins -> Clieck on Jenkins 
Under "Global credentials (unrestricted)" click Add Credentails
Choose:
Kind: secret text

Secret: token string (output of kubectl describe secrets/jenkins-token-7l68m)

Now Configuring Kubernetes plugin
Jenkins – manage Jenkins – Configure system scroll to bottom and in Add a new cloud, select Kubernetes



Kubernetes URL: Is the server from config file (https://ip-masked-here:8443)
Kubernetes server certificate key: value certificate-authority from config file ("/home/ubuntu/.minikube/ca.crt")
Credentials: credentials created in previous step.Click on “Test Connection” tab and you should get Connection test successful 

We now can reference Jenkins secret into pipeline
stage('Deployment using jenkins kubernetes plugin') {
           steps {
               withCredentials([
                   string(credentialsId: 'aws-minikube-token', variable: 'api_token')
                   ]) {
                    sh 'kubectl --token $api_token --server https://ip-masked-here:8443 --insecure-skip-tls-verify=true apply -f k8s/deployment-manifest.yml'
               }
            }
       }
