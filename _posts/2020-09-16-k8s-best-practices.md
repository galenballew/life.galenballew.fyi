---
title: "Kubernetes Best Practices"
date: 2020-09-16 00:00:00
excerpt: 'TL;DR - Blueprints for building successful applications on Kubernetes.'
categories: [cloud, tldr]
tags: [cloud, aws, gcp, azure, k8s, tldr]
featured_image: '/images/feature/3.jpg'
comments: true
share: true
---
# Table of Contents
{:.no_toc}
- toc
{:toc}

---
# [Abstract](#abstract)
[Kubernetes Best Practices](https://www.oreilly.com/library/view/kubernetes-best-practices/9781492056461/) was authored by Brendan Burns, Eddie Villalba, Dave Strebel, and Lachlan Evenson. These are the working notes that I took while reading the book. I have a solid working knowledge of Kubernetes and did not seek to regurgitate everything I read. Instead, I hope that this article consists of the juiciest tid bits and highlights that are easily consumable for anyone familiar with k8s. Enjoy! 

---
# [Setting Up a Basic Service](setting-up-a-basic-service)
Kubernetes starts with containers - immutable containers to be specific. Do not use the `latest` tag for your image versions. Instead, use a combination of semantic versioning and the SHA hash of the commit where the image was built (e.g., v1.0.1-bfeda01f).

It's recommended to set a container's Requests and Limits are set equal to each other. This results in very predictable pod behavior, as well as predictable utilization. You can certainly set the values for Requests and Limits independently in order to drive maximal utilization, but most users find that the stability/predictability from setting them equal is more valuable than the gained utilization. 

The key/values in a ConfigMap are subject to change. When a key/value needs to be updated, it may be tempting to edit the ConfigMap
YAML and apply the update in-place. However, this will not trigger an update to existing Pods using the ConfigMap - they will only use the new configuration after a restart. Rather than updating the existing ConfigMap, adding version numbers to your ConfigMaps, deploying an entirely new ConfigMap, and updating the Deployment to use this new one. Rollout of the new configuration will be automatically triggered. Additionally, the previous version of the ConfigMap will still be available in the cluster - rollback is just a matter of updating the Deployment again. 

Secrets are stored unencrypted within etcd. If you want to protect that sensitive data from a direct attack against etcd, you can provide Kubernetes with a key that it will use to encrypt the data at rest. There's more information available [in the documentation.](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

Generally speaking, managing state is hard. Also, generally speaking, stateful services (e.g., Redis as a service) are worth the extra cost. The book doesn't offer any additional information or opinions beyond this, but there is an excellent [blog article](https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-mapping-external-services) by Google about mapping to external services. The book has an entire chapter on [integrating external services](#integrating-external-services-and-kubernetes), as well as a chapter on [managing state and stateful applications!](#managing-state-and-stateful-applications)

Even if you're an individual developer, you will eventually want your k8s configuration yaml to be deployed to multiple, different endpoints (e.g., dev, stage, prod). Rather than having multiple copies of the same code, you should use a packing tool like [Helm](https://helm.sh/) to parametrize your configuration. Instead of having multiple copies of application YAML, you can have a single `values.yaml` per deployment environment and you can keep them all in a `templates/` directory at the root of your project. The book doesn't talk about it, but this also gives you the added benefit of building modular, reusable k8s building blocks. These can be shared and leveraged across the company so that teams don't need to reinvent the wheel. 

---
# [Developer Workflows](#developer-workflows)
When thinking about developer interactions with a development cluster, it can be useful to break it down into three phases:
1. Onboarding - getting access to the cluster
2. Developing - getting bootstrapped and able to deploy
3. Testing - being able to iterate quickly and efficiently

Generally speaking, it's a best practice to use a single, large cluster for all developers and break it down into namespaces to keep things tidy. There is some additional overhead of managing this cluster as opposed to creating individual clusters per developer, but the increased resource utilization and availability of shared services (e.g., logging) make it worthwhile. When you're operating in this model, it is also a good idea to give everyone read access to the entire cluster. This can help everyone debug in case your neighbor is breaking your deployment or hogging all the resources. However, it's worth noting that the default read access will include Secrets. You can create fine-grain permissions that exclude Secrets if necessary, but it's usually not a problem in a development environment. 

It can be extremely useful to automate as much of the namespace management as possible. This could be a script that creates a new namespace, adds users to it, defines the Requests and Limits, and sets a TTL. The idea of a TTL is useful because it helps keep the cluster lean and clean (and helps build good developer habits). You can enforce the TTL by writing a [CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) to clean them up. You could also replace the script that instantiates the namespace with a [CRD](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)!

If your development namespace has a TTL, then you're definitely going to want a script that installs all the necessary dependencies for your project. It is also a best practice to have a script that will delete and recreate your Deployments rather than update them in-place. In a development environment, you want your code changes to rollout quickly, but modifying the k8s rollout logic should only be done with extreme caution - you do not want to create drift between your development and production clusters. 

---
# [Monitoring and Logging in Kubernetes](#monitoring-and-logging-in-kubernetes)
There are two monitoring paradigms that complement each other and are useful in the context of Kubernetes: **USE** and **RED**. 

- **U** - Utilization
- **S** - Saturation
- **E** - Errors
<br>
<br>
- **R** - Rate
- **E** - Errors
- **D** - Duration

USE focuses more on the infrastructure, whereas RED focuses on the end-user experience for the application. These patterns should help determine what you monitor, because monitoring everything is counter-productive (poor signal:noise ratio).

Rather than focusing on the details of [the Metrics API](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/#the-metrics-api), [Metrics Server](https://github.com/kubernetes-sigs/metrics-server), [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics), or any of the multitude of monitoring tools available (e.g., [Prometheus](https://prometheus.io/docs/introduction/overview/)), I'd like to focus on what metrics should be captured. Monitoring should take a layered approach that takes into account the following:
- Physical or virtual nodes
- Cluster components
- Cluster add-ons
- End-user applications

Given these layers, what metrics should be targeted?
- Nodes
  - CPU utilization
  - Memory utilization
  - Network utilization
  - Disk utilization
- Cluster components
  - etcd latency
- Cluster add-ons
  - Cluster Autoscaler
  - Ingress Controller
- Application
  - Container memory utilization and saturation
  - Container CPU utilization
  - Container network utilization and error rate
  - Application frame-work specific metrics

With regards to logging, there are several components that you will need to capture logs from:
- Node logs
  - Docker daemon
- Kubernetes control-plane logs
  - API server
  - Controller manager
  - Scheduler
- Kubernetes audit logs
- Application controller logs

It is worth mentioning that k8s audit logs can be extremely noisy. It's worth using the [documentation](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/) to fine-tune them for your environment.

Limit the usage of log forwarders in a sidecar pattern - they consume a lot more resources. Opt for using a DaemonSet for the log forwarder and sending the logs to STDOUT. 

The book mentions SRE practices like defining SLOs and measuring them using SLIs, but those methodologies are beyond the scope of the book. They're still best practices though and should be implemented in order to provide a highly available, secure Kubernetes platform. All of the [Google books on SRE are available for free](https://landing.google.com/sre/books/). 

---
# [Configuration, Secrets, and RBAC](#configuration-secrets-and-rbac)
Generally speaking, the best way to approach any changes to a ConfigMap or Secret is to update the entire Deployment. This is because a change to the configuration object will not trigger an update of the Pods that reference/mount it. It is a best practice to use a version number in the `name` of the configuration object and change the `Deployment` to point at the new version. This will cause all of the Pods to update. It also makes rollback easy.

You can assign an `imagePullSecrets` to a `serviceaccount` that the pod will use to automatically mount the secret without having to declare it in the `pod.spec`. You can do this to the default service account for the namespace and the secret will be automatically added to all the pods in that namespace. 

RBAC has a lot in common with AWS IAM as far as best practices for scope of privilege (TL;DR - use least privilege). I've left out most of the information about RBAC, but it is worth explicitly stating that *most* application that run on Kubernetes will **not** need an RBAC role or role binding. This is because most applications do not directly interact with the k8s API itself. In the case where an application does need to interface with the k8s API, it should use a purpose-specific service account with a least privileged role to execute its goal. 

---
# [Continuous Integration, Testing, and Deployment](#continuous-integration-testing-and-deployment)
CI/CD, DevOps, GitOps, etc., all deserve and have their own books about them. With regards to best practices for Kubernetes, there will a few specific and salient points that jumped out at me:

1. Whatever CI/CD tools that you choose must be able to define the pipeline as code. Storing the pipeline in version control alongside the application code is invaluable.
2. Optimize your container images for size. Smaller images means more efficient development and lower security risk.
   - As you mature, you may become comfortable using a [distroless base image.](https://github.com/GoogleContainerTools/distroless)
   - Use multi-stage builds to slim down containers. A classic example is to build your Go static binary and place only that into the production container - not all the packages that were used to build it.
3. **Do not use the `latest` tag!** Every container image tag should point to a specific version/build of the code.
4. Kubernetes offers several different deployment strategies. If you're just getting started, rolling deployments are the easiest to use. When using blue/green or canary, be sure to understand the implications for having two versions of your application available - especially as it relates to state and/or hybrid applications.

---
# [Versioning, Releases, and Rollouts](#versioning-releases-and-rollouts)
Versioning should be a relatively straightforward task, but it's a critical one. There are so many different types of Kubernetes API objects, each with their own version, that making sure everything is properly versioned becomes mission critical in order to enable smooth operations.

As far as rollouts are concerned, it's imperative to understand that changes to the metadata fields of a Deployment will not trigger an update. Only changes to the `spec.template` will trigger an update. This becomes important as you will have versions for containers, Pods, Deployments, Services, and the application itself - all of which should be distinct from each other.  

Use a release and release version/number in your Deployment metadata. The release name and number should coordinate with the actual release from your CI/CD tooling in order to enable traceability.  

If you're using Helm, be sure to bundle services that need to be rolled back or upgraded together. You can use [Helm chart hooks](https://helm.sh/docs/topics/charts_hooks/) to make sure that release lifecycle events go smoothly. Chart hooks allow events (e.g., run a Job, mount a ConfigMap, etc.) to be triggered at specific times in the release lifecycle (e.g., `pre-install`, `post-rollback`, etc.)

---
# [Worldwide Application Distribution and Staging](#worldwide-application-distribution-and-staging)
Starting with the fundamentals, it's imperative to have your container images distributed to each region that your workload is running in. This helps to make rollouts more reliable by avoiding any networking issues that may crop up.

Global rollouts should be preceded by rigorous integration testing. Ideally, you will have a copy of your production data available for this - try to think ahead _before_ you're scaled up. Setting up good integration testing is easier early on in the development of an application and it pays serious dividends in the long run. Load-testing should also be performed. Replaying the logged traffic is a good starting point, but not always fool-proof.

Once you're ready to begin the rollout, a canary region should be selected and deployed to. Your customers will use this region as a preproduction environment to validate _their_ use of your service before continuing the rollout. Generally speaking, a decent rule of thumb is to leave this deployment in the canary region for double the average time-to-smoke. 

Lastly, be sure to document and **practice** your response to any problems or processes that you encounter. There should be runbooks for everything you need to do - nothing should be done from memory. 

---
# [Resource Management](#resource-management)
Both the general guidelines and the specific implementations of resource management depend on the workload and the underlying hardware it requires. Any specialized hardware (e.g., GPUs) should be accessed with taints and/or nodeSelectors. Use nodeSelectors when you want to _request_ specialized hardware and use taints when you want to _reserve_ it. If you're running multiple workloads with different performance requirements, be sure to include a mix of node pools with different instance types. If those workloads are variable or have unexpected spikes in usage, consider using the Horizontal Pod Autoscaler. 

As far as Pods go, everything should have its Requests and Limits defined. If a Pod's Requests and Limits are equal, the Pod will receive a _guaranteed_ Quality of Service (QoS) class. This is partly why it's recommended to set them equal when you're getting started on your k8s journey. However, guaranteed QoS requires that Requests and Limits are set _for all containers in a Pod_. 

Additionally, use PodDisruptionBudgets to manage how many Pods are (un)available during a disruption. This, in congruence with (anti-)affinity attributes, can ensure that your service remains highly available through out operations and outages. 

---
# [Networking, Network Security, and Service Mesh](#networking-network-security-and-service-mesh)
Probably the single most important decision around networking is what Container Network Interface to use. This deserves its own article. The chosen CNI should deliver the feature set required, be compatible with your control plane (particularly important when consuming a managed service like AWS EKS), and be compatible with your network observability, management, and security tooling. And, if you choose to use a CNI that does not over a network overlay, you'll need to make sure that you have enough CIDR to handle node IPs, Pod IPs, load balancers, and wiggle room for rollouts and scaling. 

The next big decision to make is the choice of Ingress controller. Just like choosing a CNI, a thorough investiation of feature set and compatability needs to be performed. Standardize the chosen Ingress controller across the enterpise because many of the specific configuration annotations vary between implementations. If you aren't consistent, your workloads won't be portable. 



