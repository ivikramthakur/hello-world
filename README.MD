# HelloWorld NGINX Kubernetes Deployment (GitOps with Argo CD)

This project demonstrates deploying an NGINX web server to Kubernetes using GitOps principles with [Argo CD](https://argo-cd.readthedocs.io/).

## Repository Structure

- [`nginx-helloworld.yaml`](nginx-helloworld.yaml): Kubernetes manifest for the NGINX Deployment and Service.

## Prerequisites

- A running Kubernetes cluster
- [Argo CD](https://argo-cd.readthedocs.io/) installed and configured
- Access to this Git repository

## Deployment via Argo CD (GitOps)

1. **Push your changes to your Git repository.**  
   Argo CD will track this repository for changes.

2. **Create an Argo CD Application:**

   You can create an Application via the Argo CD UI or by applying a manifest.  
   Example manifest (`argocd-app.yaml`):

   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: hello-nginx
     namespace: argocd
   spec:
     project: default
     source:
       repoURL: 'https://github.com/crisinis/argocd-helloworld.git'
       targetRevision: HEAD
       path: .
     destination:
       server: 'https://kubernetes.default.svc'
       namespace: default
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
   ```

   - This manifest will deploy the NGINX application defined in [`nginx-helloworld.yaml`](nginx-helloworld.yaml) using the configuration and templating described above.
   - The `path: .` field ensures Argo CD deploys the manifests from the root of this repository, including the actual application files you see here.

   Apply it with:
   ```sh
   kubectl apply -f argocd-app.yaml
   ```

3. **Argo CD will automatically deploy and manage NGINX.**

   - Any changes pushed to this repo will be automatically synced to your cluster by Argo CD.
   - You can monitor and manage the deployment via the Argo CD UI.

## Accessing NGINX

1. Get the NodePort:
   ```sh
   kubectl get service hello-nginx
   ```
2. Access NGINX at `http://<NODE_IP>:<NODE_PORT>`

## CI/CD with GitOps

- **Continuous Integration:**  
  Make changes to your manifests and push to Git.
- **Continuous Deployment:**  
  Argo CD automatically applies changes from Git to your cluster.

## Configuration

This deployment and service manifest uses Helm-style templating for the nginx container port. The port is set using the `nginxPort` variable, which is defined in `values.yaml`:

```yaml
nginxPort: 80
```

All references to the container port and probe ports use `{{ .Values.nginxPort | default 80 }}` for flexibility. To change the port, update the value in `values.yaml`.

### Probes and Resources
- **Readiness and liveness probes** are configured to check the root path (`/`) on the configured port.
- **Resource requests and limits** are set for CPU and memory to ensure proper scheduling and resource control.

### Usage
1. Edit `values.yaml` to set your desired port.
2. Apply the manifest with your preferred tool (e.g., `kubectl apply -f nginx-helloworld.yaml`).

## File Explanations for Beginners

### kustomization.yaml
This file is used by [Kustomize](https://kustomize.io/), a tool built into `kubectl` that helps you customize Kubernetes resources without modifying the original YAML files. In this project:
- It tells Kustomize to deploy the resources defined in `nginx-helloworld.yaml`.
- It generates a ConfigMap named `nginx-config` from the local `nginx.conf` file, making it available to the NGINX deployment.
- The `namespace: default` line ensures resources are created in the `default` namespace.

To apply everything with Kustomize, run:
```sh
kubectl apply -k .
```

### nginx-helloworld.yaml
This is the main Kubernetes manifest for the project. It contains:
- A **Deployment** for running the NGINX web server, including resource limits, health checks, and configuration from the ConfigMap.
- A **Service** to expose NGINX to your network, using a NodePort for external access.
- Helm-style templating is used for the port, making it easy to change via `values.yaml` if you use Helm.

If you are not using Kustomize, you can apply this file directly:
```sh
kubectl apply -f nginx-helloworld.yaml
```

## Cleanup

To remove the deployment:
```sh
kubectl delete -f argocd-app.yaml
```
Or delete the Application from the Argo CD UI.