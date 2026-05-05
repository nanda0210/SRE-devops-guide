# SRE / DevOps / CI/CD / AWS / EKS Interview Question Bank

Prepared for: `/Users/nandamac/Ankprojects/NewBackupAll/Notes-All`

Scope: SRE, DevOps, CI/CD, AWS multi-account networking, VPC, CIDR, Internet Gateway, Security Groups, IAM, KMS, EKS, CoreDNS, External Secrets, Karpenter, EBS, EFS, Fargate, Pod Identity, Argo CD, GitHub PR workflow, CloudBees/Jenkins pipelines, Helm deployments to EKS.

Reference anchors used while building this file:
- AWS VPC, subnet sharing, Transit Gateway, EKS, Pod Identity, Fargate, KMS, IAM, Karpenter best-practice docs.
- Argo CD official docs for GitOps, Helm, declarative applications, tracking strategies.
- GitHub official docs for pull requests and GitHub Actions.
- CloudBees official docs for CloudBees CI on EKS and Kubernetes.

---

## How to Use This File

For each question, practice answering in this order:

1. **One-line answer** — direct answer first.
2. **Architecture answer** — explain how it works in a real enterprise platform.
3. **Workflow** — describe the exact steps you would take.
4. **Failure mode** — mention what can break and how you troubleshoot.
5. **STAR story** — connect it to a real project using Situation, Task, Action, Result.

---

# 1. SRE Fundamentals

## Q1. What is the difference between SRE and DevOps?

**Answer:** DevOps is a culture and practice model for improving collaboration between development and operations. SRE is a concrete engineering discipline that applies software engineering to operations problems, usually with SLIs, SLOs, error budgets, automation, incident response, and reliability engineering.

**Explanation:** DevOps focuses on culture, CI/CD, collaboration, and shared ownership. SRE focuses on measurable reliability and reducing toil through automation.

**Workflow:**
1. Define service ownership.
2. Define SLIs such as availability, latency, error rate, saturation.
3. Set SLO targets.
4. Track error budget burn.
5. Automate repetitive operational work.
6. Run postmortems after incidents.

**STAR example:**
- **S:** A platform team had repeated manual deployment and incident recovery steps.
- **T:** Improve reliability and reduce manual toil.
- **A:** Added CI/CD gates, health checks, observability dashboards, rollback steps, and runbooks.
- **R:** Reduced incident response time and made releases safer.

---

## Q2. What is toil in SRE?

**Answer:** Toil is manual, repetitive, automatable operational work that scales linearly with service growth.

**Explanation:** Restarting pods manually, rotating secrets manually, approving repetitive deployments, and checking logs by hand are examples of toil.

**Workflow to reduce toil:**
1. Identify repeated manual tasks.
2. Measure frequency and time spent.
3. Automate using scripts, pipelines, controllers, or GitOps.
4. Add monitoring and alerts.
5. Document the new workflow.

---

## Q3. What are SLIs, SLOs, and SLAs?

**Answer:**
- **SLI:** A metric that measures service behavior.
- **SLO:** A reliability target based on SLIs.
- **SLA:** A formal contract with business or customer consequences.

**Example:**
- SLI: HTTP success rate.
- SLO: 99.9% successful requests over 30 days.
- SLA: If availability drops below 99.9%, customer receives service credit.

**Workflow:**
1. Pick user-facing indicators.
2. Define measurement source.
3. Set realistic target.
4. Alert on fast and slow burn rates.
5. Review reliability during releases.

---

## Q4. What is an error budget?

**Answer:** An error budget is the allowed amount of unreliability within an SLO window.

**Explanation:** If the SLO is 99.9%, the service can be unavailable for 0.1% of the measurement period. If the team burns the budget too quickly, they should slow down risky releases and focus on reliability.

**Workflow:**
1. Define SLO.
2. Calculate allowed failure percentage.
3. Track burn rate.
4. Gate releases when burn rate is too high.
5. Use postmortems to reduce future burn.

---

## Q5. How do you handle an incident as an SRE?

**Answer:** Stabilize first, communicate clearly, mitigate impact, identify root cause, recover safely, and document lessons learned.

**Workflow:**
1. Acknowledge alert.
2. Declare severity.
3. Assign incident commander.
4. Check dashboards, logs, recent deployments, infra changes.
5. Roll back or mitigate.
6. Communicate status.
7. Confirm recovery.
8. Write postmortem.
9. Create action items.

**Common follow-up:** “What if you do not know the root cause?”

**Answer:** Focus on mitigation first. Root cause can be determined after the service is stable.

---

# 2. AWS Multi-Account Architecture

## Q6. Why do companies use multiple AWS accounts?

**Answer:** Multiple AWS accounts improve security isolation, billing separation, blast-radius reduction, environment separation, and governance.

**Explanation:** A common enterprise setup separates network, shared services, security, logging, dev, stage, and production application accounts.

**Workflow:**
1. Create accounts through AWS Organizations.
2. Apply SCPs.
3. Centralize networking in a network account.
4. Centralize logs/security tooling.
5. Deploy application workloads in app accounts.
6. Use cross-account IAM roles for automation.

---

## Q7. What is a network account?

**Answer:** A network account is a centralized AWS account used to own and manage shared networking resources such as Transit Gateway, shared VPCs, shared subnets, centralized ingress/egress, DNS, and firewalling.

**Architecture answer:** In many enterprise platforms, the network account owns ALBs, shared subnets, Route 53 private hosted zones, Transit Gateway, or inspection VPCs. Application accounts run workloads and connect to this network layer.

