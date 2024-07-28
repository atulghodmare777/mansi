https://blog.stackademic.com/getting-started-with-karpenter-for-autoscaling-in-eks-9b1992876f4c
https://karpenter.sh/preview/getting-started/migrating-from-cas/

Smooth migration to the karpenter
https://karpenter.sh/docs/getting-started/migrating-from-cas/

If want to paste issues with respect to karpeter on community
GitHub - aws/karpenter-provider-aws: Karpenter is a Kubernetes Node Autoscaler built for flexibility, performance, and simplicity.

Command to check logs:
kubectl logs -f -n kube-system -c controller -l app.kubernetes.io/name=karpenter

kubectl get nodes --no-headers | wc -l








Scheduling | Karpenter
NodeClasses | Karpenter
NodePools | Karpenter
Scheduling | Karpenter
Troubleshooting | Karpenter

spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: role
                operator: In
                values:
                - non-abs-spot

Services for which affinity need to be removed
pl1service-snaqoms


Issues came while installing the karpenter
	• In deployment file dnspolicy was marked as clusterfirst that has been changed to default
	• CPU allocation was 150CPU which was increased to 500CPU
	• The karpenter was not able to schedule pods in the nodes so we have added the label in the nodeclass as follows which will attach to each node it launches, this value is present in the cluster inside the nodepool in k8s labels section:
	spec:
	  template:
	    metadata:
	      labels:
	        role: non-abs-spot
	
	• If want to pass the customised name to the node then add in the nodepool file at the end
	tags:
	    Name: dev-eks-v129
    
 

Suppose want to pass the metrics of karpenter then syntax is as follows
CrashLoopBackOff installing v.33.0 · Issue #5340 · aws/karpenter-provider-aws · GitHub

#Issue with kappenter when trying to monitor it using prometheus:
If want to copy the helm teplate, here we have added the servicemonitor which is required to monitor by the prometheus & grafana, I have just taken this command copied the containt so that I can copy the serviceMonitor file in the original kapenter yaml file and deploy once again:

helm template karpenter oci://public.ecr.aws/karpenter/karpenter --version "${KARPENTER_VERSION}" --namespace "${KARPENTER_NAMESPACE}" --create-namespace \ --set "settings.clusterName=${CLUSTER_NAME}" \ --set "settings.interruptionQueue=${CLUSTER_NAME}" \ --set controller.resources.requests.cpu=1 \ --set controller.resources.requests.memory=1Gi \ --set controller.resources.limits.cpu=1 \ --set controller.resources.limits.memory=1Gi \ --set serviceMonitor.enabled=true  > test.yaml

Aftr this as service monitor will be added but the again we have to add the tag of the matchLabel from the prometheus & grafana yaml files, here in this case while installling the pro & grafana we had following matchlabel :
release: "stable"
Add in label section of the service monitor in the karpenter

helm template karpenter oci://public.ecr.aws/karpenter/karpenter --version "${KARPENTER_VERSION}" --namespace "${KARPENTER_NAMESPACE}" \
    --set "settings.clusterName=${CLUSTER_NAME}" \
    --set "serviceAccount.annotations.eks\.amazonaws\.com/role-arn=arn:${AWS_PARTITION}:iam::${AWS_ACCOUNT_ID}:role/KarpenterControllerRole-${CLUSTER_NAME}" \
    --set controller.resources.requests.cpu=1 \
    --set controller.resources.requests.memory=1Gi \
    --set controller.resources.limits.cpu=1 \
    --set controller.resources.limits.memory=1Gi \
    --set serviceMonitor.enabled=true > karpenter.yaml


https://github.com/aws/karpenter-provider-aws/issues/6355

Implementing karpenter on uat environment:
We have 3 nodegroups present in this cluster, one of them is gpu nodegroup,
Pre-requesite to run gpu node is that nvidia-device-plugin-daemonset should be running on the cluster,
How to check that: kubectl get daemonsets -n kube-system
It should show the output with name nvdp-nvidia-device-plugin, if present then no problem, if not then deploy it by following command: 
kubectl apply -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.11.0/nvidia-device-plugin.yml

Issue came while implementing karpenter on uat with multiple node group:
# The gpu pod had was not detecting the node itself: Inour environment have given provided the toleration in the pod specification and that need to satisfied with the taint on node, hence we have added following taint on the node in the nodepool cofig file as follows:
taints:
      - key: nvidia.com/gpu
        effect: NoSchedule

