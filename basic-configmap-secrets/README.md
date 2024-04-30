Variable which store some value – but main thing in variable is here we give both key and value.
But in case of Environment variable here key is fixed we need to give value only.
Like if we are going to run mysql container using some provided image – developer of that image will not provide the root password . they have intergrated some hard code environment variable that if you will give value of them it will be attached as root password. So when you create the container and you didn’t provide the environment variable then your container will not run.

Suppose we are going to create a mysql container then we have to provide 5-7 environment variables and that will be very long. So we use  yaml file instead of  cli command . But in Yaml file there would be 10-14 line extra we have to write as environment variable.
Suppose for same type of application we need to give different value then that will not be good write again and again so that is not good.
Instead of that whatever key value or environment variable key file data anything you want to store you can store in kubernetes you can store that in configmap and secrets and later on pod can call this.
Configmap will store the data in plain text.
But Secrets will store the data in encoded form.
>> like if we create only like this
kubectl run mydb --image mysql 
it will give the error because we haven’t provided the environment variable.
If we don’t know why this error has come then you can check the log.
Kubectl logs mydb
2024-04-04 07:18:14+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
    You need to specify one of the following as an environment variable:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_ALLOW_EMPTY_PASSWORD
    - MYSQL_RANDOM_ROOT_PASSWORD

kubectl run mydb --image mysql --env MYSQL_ROOT_PASSWORD=redhat
now container will be in running status.
How can we verify that env variable which you have provided taken or not.
kubectl describe pod mydb
using describe we can also check 
kubectl exec -it mydb – env
we can check like this also 
kubectl exec -it mydb -- mysql -u root –p
this is the method to login the mysql database as root user here we can check after entering into mysql data base 
>> show databases
Like this if we have to give more env variable then command will be very big instead of that we can use yaml file.

To use od config map and secret we should create a file 
We have created a datafile
Put the environment variable in that.
MYSQL_ROOT_PASSWORD=qwerty
MYSQL_DATABASE=hcl
MYSQL_USER=testing
MYSQL_PASSWORD=mayank

Now we can create config map with this.
kubectl create configmap shrameshwar --from-env-file datafile
now like this we can create secret file also 
secrets are 3 type one of them is generic.
One is docker registory
One is TLS
kubectl create secret generic shrameshwar --from-env-file datafile .

using command r:~# kubectl describe cm Shrameshwar  we can see the data of configmap 
likewise we can check the secret value .
kubectl describe secrets Shrameshwar
but it will not show the real value because it is in encoded form.
kubectl get secret shrameshwar -o yaml
using this we can see the encoded value \.
To encode the value it uses by default base64 technique.
We can decode the value like.
echo dGVzdGluZw== | base64 –d
Now if we want to use configmap and secrets in my container pod then we can use like this.

We will create a pod in that we will mention envFrom configMapRef
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mydbcon
  name: mydbcon
spec:
  containers:
  - envFrom:
    - configMapRef:
        name: shrameshwar
    image: mysql
    name: mydbcon
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {} 

to create pod using secret created previously.
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mydbsecret
  name: mydbsecret
spec:
  containers:
  - envFrom:
    - secretRef:
       name: shrameshwar
    image: mysql
    name: mydbsecret
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
like previous steps we can check values are taken or not.
Here from secrets and congigmap it takes whole value  if you don’t want all these value then you can use like this 
env:
    - name: SECRET_KEY
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: secret-key