**Workflow:**
1. Network team creates VPC/subnets/TGW/shared services.
2. Network resources are shared through AWS RAM.
3. Application account attaches VPC or uses shared subnet.
4. Routes and security groups are configured.
5. App workloads deploy behind internal or external load balancers.

---

## Q8. What is an application AWS account?

**Answer:** An application account is an AWS account where application-specific workloads run, such as EKS clusters, application pods, EC2 instances, databases, ECR repos, and app-specific IAM roles.

**Best practice:** Keep application permissions separate from network/security/logging accounts.

---

## Q9. How would you design connectivity between a network account and application accounts?

**Answer:** Use AWS Transit Gateway, VPC sharing, VPC peering, PrivateLink, or shared subnets depending on isolation and routing needs.

**Recommended enterprise pattern:** Use Transit Gateway in the network account and share it with application accounts using AWS RAM. For shared subnet models, the network account owns the VPC and shares subnets with app accounts.

**Workflow:**
1. Create TGW in network account.
2. Share TGW with app accounts using AWS RAM.
3. Attach app VPCs to TGW.
4. Configure TGW route tables.
5. Add VPC route table entries.
6. Validate connectivity with controlled security group and NACL rules.

---

# 3. AWS VPC, CIDR, Routing, Internet Gateway

## Q10. What is a VPC?

**Answer:** A VPC is a logically isolated virtual network in AWS where resources like EC2, EKS worker nodes, RDS, and load balancers run.

**Explanation:** You define IP ranges, subnets, route tables, gateways, endpoints, and security controls.

---

## Q11. How do you choose a CIDR block for a VPC?

**Answer:** Choose a CIDR block large enough for current and future subnets while avoiding overlap with other VPCs, on-prem networks, and partner networks.

**Example:**
- `/16` gives 65,536 IPs.
- `/20` gives 4,096 IPs.
- `/24` gives 256 IPs.

**Workflow:**
1. Gather existing network ranges.
2. Avoid overlap.
3. Reserve IP space for future environments.
4. Split by AZ and subnet type.
5. Account for EKS pod IP consumption.

**Interview warning:** EKS can consume many IPs because pods often receive VPC IPs through the AWS VPC CNI.

---

## Q12. What is the difference between public and private subnets?

**Answer:** A public subnet has a route to an Internet Gateway. A private subnet does not have a direct route to an Internet Gateway and usually reaches the internet through a NAT Gateway.

**Workflow:**
1. Public subnet route table: `0.0.0.0/0 -> Internet Gateway`.
2. Private subnet route table: `0.0.0.0/0 -> NAT Gateway`.
3. Internal-only private subnets may have no internet route.

---

## Q13. What is an Internet Gateway?

**Answer:** An Internet Gateway allows resources in a VPC to communicate with the public internet when route tables, public IPs, and security rules allow it.

**Common mistake:** Attaching an Internet Gateway alone does not make a subnet public. The route table must route `0.0.0.0/0` to the Internet Gateway.

---

## Q14. What is a NAT Gateway?

**Answer:** A NAT Gateway lets resources in private subnets initiate outbound internet traffic without allowing inbound internet traffic directly to those resources.

**Example:** Private EKS worker nodes use NAT Gateway to pull images or call external APIs if VPC endpoints are not configured.

---

## Q15. What are route tables?

**Answer:** Route tables decide where network traffic is sent based on destination CIDR ranges.

**Example:**
- Local VPC route: `10.0.0.0/16 -> local`
- Internet route: `0.0.0.0/0 -> igw-xxx`
- Private outbound: `0.0.0.0/0 -> nat-xxx`
- Multi-account: `10.20.0.0/16 -> tgw-xxx`

---

## Q16. What is the difference between a Security Group and a NACL?

**Answer:** Security Groups are stateful firewalls attached to resources. NACLs are stateless firewalls applied at the subnet level.

**Explanation:** If a Security Group allows inbound traffic, return traffic is automatically allowed. With NACLs, inbound and outbound rules must both allow traffic.

**Best practice:** Use Security Groups for workload-level access control. Use NACLs for coarse subnet-level guardrails.

---

## Q17. How would you troubleshoot no internet access from an EC2 instance in a public subnet?

**Answer:** Check public IP, route table, Internet Gateway, Security Group, NACL, OS firewall, and DNS.

**Workflow:**
1. Confirm subnet route table has `0.0.0.0/0 -> IGW`.
2. Confirm VPC has attached IGW.
3. Confirm instance has public IPv4 or Elastic IP.
4. Confirm SG allows needed outbound traffic.
5. Confirm NACL allows inbound/outbound ephemeral ports.
6. Test DNS resolution.
7. Test with `curl`, `ping`, `traceroute`, or VPC Reachability Analyzer.

---

# 4. IAM, Roles, Policies, KMS

## Q18. What is IAM?

**Answer:** IAM controls authentication and authorization for AWS resources using users, groups, roles, policies, and permission boundaries.

**Best practice:** Use roles and temporary credentials instead of long-lived access keys.

---

## Q19. What is the difference between an IAM role and an IAM policy?

**Answer:** A role is an identity that can be assumed. A policy is a JSON permission document attached to a role, user, group, or resource.

**Example:** An EKS application pod assumes an IAM role through EKS Pod Identity. The attached policy allows access to only one S3 bucket or one Secrets Manager secret.

---

## Q20. What is a trust policy?

**Answer:** A trust policy defines who or what is allowed to assume an IAM role.

**Example:** For EKS Pod Identity, the trust policy allows the EKS service principal to assume the role and can use conditions to restrict namespace, service account, or cluster.

---

## Q21. What is least privilege?

**Answer:** Least privilege means granting only the permissions required to perform a task, nothing more.

