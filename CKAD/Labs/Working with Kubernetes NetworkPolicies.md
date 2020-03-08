# Working with Kubernetes NetworkPolicies     
    
## Additional Information and Resources    
  
Your company has a set of services, one called ```inventory-svc``` and another called ```customer-data-svc```. In the interest of security, both of these services and their corresponding pods have NetworkPolicies designed to restrict network communication to and from them. A new pod has just been deployed to the cluster called ```web-gateway```, and this pod need to be able to access both inventory-svc and customer-data-svc.  
  
Unfortunately, whoever designed the services and their corresponding NetworkPolicies was a little lax in creating documentation. In top of that, they are not currently available to help you understand how to provide access to the services for the new pod.  
  
Examine the existing NetworkPolicies and determine how to alter the web-gateway pod so that it can access the pods associated with both services.  
  
You will not need to add, delete, or edit any NetworkPolicies in order to do this. Simply use the existing ones and modify the web-gateway pod to provide access. All work can be done in the ```default``` namespace.  
    
  
  
### Execution    
  
#### Provide the `web-gateway` Pod with Network Access to the Pods Associated with the `inventory-svc` Service    
  
First, get a list of existing NetworkPolicies:  
  
```kubectl get networkpolicy```  
  
Examine inventory-policy more closely:  
  
```kubectl describe networkpolicy inventory-policy```  
  
```bash  
Name:         inventory-policy  
Namespace:    default  
Created on:   2020-03-08 09:06:37 +0000 UTC  
Labels:       <none>  
Annotations:  kubectl.kubernetes.io/last-applied-configuration:  
                {"apiVersion":"networking.k8s.io/v1","kind":"NetworkPolicy","metadata":{"annotations":{},"name":"inventory-policy","namespace":"default"},...  
Spec:  
  PodSelector:     app=inventory  
  Allowing ingress traffic:  
    To Port: 80/TCP  
    From:  
      PodSelector: inventory-access=true  
  Allowing egress traffic:  
    To Port: 80/TCP  
    To:  
      PodSelector: inventory-access=true  
  Policy Types: Ingress, Egress  
```  
  
Note that the policy selects pods with the label app: inventory, and provides incoming and outgoing network access to all pods with the label inventory-access: true.  
  
Modify the web-gateway pod with kubectl edit pod web-gateway.  
  
Add the inventory-access: "true" label to the pod under metdadata.labels.  
  
```bash  
...  
metdadata:  
  labels:  
    inventory-access: "true"  
...  
```  
  
Test access to the inventory-svc like so:  
  
```kubectl exec web-gateway -- curl -m 3 inventory-svc```  
  
#### Provide the `web-gateway` Pod with Network Access to the Pods Associated with the `customer-data-svc` Service  
  
Examine customer-data-policy more closely:  
  
```kubectl describe networkpolicy customer-data-policy```  
  
Note that the policy selects pods with the label app: customer-data, and provides incoming and outgoing network access to all pods with the label customer-data-access: true.  
  
Modify the web-gateway pod with kubectl edit pod web-gateway.  
  
Add the customer-data-access: "true" label to the pod under metdadata.labels:  
  
```bash  
...  
metdadata:  
  labels:  
    inventory-access: "true"  
    customer-data-access: "true"  
...  
```  
  
#### Test access to the customer-data-svc like so:  
  
  
```kubectl exec web-gateway -- curl -m 3 customer-data-svc```  
  
Personal notes:  
  
```kubectl get pod/web-gateway -o yaml --export > web-gateway.yml```  
  
  
```vi web-gateway.yml```  
  
```yaml  
kind: Pod  
metadata:  
  annotations:  
    cni.projectcalico.org/podIP: 10.244.1.5/32  
    kubectl.kubernetes.io/last-applied-configuration: |  
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"web-gateway"},"name":"web-gateway","namespace":"default"},"spec":{"containers":[{"command":["sh","-c","while true; do sleep 3600; done"],"image":"radial/busyboxplus:curl","name":"busybox"}]}}  
  creationTimestamp: null  
  labels:  
    app: web-gateway  
    inventory-access: "true"  
    customer-data-access: "true"  
  name: web-gateway  
  selfLink: /api/v1/namespaces/default/pods/web-gateway  
spec:  
  containers:  
  - command:  
    - sh  
    - -c  
    - while true; do sleep 3600; done  
    image: radial/busyboxplus:curl  
    imagePullPolicy: IfNotPresent  
    name: busybox  
    resources: {}  
    terminationMessagePath: /dev/termination-log  
    terminationMessagePolicy: File  
    volumeMounts:  
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount  
      name: default-token-rgmj8  
      readOnly: true  
  dnsPolicy: ClusterFirst  
  enableServiceLinks: true  
  nodeName: ip-10-0-1-102  
  priority: 0  
  restartPolicy: Always  
  schedulerName: default-scheduler  
  securityContext: {}  
  serviceAccount: default  
  serviceAccountName: default  
  terminationGracePeriodSeconds: 30  
  tolerations:  
  - effect: NoExecute  
    key: node.kubernetes.io/not-ready  
    operator: Exists  
    tolerationSeconds: 300  
  - effect: NoExecute  
    key: node.kubernetes.io/unreachable  
    operator: Exists  
    tolerationSeconds: 300  
  volumes:  
  - name: default-token-rgmj8  
    secret:  
      defaultMode: 420  
      secretName: default-token-rgmj8  
status:  
  phase: Pending  
  qosClass: BestEffort  
```  
  
```bash  
kubectl apply -f web-gateway.yml  
pod/web-gateway configured  
```  
  
  
Before changes:  
  