# I have 3 nodepool with configuration and it was labbeled as the nodegroup on cluster but still the gpu node was coming up but the nodeclaim of the gpu instances was in the false state, actually it should be true:
Ans: GPU nodes require nvidea plugin which was launched as an deamon set in the cluster earlier means while creating the nvidea plugin we have customized it like within the file as node selector with proper label which I have mentioned below means only with the parlicular labelled node the plugin deamonset will launch on that node, hence when cluster autoscaler or from aws console when we launch the node the deamon set pod was coming up in the kube-system namespace but when karpeter was launching the gpu node this pod was not coming up in the kube-system namespace hence the nodeclaim was coming in false state and pod which was requesting the gpu was coming in pending state even after the node was there launched by the karpenter. So we have added the label as follows:
metadata:
      labels:
        role: gpu-ai-nodegroup
        eks-node: gpu
After this we have restarted the karpenter pods and then the pods which were demanding the gpu nodegroup was coming up.
[
We checked the node pool associated with GPU-based nodes and identified that the 'Ready' status was shown as 'False'. Next, we checked if the 'nvidia' plugin was running on the GPU nodes. However, due to the missing label 'eks-node: gpu', the NVIDIA pods were unable to run. We added the label 'eks-node: gpu' to the nodes, and the NVIDIA pods started running without any issues.

As a result, your GPU-based application pods, which were stuck in the 'pending' state, transitioned to the 'running' state without any problems.

Next, we aimed to apply the labels through the Karpenter configuration rather than manually. We updated the Karpenter node pool configuration with the label 'eks-node: gpu', but encountered the error 'all available instance types exceed limits for nodepool: "gpu-ai-nodegroup"'.

We checked the 'gpu-ai-nodegroup' node pool limits, defined as 'cpu: 20', 'memory: 200Gi', and 'nvidia.com/gpu: 2', and found that none of these limits were being exceeded. To ensure Karpenter picked up the new configuration, we recreated the "gpu-ai-nodegroup" node pool and its associated EC2NodeClass, then restarted the Karpenter pods. As a result, new GPU based nodes were launched and pending GPU pods were able to schedule on these nodes without any issues.]

# Even after the pods were empty apart from deamon set like kube-system pods/fluentbit pod still the gpu node was not draining automatically by the karpenter >>
 we have earlier added the consolidation policy as follows;
spec:
  disruption:
    consolidationPolicy: WhenUnderutilized
    expireAfter: 720h

Now to resolve the issue we have added following:
disruption:
    consolidationPolicy: WhenEmpty
    consolidateAfter: 1m
    expireAfter: 720h

# We were getting the SANDBOX error in the pod description for multiple pods,:
 For this error we check the available ips In the subnet where karpenter has launched the nodes, hence the ip should be available, after checking we found that karpenter was launching all the nodes in the c zone subnet hence we have provided the subnet ids in the ec2nodeclass section of all the ec2nodeclass. After this karpenter started launching nodes in multiple subnets which were present in multiple zones.
Earlier:
	 subnetSelectorTerms:
		 - tags:
		        karpenter.sh/discovery: "uat-eks-v128" # replace with your cluster name
Now:
	subnetSelectorTerms:
	    - id: "subnet-0025c0d6d8f0d871a"
	    - id: "subnet-0c7da0348308a15fe"
	    - id: "subnet-04fac5caf8cde92aa"
	    - tags:
	        karpenter.sh/discovery: "uat-eks-v128" # replace with your cluster name
# Core DNS issue: When checked the pods of kube-sysem for coreDNS pod, by that time we found that the coreDNS pod is being schedule to the clusterAutoscaler node not the karpenter node and while implementing the karpenter I have attached the uat-eks-128 tag in the ec2nodeclass by which karpenter was picking the security group but karpenter has picked different security group which has same tag and not the security group which has been earlier attached to the clusterAutoscaler nodes, hence we have tried by adding the new sg to the clusterAutoscaler inbound rule with all traffic, but we had multiple sg with same tag hence next time karpenter might pick up different sg hence I have added directly that clusterAutoscaler sg in the ec2nodeclass like follows:
spec:
  blockDeviceMappings:
    - deviceName: /dev/xvda
      ebs:
        volumeSize: 80Gi
        volumeType: gp3
        iops: 3000
        deleteOnTermination: true
        throughput: 125
  amiFamily: AL2 # Amazon Linux 2
  role: "KarpenterNodeRole-uat-eks-v128" # replace with your cluster name
  subnetSelectorTerms:
    - id: "subnet-04fac5caf8cde92aa"
  securityGroupSelectorTerms:
    - id: "sg-057322ab18e6572b3"
  amiSelectorTerms:
    - id: "ami-0e8396897e597c078"
  tags:
    Name: uat-eks-v128

	