**Workflow:**
1. Start with no permissions.
2. Add only required actions.
3. Scope resources by ARN.
4. Add conditions.
5. Test access.
6. Use Access Analyzer or CloudTrail to refine.

---

## Q22. What is KMS used for?

**Answer:** AWS KMS is used to create and manage encryption keys for encrypting data in services such as EBS, EFS, S3, Secrets Manager, RDS, and EKS secrets encryption.

---

## Q23. What is the difference between a KMS key policy and an IAM policy?

**Answer:** A KMS key policy is the primary resource-based policy on a KMS key. IAM policies can grant permissions only if the key policy allows the IAM principal or account to use IAM authorization.

**Interview warning:** For KMS, IAM permission alone may not be enough. The key policy must also allow access.

---

## Q24. How would you troubleshoot KMS `AccessDeniedException`?

**Answer:** Check the IAM policy, KMS key policy, grants, key ARN, encryption context, region, and cross-account permissions.

**Workflow:**
1. Confirm principal identity with `aws sts get-caller-identity`.
2. Confirm the policy allows the KMS action.
3. Confirm the key policy trusts the principal/account.
4. Check resource ARN uses key ARN, not alias ARN, for key permissions.
5. Check conditions such as encryption context.
6. Check CloudTrail event details.

---

# 5. EKS Cluster Architecture

## Q25. What is Amazon EKS?

**Answer:** Amazon EKS is AWS managed Kubernetes. AWS manages the Kubernetes control plane, while customers manage worker compute through managed node groups, self-managed nodes, Fargate, or Karpenter-provisioned nodes.

---

## Q26. What are the main components of an EKS cluster?

**Answer:**
1. Kubernetes control plane.
2. Worker nodes or Fargate profiles.
3. VPC networking and subnets.
4. AWS VPC CNI.
5. CoreDNS.
6. kube-proxy.
7. IAM access integration.
8. Storage drivers such as EBS CSI and EFS CSI.
9. Add-ons like External Secrets, Karpenter, metrics-server, ingress controller, and Argo CD.

---

## Q27. What are EKS managed node groups?

**Answer:** Managed node groups are AWS-managed groups of EC2 worker nodes for EKS. AWS helps manage lifecycle operations such as provisioning, draining, and updating nodes.

**Use case:** Stable baseline workloads, system workloads, and teams that want EC2 node control without fully self-managing the node lifecycle.

---

## Q28. What is Fargate in EKS?

**Answer:** Fargate runs Kubernetes pods without managing EC2 worker nodes. You define Fargate profiles that match namespaces and labels.

**Best use cases:** Small services, isolated workloads, jobs, or workloads where node management is not desired.

**Limitations to mention:** Some DaemonSets and privileged workloads are not suitable for Fargate.

---

## Q29. What is the EKS pod execution role?

**Answer:** The pod execution role is used by EKS Fargate infrastructure components to pull images and register Fargate pods with the cluster.

**Important distinction:** Pod execution role is not the same as the application IAM role used by the pod to access AWS services.

---

## Q30. What is CoreDNS?

**Answer:** CoreDNS provides DNS resolution inside the Kubernetes cluster. It lets pods resolve Kubernetes service names and external domains.

**Troubleshooting workflow:**
1. Check CoreDNS pods: `kubectl get pods -n kube-system -l k8s-app=kube-dns`.
2. Check logs.
3. Check service: `kubectl get svc -n kube-system kube-dns`.
4. Test from a pod: `nslookup kubernetes.default`.
5. Check NetworkPolicies, security groups, and node DNS config.

---

## Q31. What are “core groups” in an EKS/platform context?

**Answer:** In platform interviews, “core groups” often means the core platform components or teams responsible for shared services such as networking, IAM, DNS, observability, ingress, secrets, storage, and cluster add-ons.

**Architecture answer:** A platform team usually separates application namespaces from system namespaces such as `kube-system`, `argocd`, `external-secrets`, `karpenter`, `ingress`, and `monitoring`.

---

## Q32. What are EKS add-ons?

**Answer:** EKS add-ons are operational components required or commonly used by EKS, such as VPC CNI, CoreDNS, kube-proxy, EBS CSI driver, and Pod Identity agent.

**Workflow:**
1. Select add-on version compatible with cluster version.
2. Install with AWS console, CLI, Terraform, or Helm.
3. Attach required IAM permissions.
4. Monitor rollout.
5. Upgrade add-ons before or during cluster upgrades.

---

# 6. EKS Networking and Security

## Q33. How does pod networking work in EKS?

**Answer:** By default, EKS uses the AWS VPC CNI, which assigns VPC IP addresses to pods. This lets pods communicate directly with VPC resources but increases IP consumption.

**Workflow:**
1. Worker node joins subnet.
2. VPC CNI attaches ENIs and secondary IPs.
3. Pods receive IPs from the subnet CIDR.
4. Security and routing follow VPC rules.

---

## Q34. Why can EKS run out of IP addresses?

**Answer:** Pods consume IP addresses from the subnet when using the AWS VPC CNI. Small subnets can run out of available IPs as nodes and pods scale.

**Solutions:**
1. Use larger subnets.
2. Use prefix delegation.
3. Add secondary CIDR blocks.
4. Use separate subnets for nodes.
5. Monitor available IPs.

---

## Q35. How do you expose an application running in EKS?

**Answer:** Use Kubernetes Service types, Ingress, AWS Load Balancer Controller, ALB, NLB, or API Gateway depending on traffic requirements.

**Workflow:**
1. Deploy app as Deployment.
2. Create Service.
3. Create Ingress or LoadBalancer service.
4. AWS Load Balancer Controller creates ALB/NLB.
5. Configure DNS.
6. Add TLS certificate.
7. Restrict traffic with security groups and ingress rules.

