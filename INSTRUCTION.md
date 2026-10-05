# RBAC: ServiceAccounts, Roles, and Bindings

## 1. How to deploy all resources
Run the provided bootstrap script to provision the local cluster and deploy all the manifests.
```bash
sh bootstrap.sh
```

## 2. Architecture & RBAC Security Rationale
The RBAC implementation follows the **Principle of Least Privilege**:
* **Granular Role Definitions:** The `.infrastructure/security/rbac.yml` defines a strict `Role` (`secret-reader`) allowing *only* the `list` verb on the `secrets` resource within the `todoapp` namespace.
* **ServiceAccount Injection:** We explicitly configure the Deployment to use `serviceAccountName: todoapp-sa`.

## 3. How to validate the changes

### Step A: Exec into the Pod
First, retrieve the name of your running application pod and open a shell inside it:
```bash
kubectl get pods -n todoapp
kubectl exec -it <pod-name> -n todoapp -- /bin/sh
```

### Step B: Validate API Access
Within the pod shell, export the dynamically mounted ServiceAccount token and issue a cURL request to the K8s API server to list the secrets in the namespace:
```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $TOKEN" [https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets](https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets)
```
*Expected Result:* The API will respond with a JSON object (`kind: SecretList`) detailing all secrets in the namespace. A screenshot of this successful output is attached to the Pull Request.