Ai node group 2 2 2
Nonabs 1 5 40
               1 13 40

rwkv-modelservice
 indicstransmodel                

Where it is present [go to ASG and in automatic scaling, scheduled actions
Schedule action for  uat-v128-node-group-nonabs-spot
Name   start time  End time Reccurence                                           Time/zone       Desired Min max
 
scale-up  2024 July 11, 07:40:00 AM +05:30 No entry 40 7 * * 1-6 Asia/kolkata 5              1      40
Scale-down 2024 July 11, 01:00:00 AM +05:30 No entry 0 1 * * * Asia/kolkata  5              1       40

In autoscaling current size desired 15 min1 max 40




uat-v128-node-group-ai-spot

Service with no ai in the ingress but affinity present in this for ai--->uat-vehicleservices-twowheeler.godigit.com




TZ=Asia/Kolkata
H 23 * * *

If you want to make the node unschedulable for any further pods then use following command;
Kubectl cordon nodename

Steps to install migrate from cluster autoscaler to karpenter:

#Set a variable for your cluster name.

KARPENTER_NAMESPACE=kube-system
CLUSTER_NAME=<your cluster name>

#Set other variables from your cluster configuration.

AWS_PARTITION="aws" # if you are not using standard partitions, you may need to configure to aws-cn / aws-us-gov
AWS_REGION="$(aws configure list | grep region | tr -s " " | cut -d" " -f3)"
OIDC_ENDPOINT="$(aws eks describe-cluster --name "${CLUSTER_NAME}" \
    --query "cluster.identity.oidc.issuer" --output text)"
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' \
    --output text)
K8S_VERSION=1.30

ARM_AMI_ID="$(aws ssm get-parameter --name /aws/service/eks/optimized-ami/${K8S_VERSION}/amazon-linux-2-arm64/recommended/image_id --query Parameter.Value --output text)"
AMD_AMI_ID="$(aws ssm get-parameter --name /aws/service/eks/optimized-ami/${K8S_VERSION}/amazon-linux-2/recommended/image_id --query Parameter.Value --output text)"
GPU_AMI_ID="$(aws ssm get-parameter --name /aws/service/eks/optimized-ami/${K8S_VERSION}/amazon-linux-2-gpu/recommended/image_id --query Parameter.Value --output text)"

# Create IAM role:

echo '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}' > node-trust-policy.json

aws iam create-role --role-name "KarpenterNodeRole-${CLUSTER_NAME}" \
    --assume-role-policy-document file://node-trust-policy.json

#Now attach the required policies to the role

aws iam attach-role-policy --role-name "KarpenterNodeRole-${CLUSTER_NAME}" \
    --policy-arn "arn:${AWS_PARTITION}:iam::aws:policy/AmazonEKSWorkerNodePolicy"

aws iam attach-role-policy --role-name "KarpenterNodeRole-${CLUSTER_NAME}" \
    --policy-arn "arn:${AWS_PARTITION}:iam::aws:policy/AmazonEKS_CNI_Policy"

aws iam attach-role-policy --role-name "KarpenterNodeRole-${CLUSTER_NAME}" \
    --policy-arn "arn:${AWS_PARTITION}:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"

aws iam attach-role-policy --role-name "KarpenterNodeRole-${CLUSTER_NAME}" \
    --policy-arn "arn:${AWS_PARTITION}:iam::aws:policy/AmazonSSMManagedInstanceCore"


#Now we need to create an IAM role that the Karpenter controller will use to provision new instances. The controller will be using IAM Roles for Service Accounts (IRSA) which requires an OIDC endpoint.

cat << EOF > controller-trust-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:${AWS_PARTITION}:iam::${AWS_ACCOUNT_ID}:oidc-provider/${OIDC_ENDPOINT#*//}"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "${OIDC_ENDPOINT#*//}:aud": "sts.amazonaws.com",
                    "${OIDC_ENDPOINT#*//}:sub": "system:serviceaccount:${KARPENTER_NAMESPACE}:karpenter"
                }
            }
        }
    ]
}
EOF

aws iam create-role --role-name "KarpenterControllerRole-${CLUSTER_NAME}" \
    --assume-role-policy-document file://controller-trust-policy.json

