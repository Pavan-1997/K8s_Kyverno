# K8s Kyverno

Kyverno is a policy engine designed for Kubernetes. It can validate, mutate, and generate configurations using admission controls and background scans. Kyverno policies are Kubernetes resources and do not require learning a new language. Kyverno is designed to work nicely with tools you already use like kubectl, kustomize, and Git. 

It can enforce policies, governence and compliance on your kubernetes cluster. Whether your kubernetes cluster is on AWS, Azure, GCP or on-premises

![image](https://github.com/Pavan-1997/K8s_Kyverno/assets/32020205/fb6b8fa4-0958-4fa4-8647-af7078ea1d72)

Policies can be defined as cluster-wide resources (using the kind ClusterPolicy) or namespaced resources (using the kind Policy.) As expected, namespaced policies will only apply to resources within the namespace in which they are defined while cluster-wide policies are applied to matching resources across all namespaces. Otherwise, there is no difference between the two types. Additional policy types include PolicyException and (Cluster)CleanupPolicy which are separate resources.

![image](https://github.com/Pavan-1997/K8s_Kyverno/assets/32020205/a0a8ad7a-5cdf-4a7d-bf55-2d401194b4c8)


We can use Kyverno in following configurations:

1. Generate -> For example, Create a default network policy whenever a namespace is created.
2. Validate -> For example, Block users from using latest tag in the deployment or pod resources.
3. Mutate -> For example, Attach pod security policy for a pod that is created without any pod security policy configuration.
4. Verify Images -> For example, Verify if the Images used in the pod resources are properly signed and verified images.

---

# Practical Implementation: 

A DevOps Engineer will write the required Kyverno Policy custom resource and commits it to a Git repository. Argo CD which is pre configured with auto-sync to watch for resources in the git repo, deploys the Kyverno Policies on to the Kubernetes cluster.

![image](https://github.com/Pavan-1997/K8s_Kyverno/assets/32020205/cbe59150-4be2-4896-9a25-a41d3c750a4c)

## Installation:

To setup this project you need to install Argo CD controller and Kyverno controller, Assuming you have Kubernetes installed.

Installation of both Kyverno and Argo CD are pretty straight forward as both of them support Helm charts and also provide a consolidated 
installation yaml files. 

### Kyverno - 

There are two easy ways to install kyverno:

1. Using Helm
2. Using the kubernetes manifest files

### Using Helm 

Add helm repo for kyverno 

```
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
```

Install kyverno in HA mode

```
 helm install kyverno kyverno/kyverno -n kyverno --create-namespace --set replicaCount=3
```

(or)

Install kyverno in Standalone mode

```
helm install kyverno kyverno/kyverno -n kyverno --create-namespace --set replicaCount=1
```

Install a specific version of kyverno

```
helm search repo kyverno -l | head -n 10
```

```
helm install kyverno kyverno/kyverno -n kyverno --create-namespace --version 2.6.5
```

### Using Kubernetes manifest yaml files

```
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.8.5/install.yaml
```

### Argo CD -

There are three ways to install Argo CD

1. Through Kubernetes manifest yaml files
```
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/install.yaml`
```
2. Helm Charts
```
https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd#installing-the-chart
```
3. Using the Argo CD Operator
```
https://argocd-operator.readthedocs.io/en/latest/install/olm/)
```
## Implementing and Verifying:

1. Installing KYVERNO using manifest file
```
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.8.5/install.yaml
```


2. Installing ArgoCD using manifest file
```
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/master/manifests/install.yaml
```


3. Use the YAML file `pod-requests-limits.yml` in the repo for ClusterPolicy with kyverno as Pod Requests Limits


4. Now there will a Kyverno Pod that is running in Kyverno namespace (we have saperate namespace for easy management)
```
kubectl get pods -A | grep kyverno
```
![image](https://github.com/Pavan-1997/K8s_Kyverno/assets/32020205/eabde41c-0c20-458b-a485-bb5e2dc19728)


5. Check for the logs 
```
kubectl logs kyverno-69f6c6485c-qwd66 -n kyverno 
```
`There will be a Dynamic Kubernetes Admission Controller that reads the policy according to definition that we create depending on which later it creates any pods or locks the creation`


6. Deploying nginx and check 
```
kubectl create deploy nginx --image=nginx
```
Below image shows that the POD will not be created due to the policy we applied with Kyverno

![NGINX_AFTER](https://github.com/Pavan-1997/K8s_Kyverno/assets/32020205/269408eb-cef2-4b4f-9733-7d498d23deda)

```
kubectl get pods | grep nginx
```
```
kubectl logs kyverno-69f6c6485c-qwd66 -n kyverno 
```

If validationFailureAction is set to Audit then change it to Enforce

`kubectl edit ClusterPolicy require-requests-limits (Change audit to enforce)`


7. Remove the ClusterPolicy 
```
kubectl get clusterpolicy
```
```
kubectl delete clusterpolicy <clusterpolicy-name>
```
Checking the Kyverno logs after the policy delete using `kubectl logs kyverno-69f6c6485c-qwd66 -n kyverno`

![CP_DELETE](https://github.com/Pavan-1997/K8s_Kyverno/assets/32020205/4dc586e3-f509-4105-b5db-038bb127f44f)


8. Install Nginx again which should install

   Follow again the steps in Step 6.

Now you can see the Nginx Pod is created without any policy 
![NGINX_AFTER](https://github.com/Pavan-1997/K8s_Kyverno/assets/32020205/fdaba1e5-3e68-4079-8757-7236f8ad2a4f)

