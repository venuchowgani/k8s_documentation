### kubernetes workloads:
**Pod**:The most basic thing in Kubernetes is a Pod. A Pod runs one or more containers together. It’s like a small box that holds your app and runs it on a node. But we usually don’t use Pods alone—we use other things to manage them better.

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
spec:
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80

```
![alt text](.\images\k8s-1.png)

**ReplicaSet**:A ReplicaSet helps make sure a certain number of the same Pods are always running. If a Pod crashes, the ReplicaSet will create a new one to replace it. But we don’t usually use ReplicaSets directly—we use something called a Deployment to handle that.

**Example:**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: my-container
          image: nginx
          ports:
            - containerPort: 80
```
![alt text](.\images\k8s-2.png)

**Deployment**:A Deployment is one of the most common ways to run apps in Kubernetes. It keeps your app running, lets you easily update it, and even roll back if something goes wrong. You can also scale your app up or down by changing how many Pods it should run.

**Example**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx
          ports:
            - containerPort: 80
```
List all Deployments:
```bash
kubectl get deployments
```
Get details of a Deployment:
```bash
kubectl describe deployment <deployment name>
```
Check Pods created by the Deployment:
```bash
kubectl get pods -l app=myapp
```
Check rollout status:
```bash
kubectl rollout status deployment/<deployment name>
```
![alt text](.\images\k8s-3.png)

**DaemonSet**:A DaemonSet is used when you want the same Pod to run on every node in the cluster. For example, if you want to collect logs or monitor every machine, you can use a DaemonSet to do that.

**Example**:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
spec:
  selector:
    matchLabels:
      app: my-daemon
  template:
    metadata:
      labels:
        app: my-daemon
    spec:
      containers:
        - name: my-daemon-container
          image: busybox
          command: [ "sh", "-c", "while true; do echo DaemonSet running; sleep 30; done" ]
```
List DaemonSets:
```bash
kubectl get daemonsets
```
Describe DaemonSet details:
```bash
kubectl describe daemonset <deamonset name>
```
 List the Pods created by the DaemonSet:
 ```bash
 kubectl get pods -l app=my-daemon -o wide
```
View logs of a Pod:
```bash
kubectl logs <pod-name>
```
![alt text](.\images\k8s-4.png)

**Statefulset**:Sometimes you need apps that remember data and have a fixed identity—like a database. For that, Kubernetes gives us a StatefulSet. It gives each Pod a fixed name and storage so it doesn’t lose data or get confused about which one it is.

1.StatefulSets also create Pods one at a time, in order. For example, Pod 0 must be ready before Pod 1 is created. When shutting down, the reverse happens—1 terminates before 0.

2.This order guarantees that distributed systems that need to start in sequence (like Kafka or Elasticsearch) can be safely and predictably deployed.

**Persistent Volume (PV) and Persistent Volume Claim (PVC)**:
3.StatefulSets usually work together with persistent storage, so each Pod has its own disk space that is not lost after restart. This is where PV and PVC come in.

4.Persistent Volume (PV): A piece of storage in the cluster (like EBS, local disk, NFS) provided by the admin. Think of it like a physical disk.

5.Persistent Volume Claim (PVC): A request by a user (or Pod) to use a PV. It's like saying, "I need 1Gi of storage, with read/write access."

6.When a StatefulSet is created, it uses something called volumeClaimTemplates to automatically create a separate PVC for each Pod. For example, myapp-0 will get www-myapp-0, and myapp-1 will get www-myapp-1
Each PVC will be bound to its own PV, giving every Pod its own private storage that survives Pod restarts or rescheduling.
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-statefulset
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
```
List the Statefulsets:
```bash
kubectl get statefulsets
```
 Describe StatefulSet:
 ```bash
 kubectl describe statefulset <statefulset name>
 ```
 Check Pods with their stable names:
 ```bash
 kubectl get pods -l app=nginx
```
 Check associated PVCs:
 ```bash
 kubectl get pvc
```
![alt text](.\images\k8s-5.png)

**Job**:If you just want to run a one-time task, like backing up a database or sending a report, you can use a Job. It will run your task and then stop when it’s done.

**Example**:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
        - name: hello
          image: busybox
          command: ["echo", "Hello from Kubernetes Job!"]
      restartPolicy: Never
  backoffLimit: 2
```
 Check Job status:
 ```bash
 kubectl get jobs
```
View the logs:
```bash
kubectl logs <pod-name>
```
Check the Pod created by the Job:
```bash
kubectl get pods
```
![alt text](.\images\k8s-6.png)