cat << EOF > controller-policy.json
{
    "Statement": [
        {
            "Action": [
                "ssm:GetParameter",
                "ec2:DescribeImages",
                "ec2:RunInstances",
                "ec2:DescribeSubnets",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeLaunchTemplates",
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceTypes",
                "ec2:DescribeInstanceTypeOfferings",
                "ec2:DescribeAvailabilityZones",
                "ec2:DeleteLaunchTemplate",
                "ec2:CreateTags",
                "ec2:CreateLaunchTemplate",
                "ec2:CreateFleet",
                "ec2:DescribeSpotPriceHistory",
                "pricing:GetProducts"
            ],
            "Effect": "Allow",
            "Resource": "*",
            "Sid": "Karpenter"
        },
        {
            "Action": "ec2:TerminateInstances",
            "Condition": {
                "StringLike": {
                    "ec2:ResourceTag/karpenter.sh/nodepool": "*"
                }
            },
            "Effect": "Allow",
            "Resource": "*",
            "Sid": "ConditionalEC2Termination"
        },
        {
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:${AWS_PARTITION}:iam::${AWS_ACCOUNT_ID}:role/KarpenterNodeRole-${CLUSTER_NAME}",
            "Sid": "PassNodeIAMRole"
        },
        {
            "Effect": "Allow",
            "Action": "eks:DescribeCluster",
            "Resource": "arn:${AWS_PARTITION}:eks:${AWS_REGION}:${AWS_ACCOUNT_ID}:cluster/${CLUSTER_NAME}",
            "Sid": "EKSClusterEndpointLookup"
        },
        {
            "Sid": "AllowScopedInstanceProfileCreationActions",
            "Effect": "Allow",
            "Resource": "*",
            "Action": [
            "iam:CreateInstanceProfile"
            ],
            "Condition": {
            "StringEquals": {
                "aws:RequestTag/kubernetes.io/cluster/${CLUSTER_NAME}": "owned",
                "aws:RequestTag/topology.kubernetes.io/region": "${AWS_REGION}"
            },
            "StringLike": {
                "aws:RequestTag/karpenter.k8s.aws/ec2nodeclass": "*"
            }
            }
        },
        {
            "Sid": "AllowScopedInstanceProfileTagActions",
            "Effect": "Allow",
            "Resource": "*",
            "Action": [
            "iam:TagInstanceProfile"
            ],
            "Condition": {
            "StringEquals": {
                "aws:ResourceTag/kubernetes.io/cluster/${CLUSTER_NAME}": "owned",
                "aws:ResourceTag/topology.kubernetes.io/region": "${AWS_REGION}",
                "aws:RequestTag/kubernetes.io/cluster/${CLUSTER_NAME}": "owned",
                "aws:RequestTag/topology.kubernetes.io/region": "${AWS_REGION}"
            },
            "StringLike": {
                "aws:ResourceTag/karpenter.k8s.aws/ec2nodeclass": "*",
                "aws:RequestTag/karpenter.k8s.aws/ec2nodeclass": "*"
            }
            }
        },
        {
            "Sid": "AllowScopedInstanceProfileActions",
            "Effect": "Allow",
            "Resource": "*",
            "Action": [
            "iam:AddRoleToInstanceProfile",
            "iam:RemoveRoleFromInstanceProfile",
            "iam:DeleteInstanceProfile"
            ],
            "Condition": {
            "StringEquals": {
                "aws:ResourceTag/kubernetes.io/cluster/${CLUSTER_NAME}": "owned",
                "aws:ResourceTag/topology.kubernetes.io/region": "${AWS_REGION}"
            },
            "StringLike": {
                "aws:ResourceTag/karpenter.k8s.aws/ec2nodeclass": "*"
            }
            }
        },
        {
            "Sid": "AllowInstanceProfileReadActions",
            "Effect": "Allow",
            "Resource": "*",
            "Action": "iam:GetInstanceProfile"
        }
    ],
    "Version": "2012-10-17"
}
EOF

aws iam put-role-policy --role-name "KarpenterControllerRole-${CLUSTER_NAME}" \
    --policy-name "KarpenterControllerPolicy-${CLUSTER_NAME}" \
    --policy-document file://controller-policy.json


# Add tags to the subnet and security group:

for NODEGROUP in $(aws eks list-nodegroups --cluster-name "${CLUSTER_NAME}" --query 'nodegroups' --output text); do
    aws ec2 create-tags \
        --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" \
        --resources $(aws eks describe-nodegroup --cluster-name "${CLUSTER_NAME}" \
        --nodegroup-name "${NODEGROUP}" --query 'nodegroup.subnets' --output text )
