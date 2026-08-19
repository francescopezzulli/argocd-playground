# ArgoCD Playground

A GitOps repository for managing OpenShift applications using ArgoCD.

## Structure

```
.
├── applications/           # ArgoCD Application manifests
│   └── oidc-client-debugger.yaml
└── manifests/             # Kubernetes/OpenShift resource manifests
    └── oidc-client-debugger/
        ├── deployment.yaml
        ├── service.yaml
        └── route.yaml
```

## Applications

### OIDC Client Debugger

A Java application for debugging OIDC/OAuth flows.

- **Namespace**: `francesco-onboarding`
- **Sync Policy**: Automated (auto-sync and self-heal enabled)
- **Manifest Path**: `manifests/oidc-client-debugger/`

## Prerequisites

- OpenShift cluster access
- ArgoCD/OpenShift GitOps installed
- Proper RBAC permissions for ArgoCD service account

## Setup

### 1. Grant ArgoCD Permissions

ArgoCD needs permissions to deploy to your namespace:

```bash
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: argocd-admin
  namespace: francesco-onboarding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: openshift-gitops-argocd-application-controller
  namespace: openshift-gitops
EOF
```

### 2. Deploy Applications

Applications are managed declaratively through Git. Simply push to the `main` branch and ArgoCD will automatically sync.

```bash
git add .
git commit -m "Update manifests"
git push origin main
```

### 3. Manual Sync (if needed)

```bash
argocd app sync openshift-gitops/oidc-client-debugger
```

## Monitoring

Check application status:

```bash
# List all applications
argocd app list

# Get detailed status
argocd app get openshift-gitops/oidc-client-debugger

# View resources in namespace
kubectl get all -n francesco-onboarding
```

## Adding New Applications

1. Create manifest directory under `manifests/<app-name>/`
2. Add Kubernetes/OpenShift resources (deployment, service, route, etc.)
3. Create ArgoCD Application in `applications/<app-name>.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <app-name>
  namespace: openshift-gitops
spec:
  project: default
  source:
    repoURL: https://github.com/francescopezzulli/argocd-playground.git
    targetRevision: main
    path: manifests/<app-name>
  destination:
    server: https://kubernetes.default.svc
    namespace: <target-namespace>
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

4. Commit and push to Git

## Troubleshooting

### Application Not Syncing

Check the application status for errors:
```bash
argocd app get openshift-gitops/<app-name>
```

### Permission Denied Errors

Ensure ArgoCD has proper RBAC permissions in the target namespace (see Setup step 1).

### Resources Not Appearing

Verify:
- Manifests are valid YAML
- Namespace exists
- Image references are correct
- Port configurations match between Service and Deployment

## Repository

- **GitHub**: https://github.com/francescopezzulli/argocd-playground
- **ArgoCD UI**: Check your OpenShift GitOps route
