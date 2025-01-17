# example horizontal pod autoscaler
To show that the pod autoscaling and cluster autoscaling works we will deploy an example application.  
The example is based on the example of [The DevOpy Guy](https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/kubernetes/autoscaling#example-application). 

## example application
Copy both example deployment files ([application](https://github.com/simonostendorf/k3s-hetzner/blob/main/deployments/hpa/example-deployment-application.yml) and [traffic](https://github.com/simonostendorf/k3s-hetzner/blob/main/deployments/hpa/example-deployment-traffic.yml)) to your local machine and apply them to the cluster with the following commands:
```bash
kubectl apply -f deployments/hpa/example-deployment-application.yml
kubectl apply -f deployments/hpa/example-deployment-traffic.yml
```

Execute the following commands to execute the traffic generator inside the traffic-container:
```bash
kubectl exec -it traffic-generator -- sh
apk add --no-cache wrk
wrk -c 5 -t 5 -d 99999 -H "Connection: Close" http://application-cpu
```
This command will install wrk,a modern HTTP benchmarking tool capable of generating load. 
The command will generate 5 concurrent connections with 5 threads for 99999 seconds. The load will be generated against the application-cpu service.

With the following command you can the the top resource consuming pods:
```bash
kubectl top pods
```

## scale pods
To apply the horizontal autoscaling run the following command:
```bash
kubectl autoscale deploy/application-cpu --cpu-percent=95 --min=1 --max=10
```
This command will scale the application-cpu deployment to a maximum of 10 pods (and minimum of 1) if the cpu usage is higher than 95%.

You can get information about the autoscaler with the following command:
```bash
kubectl get hpa -owide
```

See also [scale up and down policies](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#configurable-scaling-behavior). 

## delete deployment
To delete the hpa and the example deployment run the following command:
```bash
kubectl delete hpa application-cpu
kubectl delete -f deployments/hpa/example-deployment-application.yml
kubectl delete -f deployments/hpa/example-deployment-traffic.yml
```