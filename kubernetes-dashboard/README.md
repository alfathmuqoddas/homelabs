## Content

- dashboard-ingress.yaml -> attempt buat bikin ingress, tp gagal karena harus pake https, jadinya pake port forward
- token.txt -> generate dari service account token, untuk loginke kubernetes-dashboard

## Cara install Kubernetes Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

## Cara generate token

1. Create service account
   ```bash
   kubectl create serviceaccount admin-user -n kubernetes-dashboard
   ```
2. Assign Admin privileges:
   ```bash
   kubectl create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-user
   ```
3. Generate the Login Token:
   ```bash
   kubectl -n kubernetes-dashboard create token admin-user
   ```