```bash  
cloud_user@ip-10-0-1-101:~$ kubectl exec web-gateway -- curl -m 3 inventory-svc  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
                                 Dload  Upload   Total   Spent    Left  Speed  
  0     0    0     0    0     0      0      0 --:--:--  0:00:03 --:--:--     0  
curl: (28) Connection timed out after 3001 milliseconds  
command terminated with exit code 28  
cloud_user@ip-10-0-1-101:~$ kubectl exec web-gateway -- curl -m 3 customer-data-svc  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
                                 Dload  Upload   Total   Spent    Left  Speed  
  0     0    0     0    0     0      0      0 --:--:--  0:00:03 --:--:--     0  
curl: (28) Connection timed out after 3001 milliseconds  
command terminated with exit code 28  
```  
    
After changes:  
```bash  
cloud_user@ip-10-0-1-101:~$ kubectl exec web-gateway -- curl -m 3 inventory-svc  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
                                 Dload  Upload   Total   Spent    Left  Speed  
100   612  100   612    0     0  76480      0 --:--:-- --:--:-- --:--:--  597k  
<!DOCTYPE html>  
<html>  
<head>  
<title>Welcome to nginx!</title>  
<style>  
    body {  
        width: 35em;  
        margin: 0 auto;  
        font-family: Tahoma, Verdana, Arial, sans-serif;  
    }  
</style>  
</head>  
<body>  
<h1>Welcome to nginx!</h1>  
<p>If you see this page, the nginx web server is successfully installed and  
working. Further configuration is required.</p>  
  
<p>For online documentation and support please refer to  
<a href="http://nginx.org/">nginx.org</a>.<br/>  
Commercial support is available at  
<a href="http://nginx.com/">nginx.com</a>.</p>  
  
<p><em>Thank you for using nginx.</em></p>  
</body>  
</html>  
cloud_user@ip-10-0-1-101:~$ kubectl exec web-gateway -- curl -m 3 customer-data-svc  
<!DOCTYPE html>  
<html>  
<head>  
<title>Welcome to nginx!</title>  
<style>  
    body {  
        width: 35em;  
        margin: 0 auto;  
        font-family: Tahoma, Verdana, Arial, sans-serif;  
    }  
</style>  
</head>  
<body>  
<h1>Welcome to nginx!</h1>  
<p>If you see this page, the nginx web server is successfully installed and  
working. Further configuration is required.</p>  
  
<p>For online documentation and support please refer to  
<a href="http://nginx.org/">nginx.org</a>.<br/>  
Commercial support is available at  
<a href="http://nginx.com/">nginx.com</a>.</p>  
  
<p><em>Thank you for using nginx.</em></p>  
</body>  
</html>  
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current  
                                 Dload  Upload   Total   Spent    Left  Speed  
100   612  100   612    0     0  93908      0 --:--:-- --:--:-- --:--:--  597k  
cloud_user@ip-10-0-1-101:~$  
```  
  
  
### Network policies  
  
```yaml  
  
apiVersion: extensions/v1beta1  
kind: NetworkPolicy  
metadata:  
  annotations:  
    kubectl.kubernetes.io/last-applied-configuration: |  
      {"apiVersion":"networking.k8s.io/v1","kind":"NetworkPolicy","metadata":{"annotations":{},"name":"customer-data-policy","namespace":"default"},"spec":{"egress":[{"ports":[{"port":80,"protocol":"TCP"}],"to":[{"podSelector":{"matchLabels":{"customer-data-access":"true"}}}]}],"ingress":[{"from":[{"podSelector":{"matchLabels":{"customer-data-access":"true"}}}],"ports":[{"port":80,"protocol":"TCP"}]}],"podSelector":{"matchLabels":{"app":"customer-data"}},"policyTypes":["Ingress","Egress"]}}  
  creationTimestamp: null  
  generation: 1  
  name: customer-data-policy  
  selfLink: /apis/extensions/v1beta1/namespaces/default/networkpolicies/customer-data-policy  
spec:  
  egress:  
  - ports:  
    - port: 80  
      protocol: TCP  
    to:  
    - podSelector:  
        matchLabels:  
          customer-data-access: "true"  
  ingress:  
  - from:  
    - podSelector:  
        matchLabels:  
          customer-data-access: "true"  
    ports:  
    - port: 80  
      protocol: TCP  
  podSelector:  
    matchLabels:  
      app: customer-data  
  policyTypes:  
  - Ingress  
  - Egress  
```  
```yaml  
apiVersion: extensions/v1beta1  
kind: NetworkPolicy  
metadata:  
  annotations:  
    kubectl.kubernetes.io/last-applied-configuration: |  
      {"apiVersion":"networking.k8s.io/v1","kind":"NetworkPolicy","metadata":{"annotations":{},"name":"inventory-policy","namespace":"default"},"spec":{"egress":[{"ports":[{"port":80,"protocol":"TCP"}],"to":[{"podSelector":{"matchLabels":{"inventory-access":"true"}}}]}],"ingress":[{"from":[{"podSelector":{"matchLabels":{"inventory-access":"true"}}}],"ports":[{"port":80,"protocol":"TCP"}]}],"podSelector":{"matchLabels":{"app":"inventory"}},"policyTypes":["Ingress","Egress"]}}  
  creationTimestamp: null  
  generation: 1  
  name: inventory-policy  
  selfLink: /apis/extensions/v1beta1/namespaces/default/networkpolicies/inventory-policy  
spec:  
  egress:  
  - ports:  
    - port: 80  
      protocol: TCP  
    to:  
    - podSelector:  
        matchLabels:  
          inventory-access: "true"  
  ingress:  
  - from:  
    - podSelector:  
        matchLabels:  
          inventory-access: "true"  
    ports:  
    - port: 80  
      protocol: TCP  
  podSelector:  
    matchLabels:  
      app: inventory  
  policyTypes:  
  - Ingress  
  - Egress  
  
```