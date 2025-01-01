# EKS Cluster with Fargate Profile

This project demonstrates how to create an AWS Elastic Kubernetes Service (EKS) cluster with Fargate profiles. It involves creating the required IAM roles, configuring a Fargate profile, and deploying an example application to the EKS cluster using Fargate.

## Technologies
- **Kubernetes**
- **AWS EKS**
- **AWS Fargate**

---

## Project Description
This project consists of the following steps:
1. Creating an IAM Role for Fargate.
2. Configuring a Fargate Profile in an existing EKS cluster.
3. Deploying a sample application using the Fargate profile.
4. Cleaning up resources to avoid unnecessary charges.

---

## Prerequisites
- AWS CLI installed and configured.
- `kubectl` installed and configured for your EKS cluster.
- An existing EKS cluster.
- Proper permissions to create IAM roles, Fargate profiles, and manage Kubernetes resources.

---

## Steps

### 1. Create IAM Role for Fargate
1. Navigate to **IAM** in the AWS Management Console.
2. Click **Roles** > **Create Role**.
3. Select **AWS Service** and choose **EKS** as the use case.
4. Select **EKS - Fargate pod** and click **Next**.
5. Verify that `AmazonEKSFargatePodExecutionRolePolicy` is pre-selected.
6. Provide a name for the role, e.g., `eks-fargate-role`, and click **Create role**.

---

### 2. Create Fargate Profile
1. Open your EKS cluster (e.g., `eks-cluster-test`) in the AWS Management Console.
2. Go to **Compute** and click **Add Fargate profile**.
3. On the **Configure Fargate profile** page:
   - Enter a name, e.g., `dev-profile`.
   - Select `eks-fargate-role` as the **Pod execution role**.
   - Choose private subnets from your VPC for worker nodes.
4. Click **Next**.
5. In **Pod selectors**, enter:
   - Namespace: `dev`
   - Match Labels:
     - Key: `profile`
     - Value: `fargate`
   - (Optional: Add up to 4 more labels.)
6. Review the configuration and click **Create**.
7. Verify the profile is **Active** in the console.

---

### 3. Deploy Pod through Fargate
1. Check for existing pods running in EC2 instance node groups:
   ```bash
   kubectl get pods
   ```
2. Create a new namespace for Fargate:
   ```bash
   kubectl create ns dev
   ```
3. Deploy an application to the `dev` namespace using a configuration file (`nginx-config.yaml`):
   ```bash
   kubectl apply -f nginx-config.yaml
   ```
   Output:
   ```
   deployment.apps/nginx created
   service/nginx unchanged
   ```
4. Verify the pod status in the `dev` namespace:
   ```bash
   kubectl get pod -n dev
   ```
5. Check the nodes hosting the pods:
   ```bash
   kubectl get nodes -n dev
   ```
   Fargate nodes will be displayed in addition to the EC2 nodes.
6. Update the configuration to increase replicas to 3 and reapply:
   ```bash
   kubectl apply -f nginx-config.yaml
   ```
   Note: In Fargate, each pod runs in its own virtual machine.

7. Verify the pods and nodes:
   ```bash
   kubectl get pods -n dev -o wide
   kubectl get nodes
   ```
   Output will show 3 Fargate nodes and the existing EC2 nodes.

---

### 4. Cleanup Cluster Resources
1. Delete the node group and Fargate profiles:
   ```bash
   eksctl delete nodegroup --cluster eks-cluster-test --name <nodegroup-name>
   ```
2. Delete the cluster:
   ```bash
   eksctl delete cluster --name eks-cluster-test
   ```
3. Remove IAM roles:
   - `eks-cluster-role`
   - `eks-fargate-role`
   - `eks-node-group-role`
     
4. See that instances are in 'Terminated' state in the EC2 console.
   
6. Delete the CloudFormation stack:
   - Navigate to **CloudFormation** in the AWS Console.
   - Select and delete the `eks-worker-node-vpc-stack`.

---

## Example Configuration (`nginx-config.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namspace: dev
  labels:
    app: nginx
spec:
  replicas: 2
  selector: 
    matchLabels:
      app: nginx
      profile: fargate
  template:
    metadata:
      labels:
        app: nginx
        profile: fargate
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80 
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer


---

## Notes
- Multiple Fargate profiles can be created to support different namespaces or workloads.
- In Fargate, each pod is deployed on its own isolated virtual machine, enhancing security and scalability.

---