done

# Add tags to our security groups. This command only tags the security groups for the first nodegroup in the cluster. If you have multiple nodegroups or multiple security groups you will need to decide which one Karpenter should use.

NODEGROUP=$(aws eks list-nodegroups --cluster-name "${CLUSTER_NAME}" \
    --query 'nodegroups[0]' --output text)

LAUNCH_TEMPLATE=$(aws eks describe-nodegroup --cluster-name "${CLUSTER_NAME}" \
    --nodegroup-name "${NODEGROUP}" --query 'nodegroup.launchTemplate.{id:id,version:version}' \
    --output text | tr -s "\t" ",")

# If your EKS setup is configured to use only Cluster security group, then please execute -
SECURITY_GROUPS=$(aws eks describe-cluster \
    --name "${CLUSTER_NAME}" --query "cluster.resourcesVpcConfig.clusterSecurityGroupId" --output text)

# If your setup uses the security groups in the Launch template of a managed node group, then :[Do not execute this specific command for our current cluster setup]

SECURITY_GROUPS="$(aws ec2 describe-launch-template-versions \
    --launch-template-id "${LAUNCH_TEMPLATE%,*}" --versions "${LAUNCH_TEMPLATE#*,}" \
    --query 'LaunchTemplateVersions[0].LaunchTemplateData.[NetworkInterfaces[0].Groups||SecurityGroupIds]' \
    --output text)"

aws ec2 create-tags \
    --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" \
    --resources "${SECURITY_GROUPS}"

# Update aws-auth config-map:

kubectl edit configmap aws-auth -n kube-system

You will need to add a section to the mapRoles that looks something like this. Replace the ${AWS_PARTITION} variable with the account partition, ${AWS_ACCOUNT_ID} variable with your account ID, and ${CLUSTER_NAME} variable with the cluster name, but do not replace the {{EC2PrivateDNSName}}.

- groups:
  - system:bootstrappers
  - system:nodes
  ## If you intend to run Windows workloads, the kube-proxy group should be specified.
  # For more information, see https://github.com/aws/karpenter/issues/5099.
  # - eks:kube-proxy-windows
  rolearn: arn:${AWS_PARTITION}:iam::${AWS_ACCOUNT_ID}:role/KarpenterNodeRole-${CLUSTER_NAME}
  username: system:node:{{EC2PrivateDNSName}}


The full aws-auth configmap should have two groups. One for your Karpenter node role and one for your existing node group. [Be careful while editing the aws-auth because simple editing mistake can make clustur configuration go down, take backup of the existing auth file]

# Deploy karpenter:

export KARPENTER_VERSION="0.37.0"

We can now generate a full Karpenter deployment yaml from the Helm chart.

helm template karpenter oci://public.ecr.aws/karpenter/karpenter --version "${KARPENTER_VERSION}" --namespace "${KARPENTER_NAMESPACE}" \
    --set "settings.clusterName=${CLUSTER_NAME}" \
    --set "serviceAccount.annotations.eks\.amazonaws\.com/role-arn=arn:${AWS_PARTITION}:iam::${AWS_ACCOUNT_ID}:role/KarpenterControllerRole-${CLUSTER_NAME}" \
    --set controller.resources.requests.cpu=1 \
    --set controller.resources.requests.memory=1Gi \
    --set controller.resources.limits.cpu=1 \
    --set controller.resources.limits.memory=1Gi > karpenter.yaml

Modify the following lines in the karpenter.yaml file.

  Set node Affinity,
Edit the karpenter.yaml file and find the karpenter deployment affinity rules. Modify the affinity so karpenter will run on one of the existing node group nodes.
The rules should look something like this. Modify the value to match your $NODEGROUP, one node group per line.

affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: karpenter.sh/nodepool
          operator: DoesNotExist
        - key: eks.amazonaws.com/nodegroup
          operator: In
          values:
          - ${NODEGROUP}
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - topologyKey: "kubernetes.io/hostname"

Now that our deployment is ready we can create the karpenter namespace, create the NodePool CRD, and then deploy the rest of the karpenter resources.

kubectl create namespace "${KARPENTER_NAMESPACE}" || true
kubectl create -f \
    "https://raw.githubusercontent.com/aws/karpenter-provider-aws/v${KARPENTER_VERSION}/pkg/apis/crds/karpenter.sh_nodepools.yaml"
