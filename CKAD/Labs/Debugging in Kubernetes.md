# Debugging in Kubernetes
  
## Additional Information and Resources    
  
You recently got a new job at a company that has a robust Kubernetes infrastructure used by multiple teams. Congratulations! However, you were just told by the team that there is a problem with a service they use in the cluster and they want you to fix it. Unfortunately, no one is able to give you much information about the service. You don't even know what it is called or where it is located. All you know is there is likely a pod failing somewhere.    
    
Your team has asked you to take the lead in debugging the issue. They want you to locate the problem and collect some relevant debugging information that will help the team analyze the problem and correct it in the future. They also want you to go ahead and get the broken pod running again.    
    
### You will need to do the following:    
    
- Find the broken pod and save the pod name to the file /home/cloud_user/debug/broken-pod-name.txt.    
- In the same namespace as the broken pod, find out which pod is using the most CPU and output the name of that pod to the file /home/cloud_user/debug/high-cpu-pod-name.txt.    
- Get the broken pod's summary data in the JSON format and output it to the file /home/cloud_user/debug/broken-pod-summary.json.    
- Get the broken pod's container logs and output them to the file /home/cloud_user/debug/broken-pod-logs.log.    
- Fix the problem with the broken pod so that it enters the Running state.    
  
  
### Execution  
  
1. Since you don't know what namespace the broken pod is in, a quick way to find the broken pod is to list all pods from all namespaces:    
  
```kubectl get pods --all-namespaces```    
  
Check the ```STATUS``` field to find which pod is broken. Once you have located the broken pod, ```vi /home/cloud_user/debug/broken-pod-name.txt```, enter the name of the broken pod, and save the file.  


```bash

NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
candy-store   auth-ws                                 2/2     Running            1          73m
candy-store   candy-ws                                2/2     Running            1          73m
candy-store   cart-ws                                 0/1     CrashLoopBackOff   29         73m
default       inventory-svc                           1/1     Running            0          73m
default       shipping-svc                            1/1     Running            0          73m
internal      build-svc                               1/1     Running            0          73m
kube-system   coredns-54ff9cd656-j4zjz                1/1     Running            0          73m
kube-system   coredns-54ff9cd656-jktcn                1/1     Running            0          73m
kube-system   etcd-ip-10-0-1-101                      1/1     Running            0          73m
kube-system   kube-apiserver-ip-10-0-1-101            1/1     Running            0          73m
kube-system   kube-controller-manager-ip-10-0-1-101   1/1     Running            0          72m
kube-system   kube-flannel-ds-amd64-4rfd7             1/1     Running            0          73m
kube-system   kube-flannel-ds-amd64-6vng4             1/1     Running            0          73m
kube-system   kube-flannel-ds-amd64-8g4jl             1/1     Running            0          73m
kube-system   kube-proxy-7xp9n                        1/1     Running            0          73m
kube-system   kube-proxy-v85qh                        1/1     Running            0          73m
kube-system   kube-proxy-xpmq2                        1/1     Running            0          73m
kube-system   kube-scheduler-ip-10-0-1-101            1/1     Running            0          73m
kube-system   metrics-server-6447c7cf8c-zg69f         1/1     Running            0          73m

```

Or even better:  

```kubectl get po --all-namespaces|grep -v Running```

```bash
NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
candy-store   cart-ws                                 0/1     CrashLoopBackOff   31         80m
```

```bash
cloud_user@ip-10-0-1-101:~$ if [[ ! -f /home/cloud_user/debug/broken-pod-name.txt ]]; then mkdir -p /home/cloud_user/debug/broken-pod-name.txt; else rm -f /home/cloud_user/debug/broken-pod-name.txt; fi; kubectl get po --all-namespaces|grep -v Running > /home/cloud_user/debug/broken-pod-name.txt
cloud_user@ip-10-0-1-101:~$ vi /home/cloud_user/debug/broken-pod-name.txt
```

Edit with vim till:  

```bash
cloud_user@ip-10-0-1-101:~$ cat /home/cloud_user/debug/broken-pod-name.txt
cart-ws
```
  
2. Look at the namespace of the broken pod, and then use ```kubectl top pod``` to show resource usage for all pods in that namespace.  
  
```kubectl top pod -n <namespace>```  
  
