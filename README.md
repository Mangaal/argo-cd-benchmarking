# Benchmarking Tools for ArgoCD and ArgoWorkflows

## Current Tools

### ArgoCD/ArgoWorkflows tooling based on Amazon EKS - [Link](tools/awsbenchmarking-nodejs)
This tooling is built on nodejs and utilizes various AWS tools to deploy and run benchmarks for both ArgoCD and ArgoWorkflows on Amazon EKS. This is the same tooling utilized to capture data for the following blog posts/talks:

* [Argo CD Application Controller Scalability Testing on Amazon EKS](https://aws.amazon.com/blogs/opensource/argo-cd-application-controller-scalability-testing-on-amazon-eks/)
* [Argo CD Benchmarking - Pushing the Limits and Sharding Deep Dive](https://cnoe.io/blog/argo-cd-application-scalability)
* [Argo Workflows Controller Scalability Testing on Amazon EKS](https://cnoe.io/blog/argo-workflow-scalability)
* [Adobe/AWS: Key Takeaways from Scaling Adobe's CI/CD Solution to Support 50K Argo CD Apps ](https://www.youtube.com/watch?v=7yVXMCX62tY)

### [Kube-Burner](./tools/kube-burner)

[Kube-Burner](https://kube-burner.github.io/kube-burner/latest/) is a resource creation and metrics collection tool. It is used to test ArgoCD performance by creating multiple ArgoCD Applications and collecting performance metrics such as CPU, memory, and API server requests. For detailed setup and usage, check the [Kube-Burner README](./tools/kube-burner/README.md).


