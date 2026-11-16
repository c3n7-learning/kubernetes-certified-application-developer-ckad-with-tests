# A Quick Note on Editing Pods and Deployments

## Edit a Pod

Remember, you **cannot** edit the specification of an existing Pod, other than the following fields:

- `spec.containers[*].image`
- `spec.initContainers[*].image`
- `spec.activeDeadlineSeconds`
- `spec.tolerations`

For example, you cannot edit the environment variables, service accounts, or resource limits (all of which we will discuss later) of a running Pod. If you really need to, you have two options.

### Option 1: Use `kubectl edit`

1. Run the edit command. This opens the Pod specification in an editor (vi).

   ```bash
   kubectl edit pod <pod name>
   ```

2. Edit the required properties. When you try to save, you will be denied, because you are attempting to change a field that is not editable. A copy of the file with your changes is saved in a temporary location.

3. Delete the existing Pod:

   ```bash
   kubectl delete pod webapp
   ```

4. Create a new Pod from the temporary file:

   ```bash
   kubectl create -f /tmp/kubectl-edit-ccvrq.yaml
   ```

### Option 2: Export the definition to a file

1. Extract the Pod definition in YAML format:

   ```bash
   kubectl get pod webapp -o yaml > my-new-pod.yaml
   ```

2. Make your changes to the exported file and save:

   ```bash
   vi my-new-pod.yaml
   ```

3. Delete the existing Pod:

   ```bash
   kubectl delete pod webapp
   ```

4. Create a new Pod from the edited file:

   ```bash
   kubectl create -f my-new-pod.yaml
   ```

## Edit Deployments

With Deployments you can edit **any** field or property of the Pod template. Because the Pod template is a child of the Deployment specification, every change causes the Deployment to automatically delete and recreate the Pod with the new changes.

So if you are asked to edit a property of a Pod that is part of a Deployment, simply run:

```bash
kubectl edit deployment my-deployment
```