Identify which pod in that namespace is using the most CPU, then ```vi /home/cloud_user/debug/high-cpu-pod-name.txt```, enter the name of the pod with the highest CPU usage, and then save the file.  
  
  
```bash
cloud_user@ip-10-0-1-101:~$ kubectl get pods --all-namespaces | grep -v Running
NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
candy-store   cart-ws                                 0/1     CrashLoopBackOff   35         87m
cloud_user@ip-10-0-1-101:~$ kubectl top pod -n candy-store
NAME       CPU(cores)   MEMORY(bytes)
auth-ws    200m         8Mi
candy-ws   100m         8Mi
cloud_user@ip-10-0-1-101:~$ echo "auth-ws" > /home/cloud_user/debug/high-cpu-pod-name.txt
cloud_user@ip-10-0-1-101:~$ cat /home/cloud_user/debug/high-cpu-pod-name.txt
auth-ws
```

3. You can get the JSON data and output it to the file like this:    
  
```kubectl get pod <pod name> -n <namespace> -o json > /home/cloud_user/debug/broken-pod-summary.json```  

```bash
cloud_user@ip-10-0-1-101:~$ kubectl get pods --all-namespaces | grep -v Running
NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
candy-store   cart-ws                                 0/1     CrashLoopBackOff   35         90m
cloud_user@ip-10-0-1-101:~$ kubectl get pods cart-ws
Error from server (NotFound): pods "cart-ws" not found
cloud_user@ip-10-0-1-101:~$ kubectl get pods cart-ws --all-namespaces
error: a resource cannot be retrieved by name across all namespaces
cloud_user@ip-10-0-1-101:~$ kubectl get pods cart-ws -n candy-store
NAME      READY   STATUS             RESTARTS   AGE
cart-ws   0/1     CrashLoopBackOff   35         90m
cloud_user@ip-10-0-1-101:~$ kubectl get pods cart-ws -n candy-store -o json > /home/cloud_user/debug/broken-pod-summary.json
cloud_user@ip-10-0-1-101:~$ cat /home/cloud_user/debug/broken-pod-summary.json
```

