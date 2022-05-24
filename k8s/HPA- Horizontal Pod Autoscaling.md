
 # HPA- Horizontal Pod Autoscaling
 
## Installing the Kubernetes Metrics Server


1. Deploy the Metrics Server with the following command:
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```	

2. Verify that the metrics-server deployment is running the desired number of pods with the following command.

```
kubectl get deployment metrics-server -n kube-system
```
	

## Run and expose php-apache server 

1. Create application

```
kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
```
	

2. Create the HorizontalPodAutoscaler

```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```
	

3. You can check the current status of the newly-made HorizontalPodAutoscaler, by running

```
kubectl get hpa
```
	

## Increase the load

1. Now run a infiniti loop 

```
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
	
2. Now run the following command in another terminal

```
kubectl get hpa php-apache --watch
```
	
3. Now you will see that the pod replica will increase as the stress on the CPU increases 

4. Now delete the load generator and observe that the pods will get deleted
