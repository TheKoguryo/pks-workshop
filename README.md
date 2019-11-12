# LAB 1: Create a PKS Cluster
You have been distributed the following:
* PKS API Endpoint
* PKS Username
* PKS Password

## 1. [Install the PKS CLI](https://docs.pivotal.io/pks/1-5/installing-pks-cli.html)
* _If you don't yet have access to Pivotal Network, you can download the binaries from the `pivnet/pks-cli` folder within this repo, but still refer to the instructions above to place the CLI in the correct path for your workstation._

Verify the pks CLI install
```
pks help
```

## 2. [Install the Kubernetes CLI](https://docs.pivotal.io/pks/1-5/installing-kubectl-cli.html)
* _If you don't yet have access to Pivotal Network, you can download the binaries from the `pivnet/kubectl-cli` folder within this repo, but still refer to the instructions above to place the CLI in the correct path for your workstation._

Verify the kubectl CLI install
```
kubectl help
```

## 3. Create a PKS cluster
Login to PKS
```
pks login -a <PKS API Endpoint> -k -u <PKS Username> -p <PKS Password>
```

Create a cluster - **Replace `<N>` with your unique id for this workshop**
```
pks create-cluster workshop<N> -e workshop<N>.clusters.aws.cfscale.com -p small --wait
```

This will take 5-10 minutes, when create status is "Succeeded" continue

List all clusters - you should only see your own
```
pks clusters
```

Get the kubernetes credentials to access the cluster
```
pks get-credentials workshop<N>
```

Get the cluster info to verify cluster connectivity
```
kubectl cluster-info
```

# Lab 2: Deploy a Pod

Run a new pod with the nginx image
```
kubectl run nginx --image=nginx --generator=run-pod/v1
```

View the running Pods (re-run the command until the Status is "Running")
```
kubectl get pods
```

Describe the details of the nginx pod deployment
```
kubectl describe pod nginx
```

# Lab 3: Deployments
Delete the existing nginx pod from the last Lab
```
kubectl delete pod nginx
```

Create a new nginx deployment, using nginx version 1.16.1
```
kubectl create deployment nginx --image=nginx:1.16.1
```

Verify the Deployment, ReplicaSet and Pod were created
```
kubectl get deployment
kubectl get replicaset
kubectl get pod
```

Or get all at the same time
```
kubectl get all
```

Scale out the number of Pod replicas in the deployment
```
kubectl scale deployment/nginx --replicas=3
```

Delete a Pod - notice the ReplicaSet will create a new one a few seconds after deletion
```
# Note: the alpha-numeric value of the Pod name will be different for your deployment
kubectl delete pod nginx-8698bd8c77-fdh2r 
```

View the rollout history for the deployment
```
kubectl rollout history deployment nginx
```

Update the nginx image on the deployment to trigger a new rollout
```
kubectl set image deployment/nginx nginx=nginx:1.17.4 --record
```

View the rollout history for the deployment (again)
```
kubectl rollout history deployment nginx
```
View the deployment to verify the new image version
```
kubectl get deployment nginx -o wide
```

View all - note the old ReplicaSet at 0, and the new one at 3
```
kubectl get all
```

# Lab 4: Services

Create a service to expose the nginx deployment
```
kubectl expose deployment nginx --name=nginx-service --type=NodePort --port=80
```

Verify the label selector on the nginx-service
```
kubectl get service nginx-service -o wide
kubectl describe service nginx-service
```

Use port-forward to make the service accessible (as the worker nodes on our cluster are not reacheable over the internet)
```
kubectl port-forward service/nginx-service 8080:80
```

Follow the pod logs for the deployment in a second terminal window 
```
kubectl logs deployment/nginx -f
```

Open a browser to http://localhost:8080

Once all is verified you can kill the port-forward and logs commands

Launch a second Pod using the alpine image and override the start command for the docker image. The start command is to sleep, thus allowing the Pod to live for 3600 seconds, which will allow us to get into the Pod and poke around for a while
```
kubectl run alpine --image=alpine --generator=run-pod/v1 -- /bin/sh -c "sleep 3600"
```

Get a shell into the running alping container
```
kubectl exec alpine -i --tty -- sh
```

Within the alpine shell execute 
```
/ # env | sort                       # Note the environment variables that have been set for the nginx-service

/ # nslookup nginx-service           # This will resolve to the ClusterIP assigned to the service

/ # apk add --no-cache curl          # Install curl

/ # curl http://nginx-service        # Request the nginx home page via DNS

/ # curl http://$NGINX_SERVICE_SERVICE_HOST      # Request the nginx home page via env var

/ # exit                             # Exit the container shell
```

Delete the alpine pod
```
kubectl delete pod alpine
```
