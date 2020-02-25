#  Rolling Updates with Kubernetes Deployments 
  
## Additional Information and Resources   

Your company's developers have just finished developing a new version of their candy-themed mobile game. They are ready to update the backend services that are running in your Kubernetes cluster. There is a deployment in the cluster managing the replicas for this application. The deployment is called candy-deployment. You have been asked to update the image for the container named candy-ws in this deployment template to a new version, ```linuxacademycontent/candy-service:3```.  
  
After you have updated the image using a rolling update, check on the status of the update to make sure it is working. If it is not working, perform a rollback to the previous state.  


### Execution  

#### Perform a rolling update of the container version  

Update the deployment to the new version like so:  

```kubectl set image deployment/candy-deployment candy-ws=linuxacademycontent/candy-service:3 --record```  

Check the progress of the rolling update:  

```kubectl rollout status deployment/candy-deployment``` 

If the update is not finished after a few minutes, something may be wrong with the update. 


```bash
cloud_user@ip-10-0-1-101:~$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
candy-deployment   2/2     2            2           165m
cloud_user@ip-10-0-1-101:~$ kubectl set image deployment/candy-deployment candy-ws=linuxacademycontent/candy-service:3 --record
deployment.extensions/candy-deployment image updated
cloud_user@ip-10-0-1-101:~$ kubectl rollout status deployment/candy-deployment
Waiting for deployment "candy-deployment" rollout to finish: 1 out of 2 new replicas have been updated...
^Ccloud_user@ip-10-0-1-101:~$ kubectl get pods
NAME                                READY   STATUS             RESTARTS   AGE
candy-deployment-67d68bcc6f-9gwg2   0/1     ImagePullBackOff   0          100s
candy-deployment-6d655874b4-7525f   1/1     Running            0          169m
candy-deployment-6d655874b4-lbhcx   1/1     Running            0          169m
```

We also can check the image and restore doing the following:  

```bash
cloud_user@ip-10-0-1-101:~$ kubectl get deployment candy-deployment -o yaml --export > dc.yml
cloud_user@ip-10-0-1-101:~$ vi dc.yml
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"candy-deployment","namespace":"default"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"candy-ws"}},"template":{"metadata":{"labels":{"app":"candy-ws"}},"spec":{"containers":[{"image":"linuxacademycontent/candy-service:2","name":"candy-ws"}]}}}}
    kubernetes.io/change-cause: kubectl set image deployment/candy-deployment candy-ws=linuxacademycontent/candy-service:3
      --record=true
  creationTimestamp: null
  generation: 1
  labels:
    app: candy-ws
  name: candy-deployment
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/candy-deployment
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: candy-ws
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: candy-ws
    spec:
      containers:
      - image: linuxacademycontent/candy-service:3
        imagePullPolicy: IfNotPresent
        name: candy-ws
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}
```

#### Perform a rolling update of the container version  

In this scenario, the rolling update should fail. This is because the specified image, linuxacademycontent/candy-service:3, does not exist. Roll back to the previous version to get things working again.  

Get a list of previous revisions.  

```kubectl rollout history deployment/candy-deployment```  

```bash
cloud_user@ip-10-0-1-101:~$ kubectl rollout history
error: required resource not specified
cloud_user@ip-10-0-1-101:~$ kubectl get dc
error: the server doesn't have a resource type "dc"
cloud_user@ip-10-0-1-101:~$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
candy-deployment   2/2     1            2           173m
cloud_user@ip-10-0-1-101:~$ kubectl rollout history candy-deployment
error: the server doesn't have a resource type "candy-deployment"
cloud_user@ip-10-0-1-101:~$ kubectl rollout history deployments/candy-deployment
deployment.extensions/candy-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/candy-deployment candy-ws=linuxacademycontent/candy-service:3 --record=true

```

Undo the last revision.  

```kubectl rollout undo deployment/candy-deployment```  

Check the status of the rollout.  

```kubectl rollout status deployment/candy-deployment```  

This command should complete soon, saying the rollout was successful.  


```bash
cloud_user@ip-10-0-1-101:~$ kubectl rollout undo deployment/candy-deployment
deployment.extensions/candy-deployment rolled back
cloud_user@ip-10-0-1-101:~$ kubectl rollout status deployment/candy-deployment
deployment "candy-deployment" successfully rolled out
cloud_user@ip-10-0-1-101:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
candy-deployment-6d655874b4-7525f   1/1     Running   0          174m
candy-deployment-6d655874b4-lbhcx   1/1     Running   0          174m
```