---

## Q36. What is the difference between ALB and NLB?

**Answer:** ALB is Layer 7 and supports HTTP/HTTPS routing, host/path rules, and TLS termination. NLB is Layer 4 and supports high-performance TCP/UDP/TLS traffic.

**Use cases:**
- ALB: web apps, REST APIs, path-based routing.
- NLB: TCP services, static IP requirements, high-throughput low-latency traffic.

---

# 7. EKS IAM: IRSA and Pod Identity

## Q37. What is IRSA?

**Answer:** IAM Roles for Service Accounts lets Kubernetes service accounts assume IAM roles using OIDC federation.

**Workflow:**
1. Enable OIDC provider for cluster.
2. Create IAM role with trust policy for service account.
3. Attach IAM policy.
4. Annotate Kubernetes service account.
5. Deploy pod using that service account.

---

## Q38. What is EKS Pod Identity?

**Answer:** EKS Pod Identity lets pods access AWS services by associating an IAM role with a Kubernetes service account. It simplifies role association and credential delivery compared with older patterns.

**Workflow:**
1. Install/enable EKS Pod Identity Agent.
2. Create IAM role and trust policy for Pod Identity.
3. Attach least-privilege IAM policy.
4. Create pod identity association for namespace and service account.
5. Deploy pod using that service account.
6. Test AWS access from pod.

---

## Q39. What is the difference between node IAM role and pod IAM role?

**Answer:** Node IAM role gives permissions to the EC2 node. Pod IAM role gives permissions to a specific workload through service account mapping.

**Best practice:** Do not give broad AWS access to the node role. Use pod-level identity for application permissions.

---

# 8. EBS, EFS, Storage

## Q40. What is EBS used for in EKS?

**Answer:** EBS provides block storage for Kubernetes PersistentVolumes, usually mounted to a single pod/node at a time depending on access mode.

**Use case:** Databases, single-writer stateful workloads, persistent disk volumes.

**Workflow:**
1. Install EBS CSI driver.
2. Configure IAM permissions.
3. Create StorageClass.
4. Create PVC.
5. Pod mounts volume.

---

## Q41. What is EFS used for in EKS?

**Answer:** EFS provides shared file storage that can be mounted by multiple pods across nodes and Availability Zones.

**Use case:** Shared content, files, multi-reader/multi-writer workloads, CloudBees/Jenkins shared storage patterns.

**Workflow:**
1. Create EFS filesystem.
2. Create mount targets in subnets.
3. Configure security group NFS port 2049.
4. Install EFS CSI driver.
5. Create StorageClass/PV/PVC.
6. Mount from pod.

---

## Q42. EBS vs EFS?

**Answer:** EBS is block storage and typically single-node attached. EFS is network file storage and supports shared access across multiple nodes.

**Interview answer:** Use EBS for performance-sensitive single-writer workloads. Use EFS for shared filesystem workloads.

---

# 9. External Secrets and Secrets Management

## Q43. What is External Secrets Operator?

**Answer:** External Secrets Operator syncs secrets from external secret stores such as AWS Secrets Manager or Parameter Store into Kubernetes Secrets.

**Workflow:**
1. Store secret in AWS Secrets Manager.
2. Configure IAM role for External Secrets.
3. Create ClusterSecretStore or SecretStore.
4. Create ExternalSecret resource.
5. Operator syncs Kubernetes Secret.
6. App mounts or reads the Kubernetes Secret.

---

## Q44. What is the difference between SecretStore and ClusterSecretStore?

**Answer:** SecretStore is namespace-scoped. ClusterSecretStore is cluster-scoped and can be referenced by multiple namespaces.

**Best practice:** Use namespace-scoped stores when teams require isolation. Use ClusterSecretStore for centrally managed platform secret access with strict IAM and RBAC controls.

---

## Q45. If a secret changes in AWS Secrets Manager, will the pod automatically restart?

**Answer:** Not by default. External Secrets can sync the Kubernetes Secret, but pods usually do not restart automatically unless a reloader/controller, rollout restart, or checksum annotation pattern is used.

**Workflow:**
1. Secret changes in AWS.
2. External Secrets refreshes Kubernetes Secret based on refresh interval.
3. Application may need restart or dynamic reload.
4. Use Reloader, GitOps checksum annotation, or manual `kubectl rollout restart`.

---

# 10. Karpenter

## Q46. What is Karpenter?

**Answer:** Karpenter is a Kubernetes node lifecycle management and autoscaling tool that provisions right-sized compute for unschedulable pods and can deprovision underutilized nodes.

**Workflow:**
1. Pod cannot be scheduled.
2. Karpenter observes pending pod.
3. Karpenter evaluates NodePool constraints.
4. It creates a NodeClaim/EC2 instance.
5. Node joins cluster.
6. Pod schedules.
7. Karpenter later consolidates or expires nodes.

---

## Q47. Karpenter vs Cluster Autoscaler?

**Answer:** Cluster Autoscaler scales existing node groups. Karpenter provisions capacity more directly based on pod requirements and can choose instance types dynamically.

**Interview answer:** Karpenter is often faster and more flexible because it is not limited to predefined node group sizes in the same way.

---

## Q48. What are NodePools and NodeClasses in Karpenter?

**Answer:** A NodePool defines scheduling and lifecycle constraints. An EC2NodeClass defines AWS-specific configuration such as AMI, subnets, security groups, role, tags, and block devices.

