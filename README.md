# K8s Kyverno

Kyverno is a policy engine designed for Kubernetes. It can validate, mutate, and generate configurations using admission controls and background scans. Kyverno policies are Kubernetes resources and do not require learning a new language. Kyverno is designed to work nicely with tools you already use like kubectl, kustomize, and Git. 

It can enforce policies, governence and compliance on your kubernetes cluster. Whether your kubernetes cluster is on AWS, Azure, GCP or on-premises

We can use Kyverno in following configurations:

1. Generate -> For example, Create a default network policy whenever a namespace is created.
2. Validate -> For example, Block users from using latest tag in the deployment or pod resources.
3. Mutate -> For example, Attach pod security policy for a pod that is created without any pod security policy configuration.
4. Verify Images -> For example, Verify if the Images used in the pod resources are properly signed and verified images.

---

Practical Implementation : A DevOps Engineer will write the required Kyverno Policy custom resource and commits it to a Git repository. Argo CD which is pre configured with auto-sync to watch for resources in the git repo, deploys the Kyverno Policies on to the Kubernetes cluster.

![image](https://github.com/Pavan-1997/K8s_Kyverno/assets/32020205/cbe59150-4be2-4896-9a25-a41d3c750a4c)


---