kubectl create -f \
    "https://raw.githubusercontent.com/aws/karpenter-provider-aws/v${KARPENTER_VERSION}/pkg/apis/crds/karpenter.k8s.aws_ec2nodeclasses.yaml"
kubectl create -f \
    "https://raw.githubusercontent.com/aws/karpenter-provider-aws/v${KARPENTER_VERSION}/pkg/apis/crds/karpenter.sh_nodeclaims.yaml"
kubectl apply -f karpenter.yaml

# Create a default nodepool
Vi nodepool.yaml > 
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
        - key: kubernetes.io/os
          operator: In
          values: ["linux"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot"]
        - key: karpenter.k8s.aws/instance-category
          operator: In
          values: ["c", "m", "r"]
        - key: karpenter.k8s.aws/instance-generation
          operator: Gt
          values: ["2"]
      nodeClassRef:
        apiVersion: karpenter.k8s.aws/v1beta1
        kind: EC2NodeClass
        name: default
  limits:
    cpu: 1000
  disruption:
    consolidationPolicy: WhenUnderutilized
    expireAfter: 720h # 30 * 24h = 720h
---
apiVersion: karpenter.k8s.aws/v1beta1
kind: EC2NodeClass
metadata:
  name: default
spec:
  amiFamily: AL2 # Amazon Linux 2
  role: "KarpenterNodeRole-${CLUSTER_NAME}" # replace with your cluster name
  subnetSelectorTerms:
    - tags:
        karpenter.sh/discovery: "${CLUSTER_NAME}" # replace with your cluster name
  securityGroupSelectorTerms:
    - tags:
        karpenter.sh/discovery: "${CLUSTER_NAME}" # replace with your cluster name
  amiSelectorTerms:
    - id: "${ARM_AMI_ID}"
    - id: "${AMD_AMI_ID}"
#   - id: "${GPU_AMI_ID}" # <- GPU Optimized AMD AMI 
#   - name: "amazon-eks-node-${K8S_VERSION}-*" # <- automatically upgrade when a new AL2 EKS Optimized AMI is released. This is unsafe for production workloads. Validate AMIs in lower environments before deploying them to production.
EOF



git clone git@bitbucket.org:godigit/scripts.git
 cd karpenter
 cd dev-general and run kubectl apply -f nodepool-nodepool.yaml [for single node architecture]
OR
 cd uat-general and run kubectl apply -f nodepool-karpenter.yaml [for multi node architecture]

{ modify the karpenter.yaml and nodepool.yaml according to the requirement}

# Remove cluster autoscaler:

kubectl scale deploy/cluster-autoscaler -n kube-system --replicas=0

# Update the nodes size:
Make nodes count to 2 from aws console 

# Verify the karpenter logs:

kubectl logs -f -n kube-system -c controller -l app.kubernetes.io/name=karpenter

Non abs---->12 1 40
 ai-spot:----> 40 1 40

 - tags:
        karpenter.sh/discovery: "uat-eks-v128" # replace with your cluster name

apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0


MutatingWebhookConfiguration

apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-service
  namespace: example-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-service
  template:
    metadata:
      labels:
        app: example-service
        dev: jjones
    spec:
      containers:
      - name: example-container
        image: your-image
        ports:
        - containerPort: 80
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: "topology.kubernetes.io/zone"
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels:
              app: example-service


affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: role
                  operator: In
                  values:
                  - {{ .Values.nodeAffinity }}
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: "topology.kubernetes.io/zone"
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels:
              app: {{ .Values.app }}


nodeAffinity: ai-spot


nodeAffinity: non-abs-spot

Add zone in the karpenter file as well
Add the topology spread: DoNotSchedule for uat
Then check

spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: topology.kubernetes.io/zone
                operator: In
                values:
                - antarctica-east1
            nodeSelectorTerms:
            - matchExpressions:
              - key: role
                operator: In
                values:
                - non-abs-spot

Gpu 2 1 1
Ai 41 1 41
Nonabs 11 1 40


UAT Automatic scaling backup: inside schedule actions
For GPU


For AI spot

For non-abs Spot



Production clusters general a/c: 
Prod-eks-129: 1 nodegroup
Prod-datalake-cluster:  1 nodegroup
Prod-bot-eks: 3 nodegroup
Prod-ai-eks-126: 3 nodegroup(1 with gpu)

Production cluster infoworks:
Prod-common-eks-v30: 1 nodegroup
Prod-datalake-common-eks-v129