**Workflow:**
1. Create EC2NodeClass for AWS infrastructure settings.
2. Create NodePool for workload constraints.
3. Pods use selectors, tolerations, affinities, and resource requests.
4. Karpenter provisions matching nodes.

---

## Q49. How do you control Karpenter cost?

**Answer:** Use consolidation, Spot where appropriate, instance family constraints, resource requests, disruption budgets, node expiration, and workload right-sizing.

**Failure modes:**
- Too many instance types restricted.
- Pods request too much CPU/memory.
- PDB blocks consolidation.
- Subnet lacks IPs.
- IAM role missing EC2 permissions.

---

# 11. Argo CD and GitOps

## Q50. What is Argo CD?

**Answer:** Argo CD is a GitOps continuous delivery tool for Kubernetes. It compares desired state in Git with live state in the cluster and syncs changes.

**Workflow:**
1. Developer opens PR.
2. PR reviewed and merged.
3. Git branch contains desired Kubernetes/Helm manifests.
4. Argo CD detects change.
5. Argo CD compares desired vs live state.
6. Argo CD syncs application.
7. Health and sync status are monitored.

---

## Q51. How does Argo CD use Helm?

**Answer:** Argo CD can render Helm charts using `helm template`, but Argo CD manages the application lifecycle instead of Helm releases.

**Workflow:**
1. Store chart or chart reference in Git.
2. Define values files.
3. Create Argo CD Application.
4. Argo CD renders manifests.
5. Argo CD applies manifests to cluster.
6. Drift is detected against Git state.

---

## Q52. What is sync status in Argo CD?

**Answer:** Sync status shows whether live cluster resources match the desired state in Git.

**Common statuses:**
- Synced: live matches Git.
- OutOfSync: live differs from Git.
- Unknown: Argo CD cannot determine state.

---

## Q53. What is health status in Argo CD?

**Answer:** Health status shows whether Kubernetes resources are operational, such as Deployments available, Pods running, or Services configured.

**Troubleshooting OutOfSync/Degraded:**
1. Check Application events.
2. Check manifest diff.
3. Check Kubernetes events.
4. Check pod logs.
5. Check CRDs/webhooks.
6. Check RBAC.
7. Check image pull errors.

---

## Q54. What is app-of-apps in Argo CD?

**Answer:** App-of-apps is a pattern where one parent Argo CD Application manages multiple child Applications.

**Use case:** Bootstrap clusters with standard add-ons like ingress, monitoring, External Secrets, Karpenter, and application namespaces.

---

## Q55. How do you roll back with Argo CD?

**Answer:** Revert the Git commit or sync to a previous known-good revision.

**Best practice:** Rollback should be Git-driven to preserve auditability.

---

# 12. GitHub PR and CI/CD

## Q56. What is a GitHub pull request?

**Answer:** A pull request proposes changes from one branch into another and enables review, discussion, checks, and approval before merge.

**Workflow:**
1. Create feature branch.
2. Commit code.
3. Push branch.
4. Open PR.
5. CI runs tests/security scans.
6. Reviewers approve.
7. Merge to main.
8. Deployment pipeline or Argo CD picks up change.

---

## Q57. What should a good PR include?

**Answer:** Clear title, problem statement, solution summary, testing evidence, screenshots/logs if needed, risk/rollback plan, and linked ticket.

**FAANG-style answer:** A strong PR is small, reviewable, tested, observable, and reversible.

---

## Q58. What is GitHub Actions?

**Answer:** GitHub Actions is a CI/CD platform that automates workflows such as building, testing, scanning, packaging, and deploying code.

**Workflow:**
1. Trigger on PR or merge.
2. Checkout code.
3. Install dependencies.
4. Run tests.
5. Build artifact/container.
6. Scan image.
7. Push image to registry.
8. Update Helm values or deploy.

---

## Q59. What checks should run before deploying to EKS?

**Answer:**
1. Unit tests.
2. Integration tests.
3. Linting.
4. Docker build.
5. Vulnerability scan.
6. Helm lint/template.
7. Kubernetes schema validation.
8. Policy checks.
9. Terraform plan if infra changed.
10. Approval gate for production.

---

# 13. CloudBees / Jenkins Deployment to EKS

## Q60. How does CloudBees deploy applications to EKS?

**Answer:** CloudBees/Jenkins pipelines can build artifacts, build and push Docker images, package Helm charts, authenticate to AWS/EKS, and deploy to EKS using Helm, kubectl, or GitOps handoff through Argo CD.

**Workflow:**
1. PR merge triggers pipeline.
2. Checkout source.
3. Run tests.
4. Build Docker image.
5. Push image to ECR.
6. Package Helm chart or update values.
7. Deploy using Helm/kubectl or update GitOps repo.
8. Argo CD syncs if GitOps model is used.
9. Verify rollout.
10. Notify team.

---

## Q61. CloudBees direct deploy vs Argo CD GitOps deploy?

**Answer:** In direct deploy, CloudBees applies changes to the cluster. In GitOps deploy, CloudBees updates Git and Argo CD applies changes.

**Best practice answer:** GitOps is often preferred for auditability, drift detection, rollback, and separation of CI from CD.

---

## Q62. What credentials are needed for CloudBees to deploy to EKS?

**Answer:** CloudBees needs permissions to access source code, registry, AWS/ECR, and either EKS deployment permissions or GitOps repository write permissions.

**Secure approach:** Use OIDC or short-lived credentials where possible, store secrets securely, and use least-privilege IAM roles.

---

# 14. Helm Deployment to EKS

## Q63. What is Helm?

**Answer:** Helm is a Kubernetes package manager that templates and packages Kubernetes manifests into charts.