```json
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "annotations": {
            "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"kind\":\"Pod\",\"metadata\":{\"annotations\":{},\"name\":\"cart-ws\",\"namespace\":\"candy-store\"},\"spec\":{\"containers\":[{\"image\":\"linuxacademycontent/candy-service:2\",\"livenessProbe\":{\"httpGet\":{\"path\":\"/ealthz\",\"port\":8081},\"initialDelaySeconds\":5,\"periodSeconds\":5},\"name\":\"cart-ws\"}]}}\n"
        },
        "creationTimestamp": "2020-02-25T09:37:00Z",
        "name": "cart-ws",
        "namespace": "candy-store",
        "resourceVersion": "8137",
        "selfLink": "/api/v1/namespaces/candy-store/pods/cart-ws",
        "uid": "5f8e59fb-57b2-11ea-96ae-021f817bb7d7"
    },
    "spec": {
        "containers": [
            {
                "image": "linuxacademycontent/candy-service:2",
                "imagePullPolicy": "IfNotPresent",
                "livenessProbe": {
                    "failureThreshold": 3,
                    "httpGet": {
                        "path": "/ealthz",
                        "port": 8081,
                        "scheme": "HTTP"
                    },
                    "initialDelaySeconds": 5,
                    "periodSeconds": 5,
                    "successThreshold": 1,
                    "timeoutSeconds": 1
                },
                "name": "cart-ws",
                "resources": {},
                "terminationMessagePath": "/dev/termination-log",
                "terminationMessagePolicy": "File",
                "volumeMounts": [
                    {
                        "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
                        "name": "default-token-6wr9j",
                        "readOnly": true
                    }
                ]
            }
        ],
        "dnsPolicy": "ClusterFirst",
        "enableServiceLinks": true,
        "nodeName": "ip-10-0-1-103",
        "priority": 0,
        "restartPolicy": "Always",
        "schedulerName": "default-scheduler",
        "securityContext": {},
        "serviceAccount": "default",
        "serviceAccountName": "default",
        "terminationGracePeriodSeconds": 30,
        "tolerations": [
            {
                "effect": "NoExecute",
                "key": "node.kubernetes.io/not-ready",
                "operator": "Exists",
                "tolerationSeconds": 300
            },
            {
                "effect": "NoExecute",
                "key": "node.kubernetes.io/unreachable",
                "operator": "Exists",
                "tolerationSeconds": 300
            }
        ],
        "volumes": [
            {
                "name": "default-token-6wr9j",
                "secret": {
                    "defaultMode": 420,
                    "secretName": "default-token-6wr9j"
                }
            }
        ]
    },
    "status": {
        "conditions": [
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2020-02-25T09:37:25Z",
                "status": "True",
                "type": "Initialized"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2020-02-25T11:03:58Z",
                "message": "containers with unready status: [cart-ws]",
                "reason": "ContainersNotReady",
                "status": "False",
                "type": "Ready"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2020-02-25T11:03:58Z",
                "message": "containers with unready status: [cart-ws]",
                "reason": "ContainersNotReady",
                "status": "False",
                "type": "ContainersReady"
            },
            {
                "lastProbeTime": null,
                "lastTransitionTime": "2020-02-25T09:37:24Z",
                "status": "True",
                "type": "PodScheduled"
            }
        ],
        "containerStatuses": [
            {
                "containerID": "docker://fcb494465da8cef2d6d9ea2f4cee861123295dbb8e405ec22db77e85bc28102d",
                "image": "linuxacademycontent/candy-service:2",
                "imageID": "docker-pullable://linuxacademycontent/candy-service@sha256:614fb2c1f251a53968564f0ab22576e6af4236df8c65e19508062541a54bf4fa",
                "lastState": {
                    "terminated": {
                        "containerID": "docker://fcb494465da8cef2d6d9ea2f4cee861123295dbb8e405ec22db77e85bc28102d",
                        "exitCode": 0,
                        "finishedAt": "2020-02-25T11:03:57Z",
                        "reason": "Completed",
                        "startedAt": "2020-02-25T11:03:42Z"
                    }
                },
                "name": "cart-ws",
                "ready": false,
                "restartCount": 35,
                "state": {
                    "waiting": {
                        "message": "Back-off 5m0s restarting failed container=cart-ws pod=cart-ws_candy-store(5f8e59fb-57b2-11ea-96ae-021f817bb7d7)",
                        "reason": "CrashLoopBackOff"
                    }
                }
            }
        ],
        "hostIP": "10.0.1.103",
        "phase": "Running",
        "podIP": "10.244.1.6",
        "qosClass": "BestEffort",
        "startTime": "2020-02-25T09:37:25Z"
    }
}
```
  
4. You can get the logs and output them to the file like this:  
  
```kubectl logs <pod name> -n <namespace> > /home/cloud_user/debug/broken-pod-logs.log``` 

```bash

cloud_user@ip-10-0-1-101:~$ kubectl get pods --all-namespaces | grep -v Running
NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
candy-store   cart-ws                                 0/1     CrashLoopBackOff   37         93m
cloud_user@ip-10-0-1-101:~$ kubectl logs cart-ws -n candy-store > /home/cloud_user/debug/broken-pod-logs.log
cloud_user@ip-10-0-1-101:~$ cat /home/cloud_user/debug/broken-pod-logs.log
2020/02/25 11:09:27 [error] 6#6: *1 open() "/etc/nginx/html/ealthz" failed (2: No such file or directory), client: 10.244.1.1, server: , request: "GET /ealthz HTTP/1.1", host: "10.244.1.6:8081"
10.244.1.1 - - [25/Feb/2020:11:09:27 +0000] "GET /ealthz HTTP/1.1" 404 153 "-" "kube-probe/1.13"
2020/02/25 11:09:32 [error] 6#6: *2 open() "/etc/nginx/html/ealthz" failed (2: No such file or directory), client: 10.244.1.1, server: , request: "GET /ealthz HTTP/1.1", host: "10.244.1.6:8081"
10.244.1.1 - - [25/Feb/2020:11:09:32 +0000] "GET /ealthz HTTP/1.1" 404 153 "-" "kube-probe/1.13"
2020/02/25 11:09:37 [error] 6#6: *3 open() "/etc/nginx/html/ealthz" failed (2: No such file or directory), client: 10.244.1.1, server: , request: "GET /ealthz HTTP/1.1", host: "10.244.1.6:8081"
10.244.1.1 - - [25/Feb/2020:11:09:37 +0000] "GET /ealthz HTTP/1.1" 404 153 "-" "kube-probe/1.13"
```
  
