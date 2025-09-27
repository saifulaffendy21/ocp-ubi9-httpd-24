# UBI9 httpd Demo on OpenShift 4.14

This repository contains a minimal set of OpenShift resources that deploy the Red Hat UBI 9 based httpd 2.4 container image, expose it through a `Service`, and publish it externally through a TLS-terminated `Route`. It is intended as a quick way to validate a working OpenShift 4.14 cluster and demonstrate a basic GitOps-style workflow.

## Prerequisites

- An OpenShift Container Platform 4.14 cluster with cluster-admin or project-admin access.
- The `oc` CLI (`oc` 4.14 or later) logged in against the target cluster.
- Access to the image registry referenced in the `Deployment` (`registry.ocp4.example.com:8443/ubi9/httpd-24:latest`). Update the manifest if you need to use a different registry or tag.

## Repository Contents

| File | Description |
| --- | --- |
| `httpd-config.yaml` | Creates a ConfigMap that provides a simple `index.html` landing page. |
| `httpd-deployment` | Deploys two replicas of the UBI 9 httpd 2.4 container and mounts the ConfigMap data. |
| `httpd-service.yaml` | Exposes the pods inside the cluster on port 80. |
| `httpd-route.yaml` | Publishes the service externally with edge TLS termination and HTTP to HTTPS redirects. |

## Deployment Steps

1. **Create (or switch to) the target project/namespace.**
   ```bash
   oc new-project web
   # If the project already exists, run: oc project web
   ```

2. **Apply the ConfigMap.**
   ```bash
   oc apply -f httpd-config.yaml
   ```

3. **Deploy the application.**
   ```bash
   oc apply -f httpd-deployment
   ```

4. **Expose the deployment internally through a Service.**
   ```bash
   oc apply -f httpd-service.yaml
   ```

5. **Create the external Route.**
   ```bash
   oc apply -f httpd-route.yaml
   ```

## Validation and Testing

1. **Confirm the pods are running and ready.**
   ```bash
   oc get pods -n web
   oc rollout status deployment/httpd -n web
   ```

2. **Check service and route details.**
   ```bash
   oc get svc/httpd -n web
   oc get route/httpd -n web
   ```

3. **Test application access.**
   ```bash
   ROUTE_HOST=$(oc get route httpd -n web -o jsonpath='{.spec.host}')
   curl -k https://$ROUTE_HOST
   ```
   You should see the `It works — UBI9 httpd on OpenShift` message from the ConfigMap-provided `index.html` page.

4. **View logs if troubleshooting is required.**
   ```bash
   oc logs deployment/httpd -n web
   ```

## Customization Tips

- **Update site content:** Edit `httpd-config.yaml` and re-apply the manifest to replace the landing page.
- **Change replica count:** Modify the `replicas` value in `httpd-deployment` or run `oc scale deployment/httpd --replicas=<count> -n web`.
- **Adjust resource requests/limits:** Tune the values in the `resources` block of the deployment manifest to suit your environment.
- **Swap container image:** Replace the `image:` reference with one that is accessible from your cluster.

## Cleanup

Remove all deployed resources when finished:

```bash
oc delete -f httpd-route.yaml
oc delete -f httpd-service.yaml
oc delete -f httpd-deployment
oc delete -f httpd-config.yaml
oc delete project web
```

> **Note:** Deleting the project removes any other resources that were created in the same namespace.

## Additional Notes

- All manifests were validated on OpenShift Container Platform 4.14.
- The deployment enables OpenShift's default security model by dropping all Linux capabilities and running as a non-root UID assigned by the platform.
