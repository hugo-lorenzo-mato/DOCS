# Working with Kubernetes NetworkPolicies       
      
## Additional Information and Resources      
    
Your company has a simple data cleanup process that is run periodically for maintenance purposes. They would like to stop doing this manually in order to save time, so you have been asked to implement a cron job in the Kubernetes cluster to run this process. Create a cron job called ```cleanup-cronjob``` using the ```linuxacademycontent/data-cleanup:1``` image. Have the job run every minute with the following cron expression: ```*/1 * * * *```.    
    
    
### Execution      
    
#### Create the cron job in the cluster     
  
Create a descriptor for the cron job with `vi ~/cleanup-cronjob.yml`.  
  
```yaml  
apiVersion: batch/v1beta1  
kind: CronJob  
metadata:  
  name: cleanup-cronjob  
spec:  
  schedule: "*/1 * * * *"  
  jobTemplate:  
    spec:  
      template:  
        spec:  
          containers:  
          - name: data-cleanup  
            image: linuxacademycontent/data-cleanup:1  
          restartPolicy: OnFailure  
```  
Create the cron job in the cluster.  
  
```kubectl apply -f ~/cleanup-cronjob.yml```  
    
```bash    
kubectl get cronjobs --all-namespaces  
NAMESPACE   NAME              SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE  
default     cleanup-cronjob   */1 * * * *   False     0        <none>          7s  
```   
    
#### Allow the cron job to run successfully  
  
Give the cron job a minute or so to run once, and then check the status.  
  
```kubectl get cronjob cleanup-cronjob```  
  
You should see a timestamp under LAST-SCHEDULE, indicating the job was executed.  
    
```bash    
kubectl get cronjobs --all-namespaces -w  
NAMESPACE   NAME              SCHEDULE          SUSPEND   ACTIVE   LAST SCHEDULE   AGE  
default   cleanup-cronjob   */1 * * * *   False   0        47s         63s  
default   cleanup-cronjob   */1 * * * *   False   1         5s         81s  
default   cleanup-cronjob   */1 * * * *   False   0        15s         91s  
default   cleanup-cronjob   */1 * * * *   False   1         5s         2m21s  
default   cleanup-cronjob   */1 * * * *   False   0         15s        2m31s  
```    
You should see a timestamp under LAST-SCHEDULE, indicating the job was executed.  