5. Describe the broken pod to help identify what is wrong:  
  
```kubectl describe pod <pod name> -n <namespace>```  
  
Check the ```Events``` to see if you can spot what is wrong.  

```bash
cloud_user@ip-10-0-1-101:~$ kubectl get pods --all-namespaces | grep -v Running
NAMESPACE     NAME                                    READY   STATUS             RESTARTS   AGE
candy-store   cart-ws                                 0/1     CrashLoopBackOff   37         96m
cloud_user@ip-10-0-1-101:~$ kubectl describe pods cart-ws -n candy-store
Name:               cart-ws
Namespace:          candy-store
Priority:           0
PriorityClassName:  <none>
Node:               ip-10-0-1-103/10.0.1.103
Start Time:         Tue, 25 Feb 2020 09:37:25 +0000
Labels:             <none>
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"cart-ws","namespace":"candy-store"},"spec":{"containers":[{"image":"l...
Status:             Running
IP:                 10.244.1.6
Containers:
  cart-ws:
    Container ID:   docker://4fef21658e9ce99853bfaafb30377aed7a6f8f1b8acf3484b771ee5f8faeafc3
    Image:          linuxacademycontent/candy-service:2
    Image ID:       docker-pullable://linuxacademycontent/candy-service@sha256:614fb2c1f251a53968564f0ab22576e6af4236df8c65e19508062541a54bf4fa
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 25 Feb 2020 11:09:22 +0000
      Finished:     Tue, 25 Feb 2020 11:09:37 +0000
    Ready:          False
    Restart Count:  37
    Liveness:       http-get http://:8081/ealthz delay=5s timeout=1s period=5s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-6wr9j (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  default-token-6wr9j:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-6wr9j
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                  From                    Message
  ----     ------     ----                 ----                    -------
  Warning  Unhealthy  21m (x96 over 96m)   kubelet, ip-10-0-1-103  Liveness probe failed: HTTP probe failed with statuscode: 404
  Warning  BackOff    72s (x413 over 95m)  kubelet, ip-10-0-1-103  Back-off restarting failed container
```
  
You may notice the pod's liveness probe is failing. If you look closely, you might also notice the path for the liveness probe looks like it may be incorrect.  
  
In order to edit and fix the liveness probe, you will need to delete and recreate the pod. You should save the pod descriptor before deleting it, or you will have no way to recover it!  
  
```kubectl get pod <pod name> -n <namespace> -o yaml --export > broken-pod.yml``` 

```bash
cloud_user@ip-10-0-1-101:~$ kubectl describe pod cart-ws -n candy-store -o yaml --export > broken-pod.yml
Error: unknown shorthand flag: 'o' in -o


Examples:
  # Describe a node
  kubectl describe nodes kubernetes-node-emt8.c.myproject.internal

  # Describe a pod
  kubectl describe pods/nginx

  # Describe a pod identified by type and name in "pod.json"
  kubectl describe -f pod.json

  # Describe all pods
  kubectl describe pods

  # Describe pods by label name=myLabel
  kubectl describe po -l name=myLabel

  # Describe all pods managed by the 'frontend' replication controller (rc-created pods
  # get the name of the rc as a prefix in the pod the name).
  kubectl describe pods frontend

Options:
      --all-namespaces=false: If present, list the requested object(s) across all namespaces. Namespace in current context is ignored even if specified with --namespace.
  -f, --filename=[]: Filename, directory, or URL to files containing the resource to describe
      --include-uninitialized=false: If true, the kubectl command applies to uninitialized objects. If explicitly set to false, this flag overrides other flags that make the kubectl commands apply to uninitialized objects, e.g., "--all". Objects with empty metadata.initializers are regarded as initialized.
  -R, --recursive=false: Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests organized within the same directory.
  -l, --selector='': Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2)
      --show-events=true: If true, display events related to the described object.

Usage:
  kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | -l label] | TYPE/NAME) [options]

Use "kubectl options" for a list of global command-line options (applies to all commands).

unknown shorthand flag: 'o' in -o
```
```bash
cloud_user@ip-10-0-1-101:~$ kubectl get pod cart-ws -n candy-store -o yml --export > broken-pod.yml
error: unable to match a printer suitable for the output format "yml", allowed formats are: custom-columns,custom-columns-file,go-template,go-template-file,json,jsonpath,jsonpath-file,name,template,templatefile,wide,yaml
cloud_user@ip-10-0-1-101:~$ kubectl get pod cart-ws -n candy-store -o yaml --export > broken-pod.yml
cloud_user@ip-10-0-1-101:~$ vi broken-pod.yml
        path: /healthz 
        port: 8081
        scheme: HTTP
      initialDelaySeconds: 5
      periodSeconds: 5
      successThreshold: 1
      timeoutSeconds: 1
    name: cart-ws
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-6wr9j
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: ip-10-0-1-103
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
  - name: default-token-6wr9j
    secret:
      defaultMode: 420
      secretName: default-token-6wr9j
status:
  phase: Pending
  qosClass: BestEffort
```

