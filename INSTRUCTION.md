# INSTRUCTION.md

## Prerequisites

Before validating the changes, ensure that the Kubernetes cluster is running and all infrastructure components have been deployed by executing the bootstrap script:

```bash
./bootstrap.sh

```

## Step 1: Verify RBAC Resource Creation

Confirm that the ServiceAccount, Role, and RoleBinding have been successfully created within the `todoapp` namespace:

```bash
kubectl get serviceaccount todoapp-sa -n todoapp
kubectl get role secret-reader-role -n todoapp
kubectl get rolebinding todoapp-rb -n todoapp

```

## Step 2: Verify Deployment Configuration

Verify that the `todoapp` deployment is using the newly created `todoapp-sa` ServiceAccount:

```bash
kubectl get deployment todoapp -n todoapp -o jsonpath='{.spec.template.spec.serviceAccountName}'

```

*Expected output:* `todoapp-sa`

## Step 3: Validate Secret Listing via API

1. Retrieve the name of one of the active running application pods:
```bash
kubectl get pods -n todoapp -l app=todoapp

```


2. Access the container's shell (replace `<pod-name>` with your actual pod name):
```bash
kubectl exec -it <pod-name> -n todoapp -- /bin/sh

```


3. Inside the pod, execute the following commands to send an authenticated request to the Kubernetes API server using the z-mounted ServiceAccount token:
```sh
APISERVER="https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_PORT_443_TCP_PORT"
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

curl --cacert $CACERT --header "Authorization: Bearer $TOKEN" -X GET "$APISERVER/api/v1/namespaces/todoapp/secrets"

```



*Expected output:* A JSON object containing the list of secrets within the `todoapp` namespace (including `app-secret`), confirming status `200 OK`.
