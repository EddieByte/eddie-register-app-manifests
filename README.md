# Eddie Register App Manifests

Infrastructure-as-Code (IaC) repository for the Eddie Register App. This repo uses **Kustomize** to manage per-environment Kubernetes settings and **ArgoCD** to handle GitOps-based deployment to the cluster.

## What the Application Does

The refactored application is a small Java web registration workflow with the following user-facing flow:

- **Home page** with a welcome message and a call-to-action to create an account.
- **Registration form** that collects:
  - Full name
  - Mobile number
  - Email address
  - Password
  - Password confirmation
- **Servlet-based submission** that handles the POST request from the registration form.
- **Thank-you page** shown after a successful registration submission.
- **About page** describing the project and the deployment pipeline context.

This repository is the Kubernetes and GitOps layer that deploys that application into an EKS-ready environment.

## Repository Structure

- `base/`: Shared Kubernetes manifests used across all environments.
- `overlays/`: Environment-specific Kustomize overlays.
  - `dev/`: Development overlay.
  - `staging/`: Staging overlay.
  - `prod/`: Production overlay.
- `argocd/`: ArgoCD `Application` manifests used to bootstrap deployment from Git.

## Deployment Model

The manifests define a simple application deployment stack:

- A **Deployment** running the `eddie-register-app` container image.
- A **Service** exposing the app internally through the cluster.
- An **Ingress** fronted by an AWS ALB to make the app reachable from outside the cluster.
- **ArgoCD Application** resources for `dev` and `prod` synchronization.

## Getting Started

### 1. Preview the rendered manifests locally

You can render the manifests directly with `kubectl` / `kustomize`:

```bash
# Preview development environment
kubectl kustomize overlays/dev

# Preview production environment
kubectl kustomize overlays/prod
```

### 2. Apply a specific environment overlay

```bash
kubectl apply -k overlays/dev
kubectl apply -k overlays/prod
```

## Accessing the Application

Once deployed, the application is exposed through the AWS Application Load Balancer defined by the ingress resource.

### 1. Find the external URL

```bash
kubectl get ingress -n eddie-register-app-dev
```

Look for the **ADDRESS** column and copy the DNS name for the ALB.

### 2. Wait for DNS propagation

AWS may take a few minutes to provision and publish the load balancer DNS name. During that time, the app may return a timeout or brief 404 response.

### 3. Open the app in a browser

Use the DNS name shown by the ingress to reach the application and test the registration workflow.

---

## GitOps Workflow

1. **Modify** the Kubernetes manifests or Kustomize overlays.
2. **Commit and push** the change to GitHub.
3. **Sync** ArgoCD automatically detects the new state and reconciles the cluster.
4. **Observe** the deployment status and app availability in the ArgoCD dashboard.

## Notes

This repo is intentionally focused on the deployment and environment management layer for the registration app. The actual app behavior and screens are implemented in the application source repository, while this repo supplies the Kubernetes configuration needed to run it in a GitOps environment.