**Workflow:**
1. Create chart.
2. Define templates.
3. Configure values.
4. Run `helm lint`.
5. Run `helm template`.
6. Deploy through Helm or Argo CD.

---

## Q64. How does an application use GitHub with Helm to deploy to EKS?

**Answer:** Application code and Helm charts live in GitHub. CI builds the image and updates Helm values. CD deploys the chart to EKS either directly or through Argo CD.

**Workflow:**
1. Developer pushes code.
2. PR checks run.
3. Merge triggers image build.
4. Image pushed to ECR.
5. Helm `values.yaml` updated with image tag.
6. Argo CD detects Git change.
7. EKS deployment rolls out new pods.
8. Health checks confirm success.

---

## Q65. What are common Helm chart files?

**Answer:**
- `Chart.yaml`: chart metadata.
- `values.yaml`: default configuration.
- `templates/`: Kubernetes manifest templates.
- `templates/deployment.yaml`: Deployment template.
- `templates/service.yaml`: Service template.
- `templates/ingress.yaml`: Ingress template.
- `_helpers.tpl`: reusable template helpers.

---

## Q66. How do you troubleshoot Helm deployment failure?

**Answer:**
1. Run `helm lint`.
2. Run `helm template` to inspect rendered YAML.
3. Check values file.
4. Check Kubernetes events.
5. Check CRD availability.
6. Check namespace/RBAC.
7. Check image pull errors.
8. Check readiness/liveness probes.

---

# 15. End-to-End Enterprise Workflow

## Q67. Explain the full workflow from VPC creation to application deployment on EKS.

**Answer:**
1. Create AWS accounts: network, security/logging, app.
2. Create VPC CIDR plan.
3. Create public/private subnets across AZs.
4. Attach Internet Gateway for public subnets.
5. Add NAT Gateway or VPC endpoints for private workloads.
6. Configure route tables.
7. Configure security groups.
8. Create KMS keys for encryption.
9. Create IAM roles and policies.
10. Create EKS cluster in private subnets.
11. Install EKS add-ons: VPC CNI, CoreDNS, kube-proxy, EBS CSI, Pod Identity agent.
12. Install platform add-ons: External Secrets, Karpenter, ingress controller, Argo CD, monitoring.
13. Configure storage with EBS/EFS.
14. Configure GitHub repositories and PR checks.
15. Configure CloudBees/GitHub Actions pipelines.
16. Build image and push to ECR.
17. Deploy Helm chart through Argo CD.
18. Validate rollout, logs, metrics, and alerts.

---

## Q68. Explain how traffic reaches an application running on EKS.

**Answer:**
1. User accesses DNS name.
2. DNS resolves to ALB/NLB.
3. Load balancer receives traffic.
4. Listener routes traffic to target group.
5. Target group sends traffic to pod IP or node port depending on target type.
6. Kubernetes Service routes to pod.
7. Pod handles request.
8. Logs/metrics/traces are collected.

---

## Q69. Explain how secrets reach an application pod.

**Answer:**
1. Secret is stored in AWS Secrets Manager.
2. External Secrets Operator has IAM permission through Pod Identity/IRSA.
3. ExternalSecret references the remote secret.
4. Operator creates/updates Kubernetes Secret.
5. Pod consumes secret as environment variable or mounted file.
6. Application reads secret.
7. Rotation requires app reload or pod restart unless app supports dynamic reload.

---

## Q70. Explain the deployment path using GitHub, CloudBees, Helm, ECR, Argo CD, and EKS.

**Answer:**
1. Developer opens PR in GitHub.
2. PR triggers CloudBees CI.
3. CloudBees runs tests and scans.
4. PR approved and merged.
5. Pipeline builds Docker image.
6. Image is pushed to ECR.
7. Pipeline updates Helm values with image tag.
8. GitOps repo receives commit.
9. Argo CD detects change.
10. Argo CD renders Helm chart.
11. Argo CD applies manifests to EKS.
12. Kubernetes rolls pods.
13. Argo CD reports sync/health.
14. Monitoring verifies service health.

---

# 16. Troubleshooting Scenarios

## Q71. Pods are stuck in Pending. What do you check?

**Answer:**
1. `kubectl describe pod`.
2. Resource requests vs available capacity.
3. Node selectors/taints/tolerations.
4. PVC binding.
5. Karpenter logs.
6. Subnet IP availability.
7. Quotas and limits.
8. PodDisruptionBudget or topology constraints.

---

## Q72. Pods are in ImagePullBackOff. What do you check?

**Answer:**
1. Image name and tag.
2. Registry exists.
3. ECR permissions.
4. Node/pod execution role.
5. ImagePullSecrets if external registry.
6. Network path to registry.
7. Rate limits or authentication failures.

---

## Q73. Service is down after deployment. What do you do?

**Answer:**
1. Check Argo CD health/sync.
2. Check rollout status.
3. Check pod status.
4. Check logs.
5. Check readiness probe.
6. Check Service selector.
7. Check Ingress/ALB target health.
8. Check recent config/secret changes.
9. Roll back if customer impact is high.

---

## Q74. Argo CD says OutOfSync. What does that mean?

**Answer:** Live cluster state differs from Git desired state.

**Workflow:**
1. View diff in Argo CD.
2. Identify manual changes or generated fields.
3. Confirm Git revision.
4. Sync if desired.
5. Add ignore differences only for expected controller-managed fields.

---

## Q75. Application cannot access AWS Secrets Manager. What do you check?

**Answer:**
1. Pod service account.
2. Pod Identity/IRSA association.
3. IAM role trust policy.
4. IAM policy allows `secretsmanager:GetSecretValue`.
5. KMS decrypt permission if secret uses CMK.
6. Secret ARN and region.
7. Network access to Secrets Manager endpoint.
8. CloudTrail access denied event.

