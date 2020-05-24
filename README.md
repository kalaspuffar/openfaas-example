# openfaas-example
Small example to create a function on minikube

First we need to download docker tools, minikube, kubectl, helm and put them in our path.

```
https://github.com/docker/toolbox/releases
https://kubernetes.io/docs/tasks/tools/install-minikube/
https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows
https://github.com/helm/helm/releases
```

Putting your installation in the path on windows and on my system
they are in the tools directory so I will need to run the command
below.
```
set PATH=%PATH%;c:\tools
```

First we need to start minikube so we have the system created and running.
```
minikube start
```

Installing openfaas on kubernetes starts by creating a tiller service access rule and connecting it to the cluster. After that we need to setup the namespace for the pods to install into.
```
kubectl -n kube-system create sa tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
```

Next up we configuret the helm repository.
```
helm repo add openfaas https://openfaas.github.io/faas-netes/
```

Creating a secret in your system will decouple the security tokens from the actual installation. Here we setup the basic authentication user and password information.
```
kubectl -n openfaas create secret generic basic-auth --from-literal=basic-auth-user=admin --from-literal=basic-auth-password="YOUR_PASSWORD"
```

Next we install openfaas telling the system to use the basic auth information.
```
helm upgrade openfaas --install openfaas/openfaas --namespace openfaas --set functionNamespace=openfaas-fn --set basic_auth=true

We need to configure the client tooling by login into our new installation. Password sent via the terminal standard in is recommened.
```
minikube ip
SET OPENFAAS_URL={MINIKUBE_IP}:31112
echo "YOUR_PASSWORD" | faas-cli login --gateway http://%OPENFAAS_URL% -u admin --password-stdin
```

Waiting for the pods to start we can look at the progress using the get pods command below.
```
kubectl get pods -n openfaas
```

Now we should be able to see our installation in the web browser using http://{MINIKUBE_IP}:31112.

Next we need to setup the docker enviroment using the commands below to get the enviroment information we need to set and create a registry we can publish our images to.
```
minikube docker-env
docker run -d -p 5000:5000 --restart always --name registry registry:2
```

To simplify the creation of a new function we can use templates. The commands below ensure that we have the template for java11 installed. Other templates can be downloaded in a similar maner.
```
faas-cli template pull
faas-cli template store list
faas-cli template store pull java11
```

Checking which templates we can use a handy list command is available. Then we simply create a new function using the language and adding our repository address as a prefix in order to publish our images. Otherwise the image will be published to Docker hub and you need your username as a prefix.
```
faas-cli new --list
faas-cli new java-fn --lang java11 --prefix="localhost:5000"
```

Next we can build our newly created function with either doing a up command that will run the build, push and deploy commands in sequence or you can run them seperatly in order to handle any errors to each command that might occur.
```
faas-cli up --yaml java-fn.yml

faas-cli build  --yaml java-fn.yml
faas-cli push --yaml java-fn.yml
faas-cli deploy --yaml java-fn.yml
```

Now we can look in either the browser or use the list command to see if our function was deployed correctly.
```
faas-cli list
```
