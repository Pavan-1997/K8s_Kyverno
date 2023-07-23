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


3. Create a below YAML file with below for ClusterPolicy with kyverno as Pod Requests Limits using `vi <filename>.yaml`
```
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-requests-limits
  annotations:
    policies.kyverno.io/title: Require Limits and Requests
    policies.kyverno.io/category: Best Practices, EKS Best Practices
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/description: >-
      As application workloads share cluster resources, it is important to limit resources
      requested and consumed by each Pod. It is recommended to require resource requests and
      limits per Pod, especially for memory and CPU. If a Namespace level request or limit is specified,
      defaults will automatically be applied to each Pod based on the LimitRange configuration.
      This policy validates that all containers have something specified for memory and CPU
      requests and memory limits.
spec:
  validationFailureAction: enforce
  background: true
  rules:
  - name: validate-resources
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "CPU and memory resource requests and limits are required."
      pattern:
        spec:
          containers:
          - resources:
              requests:
                memory: "?*"
                cpu: "?*"
              limits:
                memory: "?*"
```			