---

## Q76. Karpenter is not launching nodes. What do you check?

**Answer:**
1. Pending pod events.
2. Karpenter controller logs.
3. NodePool and EC2NodeClass.
4. Subnet/security group discovery tags.
5. IAM permissions.
6. Instance type constraints.
7. AWS service quotas.
8. Capacity availability.
9. Pod requirements too restrictive.

---

## Q77. CoreDNS is failing. What breaks?

**Answer:** Service discovery breaks. Pods may fail to resolve Kubernetes services and external DNS names.

**Troubleshooting:**
1. Check CoreDNS pod status/logs.
2. Check kube-dns service.
3. Check ConfigMap.
4. Check node networking.
5. Check upstream DNS.
6. Test DNS from a debug pod.

---

## Q78. PVC is stuck Pending. What do you check?

**Answer:**
1. StorageClass exists.
2. CSI driver installed.
3. IAM permissions.
4. Requested access mode.
5. Availability Zone compatibility.
6. EBS/EFS configuration.
7. Events on PVC and provisioner logs.

---

# 17. FAANG-Style System Design Questions

## Q79. Design a production-grade EKS platform for 100 application teams.

**Answer outline:**
1. Multi-account AWS structure.
2. Central network account.
3. Separate dev/stage/prod app accounts.
4. EKS clusters per environment or tenant model.
5. Private subnets across 3 AZs.
6. Karpenter for dynamic scaling.
7. Argo CD app-of-apps for platform add-ons.
8. External Secrets with AWS Secrets Manager.
9. Pod Identity for least privilege.
10. EBS/EFS storage classes.
11. Central observability.
12. Policy enforcement.
13. CI/CD with PR gates.
14. Disaster recovery and backup.

**Tradeoffs:**
- Cluster-per-team gives isolation but higher cost.
- Shared cluster lowers cost but needs strong RBAC, quotas, and network policies.

---

## Q80. Design CI/CD for microservices deploying to EKS.

**Answer outline:**
1. GitHub PR workflow.
2. Required checks.
3. Build container image.
4. Push to ECR.
5. Scan image.
6. Update Helm values.
7. Argo CD deploys.
8. Progressive rollout.
9. Metrics-based verification.
10. Rollback through Git revert.

---

## Q81. Design secure secret management for EKS.

**Answer outline:**
1. Secrets stored in AWS Secrets Manager.
2. KMS CMK encrypts secrets.
3. External Secrets syncs to Kubernetes.
4. Pod Identity grants least-privilege access.
5. Kubernetes RBAC restricts Secret access.
6. Rotation process defined.
7. Audit with CloudTrail.
8. Restart/reload strategy implemented.

---

## Q82. Design a multi-account network for app teams.

**Answer outline:**
1. Network account owns TGW/shared VPC/DNS.
2. App accounts own workloads.
3. Security account owns inspection/logging controls.
4. Use AWS RAM for sharing.
5. Non-prod and prod route tables separated.
6. Central egress inspection if required.
7. Private endpoints for AWS services.
8. Route 53 private hosted zones for internal DNS.

---

# 18. Behavioral / STAR Questions

## Q83. Tell me about a time you improved reliability.

**STAR template:**
- **S:** A service had repeated deployment-related outages.
- **T:** Improve deployment reliability.
- **A:** Added CI checks, Helm validation, Argo CD sync visibility, readiness probes, and rollback runbook.
- **R:** Reduced failed deployments and improved recovery time.

---

## Q84. Tell me about a time you automated a manual process.

**STAR template:**
- **S:** Engineers manually created secrets and restarted pods.
- **T:** Automate secret onboarding.
- **A:** Implemented External Secrets, IAM role mapping, and documented onboarding steps.
- **R:** Reduced manual effort and improved consistency.

---

## Q85. Tell me about a time you handled a production incident.

**STAR template:**
- **S:** Deployment caused pods to fail readiness checks.
- **T:** Restore service quickly.
- **A:** Checked Argo CD, Kubernetes events, logs, and rolled back the image tag.
- **R:** Service recovered quickly; postmortem added better pre-deploy validation.

---

## Q86. Tell me about a time you had a disagreement with another team.

**STAR template:**
- **S:** App team wanted broad AWS permissions for speed.
- **T:** Keep delivery moving while protecting security.
- **A:** Proposed least-privilege role with scoped resource ARNs and conditions.
- **R:** App deployed successfully without granting unnecessary access.

---

## Q87. Tell me about a time you improved deployment speed.

**STAR template:**
- **S:** Deployments required manual approvals and kubectl commands.
- **T:** Speed up safe delivery.
- **A:** Created PR-based pipeline, automated tests, Helm templating, and Argo CD sync.
- **R:** Reduced deployment cycle time and improved auditability.

---

# 19. Rapid-Fire Interview Questions

## AWS / VPC

1. **What makes a subnet public?** Route to Internet Gateway.
2. **What makes a subnet private?** No direct route to Internet Gateway.
3. **Why use NAT Gateway?** Private outbound internet access.
4. **Why use VPC endpoints?** Private access to AWS services without internet/NAT.
5. **Why avoid overlapping CIDRs?** Routing ambiguity and failed connectivity.
6. **What is Transit Gateway?** Central hub for VPC/on-prem connectivity.
7. **What is VPC sharing?** Central VPC owner shares subnets with participant accounts.

## IAM / KMS

