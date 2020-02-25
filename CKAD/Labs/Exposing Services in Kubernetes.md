#  Exposing Services in Kubernetes      
      
## Additional Information and Resources       
    
    
Your company has just deployed two components of a web application to a Kubernetes cluster, using deployments with multiple replicas. They need a way to provide dynamic network access to these replicas so that there will be uninterrupted access to the components whenever replicas are created, removed, and replaced. One deployment is called ```auth-deployment```, an authentication provider that needs to be accessible from outside the cluster. The other is called ```data-deployment```, and it is a component designed to be accessed only by other pods within the cluster.    
    
The team wants you to create two services to expose these two components. Examine the two deployments, and create two services that meet the following criteria:      
      
auth-svc      
    
-  The service name is auth-svc.    
-  The service exposes the pod replicas managed by the deployment named auth-deployment.    
-  The service listens on port 8080 and its targetPort matches the port exposed by the pods.    
-  The service type is NodePort.    
    
data-svc    
    
-  The service name is data-svc.    
-  The service exposes the pod replicas managed by the deployment named data-deployment.    
-  The service listens on port 8080 and its targetPort matches the port exposed by the pods.    
-  The service type is ClusterIP.    
    
Note: All work should be done in the ```default``` namespace.    
    
    
### Execution      
    
#### Create the `auth-svc` service    
  
Examine the auth-deployment. Take note of the labels specified in the pod template, as well as the containerPort exposed by the containers.    
  
```kubectl get deployment auth-deployment -o yaml```  
  
Create a service descriptor file called auth-svc.yml.  
  
```yaml  
apiVersion: v1  
kind: Service  
metadata:  
  name: auth-svc  
spec:  
  type: NodePort  
  selector:  
    app: auth  
  ports:  
  - protocol: TCP  
    port: 8080  
    targetPort: 80  
```  
  
Create the service in the cluster.    
  
```kubectl apply -f auth-svc.yml```  
  
#### Create the `data-svc` service    
  
Examine the data-deployment. Take note of the labels specified in the pod template, as well as the containerPort exposed by the containers.  
  
```kubectl get deployment data-deployment -o yaml```  
  
Create a service descriptor file called data-svc.yml.  
  
```yaml  
apiVersion: v1  
kind: Service  
metadata:  
  name: data-svc  
spec:  
  type: ClusterIP  
  selector:  
    app: data  
  ports:  
  - protocol: TCP  
    port: 8080  
    targetPort: 80  
```  
Create the service in the cluster.    
  
```kubectl apply -f data-svc.yml```

  

```bash
cloud_user@ip-10-0-1-101:~$ kubectl get pods -n default
NAME                              READY   STATUS    RESTARTS   AGE
auth-deployment-b448d8b76-57xmp   1/1     Running   0          132m
auth-deployment-b448d8b76-88jvk   1/1     Running   0          132m
data-deployment-dcfff7fd6-572dd   1/1     Running   0          132m
data-deployment-dcfff7fd6-9nqbn   1/1     Running   0          132m
data-deployment-dcfff7fd6-l6l8z   1/1     Running   0          132m
cloud_user@ip-10-0-1-101:~$ kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
auth-svc     NodePort    10.109.11.62   <none>        8080:30802/TCP   25s
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          133m
cloud_user@ip-10-0-1-101:~$ kubectl apply -f data-svc.yml
The Service "data-svc" is invalid: spec.type: Unsupported value: "clusterIP": supported values: "ClusterIP", "ExternalName", "LoadBalancer", "NodePort"
cloud_user@ip-10-0-1-101:~$ vi data-svc.yml
cloud_user@ip-10-0-1-101:~$ kubectl apply -f data-svc.yml
service/data-svc created
cloud_user@ip-10-0-1-101:~$ kubectl get services
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
auth-svc     NodePort    10.109.11.62     <none>        8080:30802/TCP   75s
data-svc     ClusterIP   10.100.172.204   <none>        8080/TCP         5s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          133m
cloud_user@ip-10-0-1-101:~$ kubectl get services auth-svc -o yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"auth-svc","namespace":"default"},"spec":{"ports":[{"port":8080,"protocol":"TCP","targetPort":80}],"selector":{"app":"auth"},"type":"NodePort"}}
  creationTimestamp: "2020-02-25T13:18:31Z"
  name: auth-svc
  namespace: default
  resourceVersion: "11780"
  selfLink: /api/v1/namespaces/default/services/auth-svc
  uid: 514c784f-57d1-11ea-a880-0e9dfaf0dba9
spec:
  clusterIP: 10.109.11.62
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 30802
    port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: auth
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

To check:  

```bash
cloud_user@ip-10-0-1-101:~$ kubectl get endpoints auth-svc
NAME       ENDPOINTS                     AGE
auth-svc   10.244.2.5:80,10.244.2.6:80   7m4s
cloud_user@ip-10-0-1-101:~$ kubectl get endpoints data-svc
NAME       ENDPOINTS                                   AGE
data-svc   10.244.2.2:80,10.244.2.3:80,10.244.2.4:80   6m
```