# RBAC: ServiceAccounts, Roles, and Bindings

## 1. How to deploy all resources
Run the provided bootstrap script to provision the local cluster and deploy all the manifests (database, security rules, and application workloads). Note for Windows users: you can execute the commands inside `bootstrap.sh` sequentially if the `sh` command is not supported natively.
```bash
sh bootstrap.sh
```

## 2. Architecture & RBAC Security Rationale
The RBAC implementation follows the **Principle of Least Privilege**:
* **Granular Role Definitions:** The `.infrastructure/security/rbac.yml` defines a strict `Role` (`secret-reader`) allowing *only* the `list` verb on the `secrets` resource within the specific namespace, actively avoiding dangerous wildcards (`*`).
* **ServiceAccount Injection:** We explicitly configure the Deployment to use `serviceAccountName: todoapp-sa`.
* **Production Best Practices Context:**
  * **Token Security:** In modern Kubernetes (v1.24+), the injected tokens are no longer static, long-lived Secrets. K8s leverages *Projected Volumes* to generate short-lived, pod-bound tokens that instantly invalidate if the pod is deleted.
  * **Default Tokens:** In a production setting, we would set `automountServiceAccountToken: false` on the default ServiceAccount to prevent unauthorized API access in case of a pod breach.
  * **Cloud Integration:** For real-world cloud scenarios (AWS/GCP), ServiceAccounts would be linked directly to cloud IAM roles (e.g., IRSA in AWS) rather than relying on K8s Secrets for external infrastructure access.

## 3. How to validate the changes

### Step A: Exec into the Pod
First, retrieve the name of your running application pod and open a shell inside it:
```bash
kubectl get pods -n mateapp
kubectl exec -it <pod-name> -n mateapp -- /bin/sh
```

### Step B: Validate API Access
Within the pod shell, export the dynamically mounted ServiceAccount token and issue a cURL request to the K8s API server to list the secrets in the namespace:
```bash
# 1. Export the token
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

# 2. Query the API
curl -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $TOKEN" [https://kubernetes.default.svc/api/v1/namespaces/mateapp/secrets](https://kubernetes.default.svc/api/v1/namespaces/mateapp/secrets)
```
*Expected Result:* The API will respond with a JSON object (`kind: SecretList`) detailing all secrets in the namespace, confirming the RBAC policies correctly granted access. A screenshot of this successful output is attached to the Pull Request.