If We dont remove all pod:  

```bash
cloud_user@ip-10-0-1-101:~$ kubectl apply -f broken-pod.yml -n candy-store
The Pod "cart-ws" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
{"Volumes":[{"Name":"default-token-6wr9j","HostPath":null,"EmptyDir":null,"GCEPersistentDisk":null,"AWSElasticBlockStore":null,"GitRepo":null,"Secret":{"SecretName":"default-token-6wr9j","Items":null,"DefaultMode":420,"Optional":null},"NFS":null,"ISCSI":null,"Glusterfs":null,"PersistentVolumeClaim":null,"RBD":null,"Quobyte":null,"FlexVolume":null,"Cinder":null,"CephFS":null,"Flocker":null,"DownwardAPI":null,"FC":null,"AzureFile":null,"ConfigMap":null,"VsphereVolume":null,"AzureDisk":null,"PhotonPersistentDisk":null,"Projected":null,"PortworxVolume":null,"ScaleIO":null,"StorageOS":null}],"InitContainers":null,"Containers":[{"Name":"cart-ws","Image":"linuxacademycontent/candy-service:2","Command":null,"Args":null,"WorkingDir":"","Ports":null,"EnvFrom":null,"Env":null,"Resources":{"Limits":null,"Requests":null},"VolumeMounts":[{"Name":"default-token-6wr9j","ReadOnly":true,"MountPath":"/var/run/secrets/kubernetes.io/serviceaccount","SubPath":"","MountPropagation":null}],"VolumeDevices":null,"LivenessProbe":{"Exec":null,"HTTPGet":{"Path":"/

A: healthz","Port":8081,"Host":"","Scheme":"HTTP","HTTPHeaders":null},"TCPSocket":null,"InitialDelaySeconds":5,"TimeoutSeconds":1,"PeriodSeconds":5,"SuccessThreshold":1,"FailureThreshold":3},"ReadinessProbe":null,"Lifecycle":null,"TerminationMessagePath":"/dev/termination-log","TerminationMessagePolicy":"File","ImagePullPolicy":"IfNotPresent","SecurityContext":null,"Stdin":false,"StdinOnce":false,"TTY":false}],"RestartPolicy":"Always","TerminationGracePeriodSeconds":30,"ActiveDeadlineSeconds":null,"DNSPolicy":"ClusterFirst","NodeSelector":null,"ServiceAccountName":"default","AutomountServiceAccountToken":null,"NodeName":"ip-10-0-1-103","SecurityContext":{"HostNetwork":false,"HostPID":false,"HostIPC":false,"ShareProcessNamespace":null,"SELinuxOptions":null,"RunAsUser":null,"RunAsGroup":null,"RunAsNonRoot":null,"SupplementalGroups":null,"FSGroup":null,"Sysctls":null},"ImagePullSecrets":null,"Hostname":"","Subdomain":"","Affinity":null,"SchedulerName":"default-scheduler","Tolerations":[{"Key":"node.kubernetes.io/not-ready","Operator":"Exists","Value":"","Effect":"NoExecute","TolerationSeconds":300},{"Key":"node.kubernetes.io/unreachable","Operator":"Exists","Value":"","Effect":"NoExecute","TolerationSeconds":300}],"HostAliases":null,"PriorityClassName":"","Priority":0,"DNSConfig":null,"ReadinessGates":null,"RuntimeClassName":null,"EnableServiceLinks":true}

B: ealthz","Port":8081,"Host":"","Scheme":"HTTP","HTTPHeaders":null},"TCPSocket":null,"InitialDelaySeconds":5,"TimeoutSeconds":1,"PeriodSeconds":5,"SuccessThreshold":1,"FailureThreshold":3},"ReadinessProbe":null,"Lifecycle":null,"TerminationMessagePath":"/dev/termination-log","TerminationMessagePolicy":"File","ImagePullPolicy":"IfNotPresent","SecurityContext":null,"Stdin":false,"StdinOnce":false,"TTY":false}],"RestartPolicy":"Always","TerminationGracePeriodSeconds":30,"ActiveDeadlineSeconds":null,"DNSPolicy":"ClusterFirst","NodeSelector":null,"ServiceAccountName":"default","AutomountServiceAccountToken":null,"NodeName":"ip-10-0-1-103","SecurityContext":{"HostNetwork":false,"HostPID":false,"HostIPC":false,"ShareProcessNamespace":null,"SELinuxOptions":null,"RunAsUser":null,"RunAsGroup":null,"RunAsNonRoot":null,"SupplementalGroups":null,"FSGroup":null,"Sysctls":null},"ImagePullSecrets":null,"Hostname":"","Subdomain":"","Affinity":null,"SchedulerName":"default-scheduler","Tolerations":[{"Key":"node.kubernetes.io/not-ready","Operator":"Exists","Value":"","Effect":"NoExecute","TolerationSeconds":300},{"Key":"node.kubernetes.io/unreachable","Operator":"Exists","Value":"","Effect":"NoExecute","TolerationSeconds":300}],"HostAliases":null,"PriorityClassName":"","Priority":0,"DNSConfig":null,"ReadinessGates":null,"RuntimeClassName":null,"EnableServiceLinks":true}
```