**Cronjob**:Extending Jobs, CronJobs allow you to run Jobs on a fixed schedule, similar to the Linux cron system. You can use CronJobs to run periodic tasks like nightly backups, cleanup jobs, or scheduled report generation.

**Example**:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cronjob
spec:
  schedule: "*/1 * * * *"   # Runs every 1 minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              command: ["echo", "Hello from CronJob!"]
          restartPolicy: OnFailure
```
 Check the CronJob:
 ```bash
 kubectl get cronjobs
```
Wait 1-2 minutes, then list the Jobs it created:
```bash
kubectl get jobs
```
View the logs of one of the Pods:
```bash
kubectl logs <pod-name>
```
![alt text](.\images\k8s-7.png)

**Service**:In Kubernetes, a Service is an abstraction that defines a stable way to access Pods, even though Pods may come and go or change IP addresses. Services provide a consistent IP and DNS name, and they load balance traffic across multiple Pods.

Services select Pods using labels (selector:).They get a virtual IP address (ClusterIP) inside the cluster.kube-proxy runs on each node and configures routing rules.When traffic hits the Service IP, kube-proxy forwards it to one of the matched Pods.

This is the basic Nginx application.We can use this example for testing all types of services.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx
          ports:
            - containerPort: 80
```

**Types of Services in Kubernetes**:
1. ClusterIP (Default):
This is the default Service type in Kubernetes. It exposes the Service internally within the cluster—other Pods can access it, but users from outside the cluster cannot. It's commonly used for backend services, like when a frontend needs to talk to a backend or a database.

**Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: myapp # this hsould match with label of pod
  ports:
    - port: 80
      targetPort: 8080
```
Get the Service:
```bash
kubectl get svc
```
![alt text](.\images\k8s-8.png)
2.NodePort:
A NodePort Service exposes your application on a static port on every node's IP address. You can access it externally using NodeIP:NodePort. This is useful for quick access during testing or when not using a cloud LoadBalancer.

**Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007
```
![alt text](.\images\k8s-9.png)
3.LoadBalancer:
This type works in cloud environments like AWS or GCP. It creates an external load balancer that routes traffic to your Service. It’s the easiest way to expose an application publicly in the cloud, especially for production apps or APIs.

**Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```
![alt text](.\images\k8s-10.png)
To Access the application open any browser and type:
```bash
http://<External-IP>
```
4.ExternalName:This special type doesn’t route traffic to Pods. Instead, it returns a CNAME (DNS alias) to an external DNS name like example.com. It’s useful when your Pods need to reach an external service such as a managed database.

**Example**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: example.com
```
**Annotations**:
Annotations are key-value pairs that you can attach to any Kubernetes object (like Pods, Services, Deployments, etc.) to store extra metadata.
Unlike labels (which are used for selection and grouping), annotations are meant for informational purposes — for tools, automation, and humans to use.

**Example Use Cases for Annotations**:
1.Storing the last modified timestamp of a resource.
2.Adding external documentation links or notes.
3.Used by tools like Ingress controllers, service meshes, and monitoring agents.
4.Used to pass config values to cloud controllers (like service.beta.kubernetes.io/aws-load-balancer-type).