8. **What is a role?** Assumable AWS identity.
9. **What is a policy?** Permission document.
10. **What is a trust policy?** Defines who can assume a role.
11. **What is a key policy?** Resource policy attached to a KMS key.
12. **Why avoid `Resource: "*"` for KMS?** It can over-grant access across keys/accounts.
13. **What is least privilege?** Only required permissions.

## EKS

14. **What does CoreDNS do?** Cluster DNS resolution.
15. **What does kube-proxy do?** Service networking rules.
16. **What does VPC CNI do?** Assigns pod IPs from VPC.
17. **What is Pod Identity?** IAM role association for pods.
18. **What is Fargate?** Serverless pod compute.
19. **What is Karpenter?** Dynamic node provisioning and consolidation.
20. **EBS vs EFS?** Block single-writer vs shared network filesystem.

## CI/CD / GitOps

21. **What is GitOps?** Git is the source of truth for desired state.
22. **What is Argo CD sync?** Applying Git desired state to cluster.
23. **What is Helm?** Kubernetes templating/package manager.
24. **What is a PR gate?** Required validation before merge.
25. **Direct deployment vs GitOps?** Pipeline applies directly vs pipeline updates Git and Argo applies.

---

# 20. Commands to Know

## AWS

```bash
aws sts get-caller-identity
aws eks update-kubeconfig --region us-west-2 --name <cluster-name>
aws eks describe-cluster --name <cluster-name> --region us-west-2
aws iam get-role --role-name <role-name>
aws kms describe-key --key-id <key-id>
```

## Kubernetes

```bash
kubectl get nodes -o wide
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
kubectl rollout status deploy/<deployment> -n <namespace>
kubectl rollout restart deploy/<deployment> -n <namespace>
```

## Helm

```bash
helm lint ./chart
helm template myapp ./chart -f values.yaml
helm upgrade --install myapp ./chart -n myns -f values.yaml
helm history myapp -n myns
helm rollback myapp <revision> -n myns
```

## Argo CD

```bash
argocd app list
argocd app get <app-name>
argocd app diff <app-name>
argocd app sync <app-name>
argocd app history <app-name>
```

## Troubleshooting DNS

```bash
kubectl run dns-test --image=busybox:1.36 -it --rm --restart=Never -- nslookup kubernetes.default
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

# 21. Strong Interview Closing Summary

A strong SRE/DevOps/AWS/EKS interview answer should sound like this:

> I would design the platform using a multi-account AWS model with a centralized network account and separate application accounts. I would plan non-overlapping CIDR ranges, private EKS clusters across multiple AZs, least-privilege IAM roles, KMS encryption, Pod Identity for workload access, External Secrets for secret delivery, Karpenter for dynamic scaling, EBS/EFS for storage needs, and Argo CD for GitOps-based deployment. Application teams would use GitHub PRs and CI checks to build/test/scan images, push to ECR, update Helm values, and let Argo CD deploy to EKS. For reliability, I would add observability, SLOs, alerting, rollback workflows, and postmortems.

---

# 22. Extra STAR Stories You Can Customize

## Story A: Building EKS Platform Foundation

**S:** The organization needed a scalable AWS foundation for applications across on-prem and SaaS-style environments.

**T:** Build secure cloud infrastructure and deploy applications on EKS.

**A:** Created VPC, security groups, Internet Gateway/NAT paths, load balancer integration, EKS cluster, EFS, External Secrets, Karpenter, Argo CD, and CI/CD pipeline integration.

**R:** Enabled repeatable application deployment, improved platform scalability, and created a reliable foundation for future workloads.

---

## Story B: Secret Onboarding Automation

**S:** Application teams needed a secure and repeatable way to consume secrets in Kubernetes.

**T:** Replace manual Kubernetes Secret creation with a controlled workflow.

**A:** Integrated AWS Secrets Manager, KMS, External Secrets Operator, IAM roles, and namespace-level access patterns.

**R:** Reduced manual work, improved security, and made secret delivery auditable.

---

## Story C: GitOps Deployment Standardization

**S:** Different teams deployed applications inconsistently using manual commands and custom scripts.

**T:** Standardize deployment to EKS.

**A:** Introduced Helm chart structure, GitHub PR workflow, CI checks, image publishing to ECR, and Argo CD sync.

**R:** Improved deployment consistency, rollback safety, and auditability.

---

## Story D: Karpenter Cost Optimization

**S:** EKS clusters were over-provisioned with fixed node groups.

**T:** Improve cost efficiency without reducing reliability.

**A:** Added Karpenter NodePools, right-sized pod requests, enabled consolidation, and separated system/application workloads.

**R:** Improved utilization and reduced unnecessary compute spend.

---

# 23. Final Practice Prompts

Use these to rehearse:

1. Explain a full AWS-to-EKS platform build from scratch.
2. Explain how a GitHub PR becomes a running pod in EKS.
3. Explain how a secret in AWS Secrets Manager reaches a pod.
4. Explain how Karpenter decides to launch a node.
5. Explain why a pod is stuck Pending.
6. Explain why an app cannot reach the internet.
7. Explain how to secure IAM permissions for pods.
8. Explain how Argo CD handles drift.
9. Explain how Helm is used in GitOps.
10. Explain how you would design a multi-account AWS network.

---

# 🧠 FAANG Mock Interview Questions

## EKS Pod to S3 Access

**Question:** How do you allow an EKS pod to securely access an S3 bucket?

**Answer:**  
Use IRSA or EKS Pod Identity instead of static AWS keys.

Workflow:
1. Create IAM policy with least-privilege S3 access.
2. Attach policy to IAM role.
3. Map IAM role to Kubernetes ServiceAccount.
4. Pod uses temporary credentials through STS.
5. Validate from inside pod:

```bash
aws sts get-caller-identity