So We have to remove it:  

```bash
cloud_user@ip-10-0-1-101:~$ kubectl delete pod cart-ws -n candy-store
pod "cart-ws" deleted

cloud_user@ip-10-0-1-101:~$
cloud_user@ip-10-0-1-101:~$ kubectl apply -f broken-pod.yml -n candy-store
pod/cart-ws created
```

Delete the broken pod:  
  
```kubectl delete pod <pod name> -n <namespace>```  
  
Now, edit the descriptor file, and fix the ```path``` attribute for the liveness probe: ```vi broken-pod.yml```.  
  
Recreate the broken pod with the fixed probe:  
  
```kubectl apply -f broken-pod.yml -n <namespace>```  
  
Check to make sure the pod is now running properly:  
  
```kubectl get pod <pod name> -n <namespace>```  
  
```bash
cloud_user@ip-10-0-1-101:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
candy-store   auth-ws                                 2/2     Running   1          106m
candy-store   candy-ws                                2/2     Running   1          106m
candy-store   cart-ws                                 1/1     Running   0          91s
default       inventory-svc                           1/1     Running   0          106m
default       shipping-svc                            1/1     Running   0          106m
internal      build-svc                               1/1     Running   0          106m
kube-system   coredns-54ff9cd656-j4zjz                1/1     Running   0          106m
kube-system   coredns-54ff9cd656-jktcn                1/1     Running   0          106m
kube-system   etcd-ip-10-0-1-101                      1/1     Running   0          106m
kube-system   kube-apiserver-ip-10-0-1-101            1/1     Running   0          105m
kube-system   kube-controller-manager-ip-10-0-1-101   1/1     Running   0          105m
kube-system   kube-flannel-ds-amd64-4rfd7             1/1     Running   0          106m
kube-system   kube-flannel-ds-amd64-6vng4             1/1     Running   0          106m
kube-system   kube-flannel-ds-amd64-8g4jl             1/1     Running   0          106m
kube-system   kube-proxy-7xp9n                        1/1     Running   0          106m
kube-system   kube-proxy-v85qh                        1/1     Running   0          106m
kube-system   kube-proxy-xpmq2                        1/1     Running   0          106m
kube-system   kube-scheduler-ip-10-0-1-101            1/1     Running   0          106m
kube-system   metrics-server-6447c7cf8c-zg69f         1/1     Running   0          106m
```  
  
  
