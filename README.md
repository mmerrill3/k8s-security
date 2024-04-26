# k8s-security

A non comprehensive road map to better security posture with kubernetes in the Clouds... DevSecOps.

Obviously, there are greater details and cultural preferences in every organization.


## Security Areas

Before jumping into running workloads as container in any Cloud provider, some things below to consider.


### Cloud Provider Level

#### Azure Policy

Managemenent Groups and policies in Azure should be implemented to enforce access restrictions.

#### AWS Policy

Similar to Azure, implemented to enforce access restrictions.  Since Infrastructure as code is desired, we do not want to create drift between what is deployed, and the current state of an IaC defintion.

### Orchestration Framework Level

Things that can be done within a kubernetes cluster, controller by either a product owner or a cluster administrator.

#### Observability

It's imperative to have something as simple as the [kube-prometheus](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) stack installed on a cluster.  This comes with default grafana dashboards, and alert manager rules that can let engineers know about issues proactively.

Define an escalation path, and have well defined roles for who can help when issues arise.

See [CNCF Landscape](https://landscape.cncf.io/) for other solutions.

#### Compliance

This is critical, it is imperative to deploy a tool like [OPA Gatekeeper](https://github.com/open-policy-agent/gatekeeper) as a policy enforcement engine.  Look to kubesec.io's list of common policies [here](https://github.com/raspbernetes/k8s-security-policies).

Leveraging a centralized instance (control plane), security organizations can enforce delivery of tools into clusters

- gatekeeper
- rego policies for gatekeeper
- malware detection daemonset
- intrusion detection daemonset
- 

#### Configuration Benchmarks

Leveraging tools like [kubebench](https://aquasecurity.github.io/kube-bench/v0.6.15/), we should be generating reports against CSI benchmarks for best practices, and protection against known vulnerabilities.

##### Penetration Testing

As part of the configuration benchmarks, penetration tests can be ran to check configuration against common attack methods.  See [kube-hunter](https://github.com/aquasecurity/kube-hunter) as an example.

#### Image Scanning

Images should be scanned at rest to identity vulnerabilities before they are published to an OCI repository.  Usually, this is an out of the box offering for many OCI repositories like AWS ECR, or AWS ACR.

Images should be able to be scanned alive in an infrastructure as well.  This can be accomplished using [starboard](https://aquasecurity.github.io/starboard/v0.15.12/) or other tools like snyk.io, or Palo Alto's [Prisma cloud](https://www.paloaltonetworks.com/prisma/cloud).  See [CNCF Landscape](https://landscape.cncf.io/) for others.

#### Tagging Strategy

All resources created to support workloads, either in the cloud provider or by the kubernetes API (pods, deployment, stateful sets, jobs) should have a well defined tagging and label strategy.  This is useful beyond security (see financial operations), but helps a security engineer quickly identify context around a workload. 
- 
### Organization

Cultural points within an organization that should be considered 

#### Continuous Integration

Enable webhooks in your git repositories to kick of continuous integration build upon each commit or PR creation.

#### Tests

Use containers to simulate a test environment whenever possible.  Ideally, each enhancement or bugfix should be paired with updates to a testing framework.

#### Infrastructure as Code for Cloud resources

Leverage tools like terraform or openTofu to easily create Cloud infrastructure, including managed kubernetes instances like Amazon EKS or Azure AKS.  By policy, access to create these resources should be limited so the infrastructure can be described and committed to a git repository.

This will allow for automation and reconciliation of infrastructure that is desired.

**Note** 
Avoid software workload definitions in your infrastructure.  Leverage a separate deployment model in regards to applications.

#### IDE

Leverage integration with linting and static analysis plugins to identify performance, resource, or security issues from the beginning.

#### Code Reviews

Tried and true, requiring digital signatures on pull requests by a repository owner, and managing feedback, can help with preventing vulnerabilities from being introduced.

Analysis of tests that truly make sense.

#### Pull Request Management

Processes and checks that should be place before merges occur in git and release tags are made.
For example:
- prevent commits to long lived branches like main.
- commit messaging is intuitive and follows a pattern.  For example, use prefixes like doc:, or feat:.  Squash extraneous commit messages.

Leverage chatops to automate steps in a PR flow, like /lgtm, or /ok-to-test, or /approve

Enable helper utilities to add tags to PRs for complexity scoring, or for PR size.  Automatically assigning PRs to reviewers.

#### Git Flows

Define how software is tagged and gets into environments.
Define the graduation between environments, for instance, from staging/qa into production.  Can you use simple gitflows or can you use a tool like [kargo](https://github.com/akuity/kargo) to package "releases" and their configuration.

#### Deployments

At the very least, don't apply manifests into kubernetes manually.  Use a tool like [helm](https://github.com/helm/helm) to deploy workloads.

Ideally, leverage a Continuous Delivery tool like [argoCD](https://github.com/argoproj/argo-cd) to enforce applications are running in a target cluster with a given configuration.  This prevents users from altering configurations, or workload specifications.

Offer patterns to other for best practices around the following areas.  These can be automated in a fully managed scenario:

- automatic DNS integration for kubernetes services and ingresses
- automatic installation of a prometheus stack
- cert manager integration
- ingress controller like nginx
- istio as a service mesh
- secrets management
- grafana loki or ELK 

#### Agile/Fragile

Find the right process for your engineers for how to deliver code.  Don't do agile practices if they add no value.

#### SRE Practices

Blameless post mortems with actionable follow up tasks.
Setup dashboards early for applications, and instrument the applications to emit meaningful metrics.
Add dashboards for common patterns, like HTTP metrics for an ingress controller.