**Example**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"   # tells AWS to use NLB instead of ELB
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 80
```
Add or Edit Annotations:

```bash
kubectl annotate deployment myapp team=dev
```
```bash
kubectl annotate deployment myapp team=qa --overwrite
```
**Ingress**:
Ingress in Kubernetes is a resource that manages external access to services within a cluster, typically HTTP or HTTPS traffic. It allows you to define rules for routing traffic to different services based on URL paths, hostnames, or other criteria, without exposing multiple services to the outside world directly. Essentially, Ingress acts as a gateway that directs incoming requests to the appropriate services within the cluster.

**Why Ingress**:
Imagine you run multiple web applications (e.g., a blog, an e-commerce site, and an API) on the same Kubernetes cluster. Without Ingress, you’d need to expose each service with its own LoadBalancer or NodePort, which can be expensive and complex to manage. Ingress simplifies this by allowing you to route traffic to the correct service based on the URL path or hostname, all under a single external IP address.

Ingress doesn’t directly handle traffic — it needs an Ingress Controller, which is a tool (like NGINX or AWS ALB Ingress Controller) that watches the Ingress rules and routes the traffic properly.

Ingress also supports TLS, so you can handle HTTPS traffic. You just need a certificate, which can be provided by:

AWS ACM (for ALB Ingress),
cert-manager with Let’s Encrypt (for NGINX Ingress)

**Common Ingress Controllers**:
1.NGINX Ingress Controller (popular for most use cases).
2.AWS ALB Ingress Controller (great for EKS).
3.Traefik, HAProxy, etc.

**Installing steps AWS ALB Ingress Controller**:


1.Navigate to the cluster you created, click on the Overview tab, and copy the OpenID Connect provider URL.
![alt text](.\images\k8s-11.png)
2.Go to the IAM section in the AWS Console and click on Identity providers.
![alt text](.\images\k8s-12.png)
3.Click on Add provider to begin setting up a new identity provider.
![alt text](.\images\k8s-13.png)
4.Select OpenID Connect as the provider type, paste the OpenID Connect provider URL from your cluster into the Provider URL field, set the Audience to "sts.amazonaws.com", and then click on Add provider to complete the setup.
![alt text](.\images\k8s-14.png)
5.To create a new IAM policy, go to the IAM section in the AWS Management Console and click on Policies under Access management. Then, click the Create policy button.
![alt text](.\images\k8s-15.png)
6.In the policy creation wizard, select the JSON tab, then paste the following policy into the policy editor to define the required permissions.
```jason
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateServiceLinkedRole"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeAccountAttributes",
                "ec2:DescribeAddresses",
                "ec2:DescribeAvailabilityZones",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeVpcs",
                "ec2:DescribeVpcPeeringConnections",
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInstances",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DescribeTags",
                "ec2:GetCoipPoolUsage",
                "ec2:DescribeCoipPools",
                "ec2:GetSecurityGroupsForVpc",
                "ec2:DescribeIpamPools",
                "ec2:DescribeRouteTables",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeLoadBalancerAttributes",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeListenerCertificates",
                "elasticloadbalancing:DescribeSSLPolicies",
                "elasticloadbalancing:DescribeRules",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeTargetGroupAttributes",
                "elasticloadbalancing:DescribeTargetHealth",
                "elasticloadbalancing:DescribeTags",
                "elasticloadbalancing:DescribeTrustStores",
                "elasticloadbalancing:DescribeListenerAttributes",
                "elasticloadbalancing:DescribeCapacityReservation"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "cognito-idp:DescribeUserPoolClient",
                "acm:ListCertificates",
                "acm:DescribeCertificate",
                "iam:ListServerCertificates",
                "iam:GetServerCertificate",
                "waf-regional:GetWebACL",
                "waf-regional:GetWebACLForResource",
                "waf-regional:AssociateWebACL",
                "waf-regional:DisassociateWebACL",
                "wafv2:GetWebACL",
                "wafv2:GetWebACLForResource",
                "wafv2:AssociateWebACL",
                "wafv2:DisassociateWebACL",
                "shield:GetSubscriptionState",
                "shield:DescribeProtection",
                "shield:CreateProtection",
                "shield:DeleteProtection"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupIngress"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateSecurityGroup"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags"
            ],
            "Resource": "arn:aws:ec2:*:*:security-group/*",
            "Condition": {
                "StringEquals": {
                    "ec2:CreateAction": "CreateSecurityGroup"
                },
                "Null": {
                    "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags",
                "ec2:DeleteTags"
            ],
            "Resource": "arn:aws:ec2:*:*:security-group/*",
            "Condition": {
                "Null": {
                    "aws:RequestTag/elbv2.k8s.aws/cluster": "true",
                    "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:DeleteSecurityGroup"
            ],
            "Resource": "*",
            "Condition": {
                "Null": {
                    "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:CreateTargetGroup"
            ],
            "Resource": "*",
            "Condition": {
                "Null": {
                    "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:CreateRule",
                "elasticloadbalancing:DeleteRule"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:RemoveTags"
            ],
            "Resource": [
                "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*",
                "arn:aws:elasticloadbalancing:*:*:loadbalancer/net/*/*",
                "arn:aws:elasticloadbalancing:*:*:loadbalancer/app/*/*"
            ],
            "Condition": {
                "Null": {
                    "aws:RequestTag/elbv2.k8s.aws/cluster": "true",
                    "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:RemoveTags"
            ],
            "Resource": [
                "arn:aws:elasticloadbalancing:*:*:listener/net/*/*/*",
                "arn:aws:elasticloadbalancing:*:*:listener/app/*/*/*",
                "arn:aws:elasticloadbalancing:*:*:listener-rule/net/*/*/*",
                "arn:aws:elasticloadbalancing:*:*:listener-rule/app/*/*/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:ModifyLoadBalancerAttributes",
                "elasticloadbalancing:SetIpAddressType",
                "elasticloadbalancing:SetSecurityGroups",
                "elasticloadbalancing:SetSubnets",
                "elasticloadbalancing:DeleteLoadBalancer",
                "elasticloadbalancing:ModifyTargetGroup",
                "elasticloadbalancing:ModifyTargetGroupAttributes",
                "elasticloadbalancing:DeleteTargetGroup",
                "elasticloadbalancing:ModifyListenerAttributes",
                "elasticloadbalancing:ModifyCapacityReservation",
                "elasticloadbalancing:ModifyIpPools"
            ],
            "Resource": "*",
            "Condition": {
                "Null": {
                    "aws:ResourceTag/elbv2.k8s.aws/cluster": "false"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:AddTags"
            ],
            "Resource": [
                "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*",
                "arn:aws:elasticloadbalancing:*:*:loadbalancer/net/*/*",
                "arn:aws:elasticloadbalancing:*:*:loadbalancer/app/*/*"
            ],
            "Condition": {
                "StringEquals": {
                    "elasticloadbalancing:CreateAction": [
                        "CreateTargetGroup",
                        "CreateLoadBalancer"
                    ]
                },
                "Null": {
                    "aws:RequestTag/elbv2.k8s.aws/cluster": "false"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:DeregisterTargets"
            ],
            "Resource": "arn:aws:elasticloadbalancing:*:*:targetgroup/*/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:SetWebAcl",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:AddListenerCertificates",
                "elasticloadbalancing:RemoveListenerCertificates",
                "elasticloadbalancing:ModifyRule",
                "elasticloadbalancing:SetRulePriorities"
            ],
            "Resource": "*"
        }
    ]
}
```
![alt text](.\images\k8s-16.png)
7.Click on Next to proeed with other steps.
![alt text](.\images\k8s-17.png)
8.Enter a name for your policy, then scroll down slightly and click on Create policy to finalize and save it.
![alt text](.\images\k8s-18.png)
9.In the IAM console, click on Roles from the sidebar, then click on Create role to begin setting up a new IAM role.
![alt text](.\images\k8s-19.png)
10.Select Web identity as the trusted entity type, then choose the Identity provider and Audience from the dropdown menus using the values you created in the previous step.
![alt text](.\images\k8s-20.png)
11.In the condition section, choose the Key ending with :sub, set the Condition to StringEquals, and enter your service account as the Value (e.g,system:serviceaccount:kube-system:aws-load-balancer-controller). Then click Next to proceed.
![alt text](.\images\k8s-21.png)
12.Select the policy you created earlier from the list, then click Next to continue to the next step.
![alt text](.\images\k8s-22.png)
13.Provide a descriptive name for the role that reflects its purpose.
![alt text](.\images\k8s-23.png)
14.After naming the role, scroll down and click on Create role to finalize and create the IAM role.
![alt text](.\images\k8s-24.png)
15.Copy the arn of the role 
![alt text](.\images\k8s-27.png)
16.Create ServiceAccount with below yaml file. paste the arn of role created.
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-load-balancer-controller
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: <Role arn>
```
![alt text](.\images\k8s-28.png)
17.Add the eks-charts Helm chart repository. 
```bash
helm repo add eks https://aws.github.io/eks-charts
```
18.Update your local repo to make sure that you have the most recent charts.
```bash
helm repo update eks
```
19.Install the AWS Load Balancer Controller using helm
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=<name of ServiceAccount> \
  --version 1.13.0
  ```
![alt text](.\images\k8s-29.png)
20.Verify that the controller is installed
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
![alt text](.\images\k8s-30.png)
**Installing Nginx Ingress Controller**:

1. Create Namespace:
```bash
kubectl create namespace ingress-nginx
```
2.Apply the Official NGINX Ingress Controller YAML
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/aws/deploy.yaml
```
![alt text](.\images\k8s-31.png)
3.Verify the Ingress Controller Pods
```bash
kubectl get pods -n ingress-nginx
```
![alt text](.\images\k8s-32.png)
Check the Service
```bash
kubectl get svc -n ingress-nginx
```
**Example**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/backend-protocol: HTTP
    alb.ingress.kubernetes.io/subnets: <subnet-id1>,<subnet-id2> # <-- Replace with actual subnet IDs
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```
![alt text](.\images\k8s-34.png)
To Access Application open any browser and type
```bash
http://<ADDRESS>
```
![alt text](.\images\k8s-35.png)
**ConfigMaps**:A ConfigMap is a Kubernetes object used to store non-sensitive configuration data such as key-value pairs, environment variables, or entire configuration files. It's useful for separating configuration from application code, making your app more portable and easier to update.
**Use Cases**:
1.Store app configuration settings
2.Inject environment variables
3.Mount as config files into pods
4.Share configs across multiple pods

**Example1**:Create a ConfigMap from Literal Values
```bash
kubectl create configmap my-config --from-literal=APP_MODE=production --from-literal=APP_VERSION=1.0
```
Check it:
```bash
kubectl get configmap my-config -o yaml
```
![alt text](.\images\k8s-36.png)
**YAML Example**: ConfigMap Definition
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  APP_MODE: "development"
  APP_VERSION: "1.2.3"
```
**Using ConfigMap in a Pod (Environment Variables)**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-env-demo
spec:
  containers:
  - name: demo-container
    image: nginx
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: my-app-config
          key: APP_MODE
```
**Using ConfigMap as a Volume**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-volume-demo
spec:
  containers:
  - name: demo-container
    image: nginx
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-app-config
```
Then the contents of the ConfigMap will appear as files under /etc/config.

**full example with kubectl, pod, and verification steps to test it in your cluster**:
Create a ConfigMap YAML:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  APP_MODE: "production"
  APP_VERSION: "1.0.0"
```
```bash
kubectl apply -f my-configmap.yaml
```
Check it:
```bash
kubectl get configmap my-config -o yaml
```
Use ConfigMap in a Pod as Environment Variables:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: APP_MODE
    - name: APP_VERSION
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: APP_VERSION
```
```bash
kubectl apply -f my-pod.yaml
```
Verify the Environment Variables:

Check if the pod is running:
```bash
kubectl get pods
```
Then exec into the pod and check the environment:
```bash
kubectl exec -it configmap-demo -- sh
```
You should see:
```nginx
production
1.0.0
```
![alt text](.\images\k8s-37.png)
**ConfigMap as Volume Example**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config-volume
data:
  app.properties: |
    APP_MODE=staging
    APP_VERSION=2.1.0
```
Create it:
```bash
kubectl apply -f configmap-volume.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "cat /etc/config/app.properties && sleep 3600"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-config-volume
```
Apply it:
```bash
kubectl apply -f volume-pod.yaml
```
**Verify the Mounted File:**
Wait for the pod to run, then:
```bash
kubectl exec -it configmap-volume-demo -- sh
cat /etc/config/app.properties
```
Expected output:
```ini
APP_MODE=staging
APP_VERSION=2.1.0
```
![alt text](.\images\k8s-38.png)

**Secrets**:In Kubernetes, a Secret is an object used to store sensitive data such as passwords, tokens, SSH keys, or certificates. Storing this data in a Secret is more secure and flexible than putting it directly in a Pod definition or container image.

types:
1. Opaque Secret (Default):
Stores any key-value pair, base64-encoded.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=     # base64 for 'admin'
  password: MWYyZDFlMmU2N2Rm  # base64 for '1f2d1e2e67df'
```
2.kubernetes.io/dockerconfigjson:
Used to pull images from a private Docker registry.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: regcred
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-json>
```
You can create it with:
```bash
kubectl create secret docker-registry regcred \
  --docker-username=<username> --docker-password=<password> \
  --docker-email=<email> --docker-server=<registry>
```
 3.kubernetes.io/tls:
 Stores a TLS certificate and private key.
 ```yaml
 apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```
Or create it via:
```bash
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt --key=path/to/tls.key
```
4.kubernetes.io/basic-auth:
Stores a basic username and password.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth
type: kubernetes.io/basic-auth
stringData:
  username: admin
  password: secret123
```
5.kubernetes.io/ssh-auth:
Stores SSH private key data.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ssh-key-secret
type: kubernetes.io/ssh-auth
data:
  ssh-privatekey: <base64-encoded-private-key>
```
**Example**:
Secret YAML:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
stringData:
  username: admin
  password: secret123
```
```bash
kubectl apply -f my-secret.yaml
```
 Deployment YAML:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secret-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: secret-app
  template:
    metadata:
      labels:
        app: secret-app
    spec:
      containers:
        - name: mycontainer
          image: nginx
          command: ["/bin/sh", "-c"]
          args: ["sleep 3600"]
          env:
            - name: USERNAME
              valueFrom:
                secretKeyRef:
                  name: my-secret
                  key: username
            - name: PASSWORD
              valueFrom:
                secretKeyRef:
                  name: my-secret
                  key: password
          volumeMounts:
            - name: secret-volume
              mountPath: "/etc/secret"
              readOnly: true
      volumes:
        - name: secret-volume
          secret:
            secretName: my-secret
```
```bash
kubectl apply -f secret-deployment.yaml
```
 Test the Secret in Pods:
 Get a Pod name:
 ```bash
 kubectl get pods -l app=secret-app
```
```bash
kubectl exec -it <pod-name> -- /bin/sh
```
Inside the Pod:
```bash
echo $USERNAME
echo $PASSWORD
cat /etc/secret/username
cat /etc/secret/password
```
You should see:
```nginx
admin
secret123
```
![alt text](.\images\k8s-39.png)
**Kubernetes Scheduling and Placement**:
Here are the types of Kubernetes Scheduling and Placement:


**Node Selector**:nodeSelector is the simplest form of node selection constraint in Kubernetes. It allows you to assign a Pod to a specific node (or group of nodes) by using labels. When a Pod is created with a nodeSelector, Kubernetes scheduler places that Pod only on nodes that match the label.

This is useful when you want certain workloads to run only on nodes with special capabilities like GPU, SSD, or a specific zone.

**Example**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-on-ssd
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disktype: ssd
```

This tells Kubernetes to only place this Pod on a node that has the label disktype=ssd.
Testing Commands:

Label a node::
```bash
kubectl get nodes
kubectl label nodes <node-name> disktype=ssd
```
Apply the Pod YAML:
```bash
kubectl apply -f pod.yaml
```
Check where the Pod is scheduled:
```bash
kubectl get pod nginx-on-ssd -o wide
```
To test failure case (if no node has label):
```bash
kubectl label nodes <node-name> disktype-
```
Then reapply the pod. It will stay in Pending state:
```bash
kubectl delete pod nginx-on-ssd
kubectl apply -f nginx-node-selector.yaml
kubectl describe pod nginx-on-ssd
```
You’ll see a scheduling error because no node matches the selector.
![alt text](.\images\k8s-40.png)
![alt text](.\images\k8s-41.png)

**Pod Affinity**:Pod Affinity allows you to schedule a Pod on the same node (or close) to another Pod based on labels. This is helpful for workloads that benefit from being co-located, such as web and cache servers.

**Example**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - web
          topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx
```
**topologyKey: kubernetes.io/hostname** means it tries to place this Pod on the same node as the matched Pod.
Testing Commands for Pod Affinity:
Create the base Pod:
```bash
kubectl run base-pod --image=nginx --labels=app=web
```
Apply the Pod with affinity:
```bash
kubectl apply -f with-pod-affinity.yaml
```
Verify if they are on the same node:
```bash
kubectl get pod -o wide
```
Check the NODE column — both should be on the same node.
![alt text](.\images\k8s-42.png)
**Pod Anti-Affinity**:Pod Anti-Affinity ensures that a Pod is not scheduled on the same node (or topology) as other Pods with a matching label. Use this for high availability (e.g., don't put all replicas on one node).
**Example**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-anti-affinity
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - web
          topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx
```
This ensures it won't schedule on a node that has another Pod with label app=web.
Testing Pod Anti-Affinity:
```bash
kubectl run base-pod --image=nginx --labels=app=web
```
Apply anti-affinity Pod:
```bash 
kubectl apply -f with-pod-anti-affinity.yaml
```
Check scheduling:
```bash
kubectl get pod -o wide
```
It will schedule on a different node, or stay in Pending if no such node exists.
![alt text](.\images\k8s-43.png)

**Node Affinity**:nodeAffinity is a more expressive and flexible way to control which nodes your Pods can be scheduled on, compared to nodeSelector. It uses label selectors and supports:
**requiredDuringSchedulingIgnoredDuringExecution**: Mandatory rules for scheduler.
**preferredDuringSchedulingIgnoredDuringExecution**: Soft preferences, not strict.
**Example**:
**Pod YAML with requiredDuringSchedulingIgnoredDuringExecution**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values:
            - east
  containers:
    - name: nginx
      image: nginx
```
This Pod will only run on nodes labeled zone=east.
Testing Commands:
Label your node:
```bash
kubectl label nodes <your-node-name> zone=east
```
Apply the Pod YAML:
```bash
kubectl apply -f node-affinity-pod.yaml
```
Check which node it was scheduled to:
```bash
kubectl get pod node-affinity-pod -o wide
```
![alt text](.\images\k8s-44.png)
**YAML Example with preferredDuringSchedulingIgnoredDuringExecution**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: preferred-node-affinity
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - east
  containers:
    - name: nginx
      image: nginx
```
This Pod prefers zone=east nodes but can fall back to others if none are available.

Remove the label
```bash
kubectl label nodes <your-node-name> zone-
```
Apply the file
```bash
kubectl apply -f node-affinity-pod.yaml
```
Check it
```bash
kubectl describe pod node-affinity-pod
```
![alt text](.\images\k8s-45.png)

**Taints and Tolerations**:

**Taints** are applied to nodes and say: "Do not schedule Pods here unless they tolerate this condition.

**Tolerations** are applied to Pods and allow them to ignore specific taints on nodes.
This mechanism is used to repel general workloads from special-purpose nodes (like GPU nodes, infra nodes, etc.) unless explicitly allowed.

**Example**:
 Pod YAML Without Toleration 
 ```yaml
 apiVersion: v1
kind: Pod
metadata:
  name: no-toleration-pod
spec:
  containers:
    - name: nginx
      image: nginx
```
Apply a Taint to the Node
```bash
kubectl taint nodes <node-name> key=value:NoSchedule
```
Apply file:
```bash
kubectl apply -f no-toleration-pod.yaml
```
check it:
```bash
kubectl describe pod no-toleration-pod
```
You’ll see the Pod is in Pending due to the taint.
![alt text](.\images\k8s-46.png)
![alt text](.\images\k8s-47.png)
Pod YAML With Toleration (Schedules Successfully)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "env"
    operator: "Equal"
    value: "prod"
    effect: "NoSchedule"
  containers:
    - name: nginx
      image: nginx
```
![alt text](.\images\k8s-48.png)


**HorizontalPodAutoscaler**:
HorizontalPodAutoscaler (HPA) automatically increases or decreases the number of Pods in a Deployment (or StatefulSet, etc.) based on CPU, memory, or custom metrics.

Metrics Server must be running (used by HPA to get CPU/memory metrics).

Install Metrics Server
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Verify:
```bash
kubectl get deployment metrics-server -n kube-system
```
**Example**:

Deploy a Sample App (with CPU requests)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-demo
  template:
    metadata:
      labels:
        app: hpa-demo
    spec:
      containers:
      - name: hpa-demo
        image: k8s.gcr.io/hpa-example
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 200m
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hpa-demo
spec:
  selector:
    app: hpa-demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

```
Apply:
```bash
kubectl apply -f hpa-deployment.yaml
```
Create HPA Object
Create hpa.yaml:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
Apply:
```bash
kubectl apply -f hpa.yaml
```
Generate CPU Load
Run a busybox pod to create load:
```bash
kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
```
Then run:
```bash
while true; do wget -q -O- http://hpa-demo; done
```
Watch HPA Scaling
Check HPA status:
```bash
kubectl get hpa
```
Watch pod scaling:
```bash
kubectl get pods -l app=hpa-demo -w
```
![alt text](.\images\k8s-49.png)
![alt text](.\images\k8s-50.png)

**VPA**:Vertical Pod Autoscaler (VPA) automatically adjusts a pod’s CPU and memory requests/limits based on actual usage over time. Unlike HPA, it doesn’t scale the number of pods, but instead tunes individual pod resources.

It’s ideal for:

  Background or batch jobs

  Services with stable load

  Improving resource efficiency

**Example**:

Install the VPA components

Clone the repository:
```bash
  git clone https://github.com/kubernetes/autoscaler.git
  cd autoscaler/vertical-pod-autoscaler
```
Run the installation script:
```bash
  ./hack/vpa-up.sh
```
Verify the installation:
```bash
  kubectl get pods -n kube-system | grep vpa
```
Deploy a test workload
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vpa-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vpa-demo
  template:
    metadata:
      labels:
        app: vpa-demo
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            memory: "50Mi"
            cpu: "50m"
          limits:
            memory: "100Mi"
            cpu: "100m"
---
apiVersion: v1
kind: Service
metadata:
  name: vpa-demo
spec:
  selector:
    app: vpa-demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```
Create the VPA resource
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-demo
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: vpa-demo
  updatePolicy:
    updateMode: "Auto"
```
You can also use updateMode: "Off" to just view recommendations without applying them.

Let it gather metrics

Wait 5–10 minutes. You can simulate some load if you want using:
```bash
kubectl exec -it $(kubectl get pod -l app=vpa-demo -o jsonpath='{.items[0].metadata.name}') -- sh
```
Inside the pod:
```bash
while true; do echo "ping" > /dev/null; done
```
This will keep the CPU active for testing.

Check VPA recommendations
```bash
kubectl describe vpa vpa-demo
```
Look for this section:
```yaml
Recommendations:
  Container Name: nginx
  Lower Bound:
    Cpu:     25m
    Memory:  30Mi
  Target:
    Cpu:     65m
    Memory:  90Mi
  Upper Bound:
    Cpu:     120m
    Memory:  130Mi
```
If VPA is in Auto mode, it will evict and recreate pods using these new values.

Verify updated pod

After the pod restarts, check the new resource requests:
```bash
kubectl get pod -l app=vpa-demo -o jsonpath='{.items[*].spec.containers[*].resources}'
```
![alt text](.\images\k8s-51.png)
![alt text](.\images\k8s-52.png)

**Cluster Autoscaler**:Cluster Autoscaler is a Kubernetes component that automatically adds or removes EC2 nodes in your EKS cluster based on pending pods that cannot be scheduled due to resource constraints. It adjusts the size of your Auto Scaling Group (ASG) dynamically to match the workload.

If pods are pending (due to insufficient CPU/memory), Cluster Autoscaler increases the node count.

If nodes are underutilized and the pods can be scheduled elsewhere, it terminates those nodes (scale down).


### kubernetes Security:

**Authentication**:Authentication is the process of verifying who you are (user, service account, or API client).

EKS uses IAM to authenticate users.

Update kubeconfig with your IAM identity.
```bash
aws eks update-kubeconfig --region ap-south-1 --name my-cluster
```
Test:
```bash
kubectl get svc
```

**Authorization**:Authorization determines what you can do after you're authenticated. Kubernetes uses RBAC (Role-Based Access Control) by default.

**Example**:

Creating RBAC for Access Control
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
```
You can impersonate the service account and check access:
```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:dev-user
```
Output:
```bash
yes
```
Now try something not allowed:
```bash
kubectl auth can-i delete pods --as=system:serviceaccount:default:dev-user
```
```nginx
no
```
![alt text](.\images\k8s-53.png)
**Network Policies**:A NetworkPolicy in Kubernetes is a way to control the traffic flow between Pods, or between Pods and other network endpoints. It acts like a firewall within the Kubernetes cluster. Network policies are implemented by the network plugin (e.g., Calico, Cilium) — so they only work if the plugin supports them.

By default, all traffic is allowed between pods in Kubernetes. Once any NetworkPolicy is applied to a pod, only the traffic explicitly allowed will be accepted.

By default EKS doesn't support network policy 

Enabling networkpolicy:

1.Click on the cluster name  in the EKS Clusters list. This opens the cluster overview page, where you can view and manage configuration, node groups, networking, workloads, logging, and more.
![alt text](.\images\k8s-54.png)
2.Navigate to the Add-ons tab to manage or view installed EKS add-ons.
![alt text](.\images\k8s-55.png)
3.Scroll down a bit.Click on the Amazon VPC CNI link under the Add-ons tab. 
![alt text](.\images\k8s-56.png)
4.On the vpc-cni add-on detail page, click the Edit button to modify the configuration 
![alt text](.\images\k8s-57.png)
5.In the Edit configuration section of the VPC CNI add-on, the JSON value {"enableNetworkPolicy": "true"} was provided to enable Kubernetes NetworkPolicy support within the EKS cluster.
![alt text](.\images\k8s-58.png)
6.After setting the configuration value to {"enableNetworkPolicy": "true"}.Click on Save to save the changes.
![alt text](.\images\k8s-59.png)
7. Wait for few minutes for VPC CNI to be updated.
![alt text](.\images\k8s-60.png)






**Exampl**:

Restricting Pod Ingress with NetworkPolicy

Create a Namespace  called network:
```bash
kubectl create ns network
```

Create pod-a
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
  namespace: network
  labels:
    app: pod-a
spec:
  containers:
  - name: httpd
    image: httpd
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: pod-a
  namespace: network
spec:
  selector:
    app: pod-a
  ports:
    - port: 80
      targetPort: 80
```
```bash
kubectl apply -f Pod-a.yaml
```
Create Pod-b
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
  namespace: network
  labels:
    app: pod-b
spec:
  containers:
  - name: pod-b
    image: curlimages/curl
    command: ["sleep", "3600"]
```
```bash
kubectl apply -f Pod-b.yaml
```
Create Pod-c
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-c
  namespace: network
  labels:
    app: pod-c
spec:
  containers:
  - name: pod-c
    image: curlimages/curl
    command: ["sleep", "3600"]
```
```bash 
kubectl apply -f pod-c.yaml
```
Create network plicy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-pod-b
  namespace: network
spec:
  podSelector:
    matchLabels:
      app: pod-a
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: pod-b
```
```bash
kubectl apply -f policy.yaml
```
The above setup Demonstrate how to restrict pod-to-pod communication in Kubernetes using NetworkPolicy, allowing only specific pods (like pod-b) to access a target pod (pod-a) while blocking all others (like pod-c) within the same namespace.

From pod-b (should succeed):
```bash
kubectl exec -n network pod-b -- curl pod-a:80
```
Expected: You should see the HTML output from the Apache server running in pod-a.

 From pod-c (should fail)
 ```bash
 kubectl exec -n network pod-c -- curl pod-a:80
```
Expected: The request should fail (timeout or connection refused) because pod-c is not allowed by the NetworkPolicy.
![alt text](.\images\k8s-61.png)
![alt text](.\images\k8s-62.png)












