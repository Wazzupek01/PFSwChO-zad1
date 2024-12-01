# **PFSwChO 2024 - Task 1**

### **Namespace Creation**
- **Objective**: Create a namespace `zad1` to encapsulate all related Kubernetes objects.
- **File**: [`namespace.yml`](namespace.yml)

### **1. Resource Quota**
- **Objective**: Define resource limits for the namespace `zad1`:
  - Maximum Pods: 10
  - CPU: 2 (2000m)
  - Memory: 1.5 Gi
- **File**: [`resource-quota.yml`](resource-quota.yml)

### **2. Pod `worker`**
- **Objective**: Create a Pod named `worker` based on the `nginx` image with specific resource limits and requests:
  - Limits: 200m CPU, 200Mi RAM
  - Requests: 100m CPU, 100Mi RAM
- **File**: [`worker-pod.yml`](worker-pod.yml)

### **3. Deployment and Service**
- **Objective**: Deploy the `php-apache` application with resource constraints:
  - Deployment Limits: 250m CPU, 250Mi RAM
  - Deployment Requests: 150m CPU, 150Mi RAM
- **Files**:
  - [`php-apache-deployment.yml`](php-apache-deployment.yml)

### **4. Horizontal Pod Autoscaler**
- **Objective**: Autoscale the `php-apache` Deployment based on CPU utilization:
  - Minimum Replicas: 1
  - Maximum Replicas: 6 - quota is set to 1.5G and php-apache limit is 250M so it's possible to run 6 replicas within quota. If memory was not a limit maximum replicas could be 8 because it would be limited by CPU quota set to 2000m, while deployment limit is 250m.
  - Target CPU Utilization: 50%
- **File**: [`php-apache-hpa.yml`](php-apache-hpa.yml)

### **5. Run and verify**
- **Runing**:
    ```bash
        minikube addons enable metrics-server
        kubectl apply -f namespace.yml
        kubectl apply -f resource-quota.yml
        kubectl apply -f worker-pod.yml
        kubectl apply -f php-apache-deployment.yml
        kubectl apply -f php-apache-hpa.yml
    ```
- **Verification**:
    ```bash
        kubectl get namespaces
        kubectl get resourcequotas -n zad1
        kubectl describe resourcequota zad1-quota -n zad1
        kubectl get pods -n zad1
        kubectl describe pod worker -n zad1
        kubectl get deployments -n zad1
        kubectl describe deployment php-apache -n zad1
        kubectl get services -n zad1
        kubectl describe service php-apache-service -n zad1
        kubectl get hpa -n zad1
        kubectl describe hpa php-apache-hpa -n zad1
    ```


### **6. Load Testing**
- **Objective**: Generate load to test the HPA functionality.
- **How to Run**:
  - ```bash
    kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache-service.zad1.svc.cluster.local; done"
    ```
  - Then open second terminal and run:
    ```bash
    kubectl get hpa -n zad1 --watch
    ```


## **Bonus Task**

### **1. Updating the Application with HPA Enabled**
- **Answer**: Yes, it is possible to update an application under HPA. 
Reference: [Autoscaling during rolling update](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#autoscaling-during-rolling-update).
  

### **2. Rolling Update Strategy**
- **Parameters**:
  - `maxSurge`: 1  - allows 1 additional replica above the desired count during updates
  - `maxUnavailable`: 0 - ensures no disabled replicas during update
- **HPA Adjustment**: It's necessary to lower maxReplicas to 5 in order to allow 1 additional replica to run.