DevSecOps refers to the integration of security practices into a DevOps software delivery model, with a goal of enabling developers to build more secure software. With this movement over the past years, developers have shifted left and are implementing the tools and practices supplied by the DevSecOps ecosystem to deliver more secure code.

> The term DevSecOps has been heavily watered down from where it started and takes on different meanings based on who you ask or where you look up the definition. Additionally, it only applies to developer security practices for how software is written and deployed.

Enter MeshSecOps…

If we take a deeper look at the more progressive “cloud native” production environments where this newly secured DevSecOps-stamped code will run, it’s pretty clear to see that it is a mesh of Microservices (or service mesh). A service mesh allows developers to shift the complexity of security, reliability and observability away from their application code into the mesh platform.

DevSecOps is a movement about securing code
MeshSecOps is a movement about securing the new mesh platform

Just as we helped developers write more secure code through DevSecOps principals, it’s time that we help (Platform, SREs, DevOps, Infrastructure) Engineers run, monitor and secure the  runtime compute and network platform in production service mesh environments through MeshSecOps.

> MeshSecOps inherits the DevSecOps principles that are timeless and relevant to today’s Microservice and Service Mesh engineering environments - removing the ones that should be implicit or are beyond our control to change. 

The following principles of MeshSecOps were gathered from leading edge Platform Engineering practitioners and condensed to be easy to understand. They meet Zero Trust Architecture guidelines from NIST Security Standards for Microservices (SP 800-204) and the U.S. Department of Defense Zero Trust Reference Architecture frameworks. 

### Principle #1: Assume no implicit or explicit trusted zone in mesh networks.

* Mesh Microsegmentation
  * A security technique that creates secure zones in cloud deployments and on-prem deployments to allow organizations to isolate workloads from one another and secure them individually. Mesh Microsegmentation is effective for reducing the surface area exposed to attackers and their ability to move laterally once inside the network. Most segmentation tools are built for a previous generation (Web 2.0) with a slower rate of change than what cloud native environments see today. The service mesh can help facilitate communication between segments while providing the opportunity for higher-level policy to be applied through network, eBPF, and Service-based Access Control (SBAC) policies.

### Principle #2: Identity-based authentication and authorization are strictly enforced for all connections and access to infrastructure, data, and services.

* Identity and Certificate Management
  * When an application runs within a service mesh environment, each service is provided with an identity. This identity is used when connecting to other microservices running in the service mesh. Service identities enable mutual authentication of services to validate that a connection is allowed and to enforce authorization policies.
  * A service mesh should issue certificates to all workloads, rotate those certificates, and validate them at runtime,  without application involvement. 
    * The service mesh encodes an application identity into the X.509 certificates it issues to workloads (via SPIFFE). 
    * For a production cluster setup, it is highly recommended to use a production-ready Certificate Authority for signing requests.

* Secret Management
  * A secret is anything that you want to tightly control access to, such as API keys, passwords, certificates, and more. A service mesh requires access to a multitude of secrets: database credentials, API keys for external services, credentials for service-oriented architecture communication, etc. Understanding who is accessing what secrets is already very difficult and platform-specific. Adding on key rolling, secure storage, and detailed audit logs is almost impossible without a custom solution. This is where solutions like Hashicorp Vault steps in.
  * By default, Secrets are stored in etcd using base64 encoding. In environments with stringent security policies, this might not be acceptable, so additional security measures are needed to protect them.
  * For service mesh, a key management system should be able to provision secrets for multiple clusters at once.

### Principle #3: Machine to machine (M2M) authentication and authorization are strictly enforced for communication between servers and the applications.

* Authentication 
  * Authn (or Authentication) Is used to verify the person or service is who they say they are. Mutual TLS, or mTLS for short, is a method for mutual authentication. mTLS ensures that the parties at each end of a network connection are who they claim to be by verifying that they both have the correct private key. Once you have an Auth token or Service/Workload ID established through mTLS, you can access resources through gatekeepers at the workload-level based on roles and other permission identifiers contained within the token. 
  * In service mesh, there are 2 types of authentication providers:
    * Peer authentication: used for service-to-service authentication to verify the client making the connection. Peer authentication provides each service with a strong identity representing its role to enable interoperability across clusters and clouds. 
    * Secures service-to-service communication.
    * Provides a key management system to automate key and certificate generation, distribution, and rotation.
  * Request authentication:  Service mesh also allows for request-level authentication with JSON Web Token (JWT) validation and a streamlined deployment using a custom authentication provider or any OpenID Connect providers (ORY Hydra, Keycloak, Auth0, Firebase Auth, Google Auth)

* Authorization 
  * To fully lock down traffic, it is recommended to configure authorization policies. Authz (or Authorization) can be implemented with or without Authentication. Depending on the architecture, normally Authn will have taken place and the user will bear some type of token (e.g. JWT) in their Authz request.
  * Authz provides fine-grained policies to allow or deny traffic across workloads in the mesh. 
  * Projects like OPA Gatekeeper and Kyverno exist to provide Authz policy on top of Kubernetes. The OPA-Envoy plugin is frequently deployed in Kubernetes environments as a sidecar container to provide fast policy-based decisions.
  * Other projects provide Service-based Access Control (SBAC) to enforce the flow of sensitive data between services. SBAC enforces traffic across workload identities as an overlay to existing Authz policies.
  * Authz Policy Bypass
    * The enforcement point for authorization policies is the mesh proxy instead of the usual resource access point within the containerized application. A policy mismatch happens when the mesh proxy and the containerized application interpret the request differently. A mismatch can lead to either unexpected rejection or a policy bypass. The latter is usually a security incident that needs to be fixed immediately, and it’s also why we need path normalization in the authorization policy.
    * Specialized micro firewall projects provide additional normalization options. They can be deployed as part of the service mesh sidecars or ingress gateway to normalize requests in the mesh. The authorization policy will then be enforced on the normalized requests. 

### Principle #4: Risk profiles, generated in near real-time from monitoring and assessment of both user and device behaviors, are used in authorizing users and devices to resources.

* MicroFirewall
  * Malicious traffic traveling between services contains different signatures than normal internet based client traffic. Because of the limited nature of request signals in M2M mesh environments, all incoming requests and outgoing responses (east-west, north-south) should be analyzed to define risk profiles. 
For example, a single local IP address with no identifying client side signals may attempt to DDOS Service A by sending too many requests that are compute-intensive.
  * A Microfirewall maintains three functions to enforce risk profiles - Block, Rate Limit, and Allow. These functions may leverage the following mesh-related signals to identify attacks:
    * Workload Identity (SPIFFE ID)
    * Local IP Address
    * Authentication Token (JWT) or other request header/body signals

* Inline Response Analysis
  * Response codes (2xx,3xx,4xx,etc.) do not provide enough data to determine if an exploit has been executed in situations like SSRF or RCE types of attacks across microservices. Through the use of Inline Response Analysis across Layers 4-7, predefined exploit patterns (or payloads) in live traffic can be identified and remediated. This provides zero day defense against unknown attacks that can execute a system command based on request traffic. 
  * Additionally, if attacks like SQL Injection, Broken Object Level Authorization, or other API vulnerabilities release a large amount of sensitive data, Inline Response Analysis should provide audit and remediation capabilities. 

* Token/Identity Fraud Analysis 
  * Services collaborate to fulfill a business case. But when all services in the service mesh require authorization, service requests must contain authorization data. In other words, services need to be able to share tokens with other services. Oftentimes, these tokens can be forged or manipulated to access unauthorized parts of applications and sensitive data.
Token Fraud scenarios include:
    * Post-MFA hijacking
    * Post-Authz hijacking
    * Token Theft
    * Token Forgery
  * Strategies for Auth token usage and issuance within the mesh should be understood. Token forwarding and exchange approaches vary wildly across architectures. Bottom line, a token with many privileges is eventually a target for exploits and breaks the principle of least privilege.

### Principle #5: All sensitive data is encrypted both in transit and at rest.

* Encryption in transit
  * One of the main reasons organizations adopt a service mesh architecture is to achieve Zero Trust. The service mesh should take care of provisioning strong identities to every workload with certificates (e.g. X.509). 
  * However, mesh proxies are normally configured in permissive mode by default, meaning they will accept both mutual TLS and plaintext traffic. This weakens the security posture of the mesh and migration to strict mode should be made immediately.
  * Transport authentication with Mutual TLS alone is not always enough to fully secure traffic, however, as it provides only authentication, not authorization. This means that anyone with a valid certificate can still access a service.
  * ALTS is used for securing Remote Procedure Call (RPC) communications. The ALTS trust model has been tailored for cloud-like containerized applications. Identities are bound to entities instead of to a specific server name or host. This trust model facilitates seamless microservice replication, load balancing, and rescheduling across hosts.

### Principle #6: All events are to be continuously monitored, collected, stored, and analyzed to assess compliance with security policies.

* Monitoring/Alerting/Auditing
  * Service mesh should be integrated out-of-the-box with Prometheus or other cloud native time series databases and monitoring systems. Prometheus collects various traffic-related metrics and provides a rich query language for visualizations and reporting. 
  * At a minimum, the four golden signals should be monitored:
    * Latency: the time it takes to serve a request.
    * Traffic: the total number of requests across the network.
    * Errors: the number of requests that fail.
    * Saturation: the load on your network and servers.
* From an API and service security perspective, sensitive data monitoring, through techniques like Inline Response Analysis, should also be implemented to understand spikes and anomalies within mesh traffic. For example, if a spike in sensitive data emission does not correlate with a spike in normal traffic, there is a high likelihood that an API is emitting more sensitive data than expected.

* Traffic Management 
  * A service mesh connects to a service discovery system so that it knows where all the endpoints are, and which services they belong to. For example, if you’ve installed Istio on a Kubernetes cluster, then Istio automatically detects the services and endpoints in that cluster.
  * In many cases you might want more fine-grained control over what happens to your mesh traffic:
    * Direct a particular percentage of traffic to a new version of a service as part of A/B testing
    * Apply a different load balancing policy to traffic for a particular subset of service instances. 
    * Apply special rules to traffic coming into or out of your mesh, or add an external dependency of your mesh to the service registry. 
  * These scenarios are handled through custom configuration to the mesh traffic management API.

* (Resiliency) Timeouts/Retries
  * A timeout is the amount of time that a mesh proxy should wait for replies from a given service, ensuring that services don’t hang around waiting for replies indefinitely and that calls succeed or fail within a predictable timeframe. 
  * A retry setting specifies the maximum number of times a mesh proxy attempts to connect to a service if the initial call fails.
* (Resiliency) Blue-Green Deployments
  * Blue-green deployment is a technique that reduces downtime and risk by running two identical production environments called Blue and Green.
  * At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live and Green is idle.
* Canary Rollouts
  * Canary deployments are a pattern for rolling out releases to a subset of users or servers. The idea is to first deploy the change to a small subset of servers, test it, and then roll the change out to the rest of the servers. The canary deployment serves as an early warning indicator with less impact on downtime: if the canary deployment fails, the rest of the servers aren’t impacted.

### Principle #7: Policy management and distribution is centralized.

> MeshSecOps drives practitioners to mandate and manage centralized security policy across the entire organization allowing for easy deployment of authorization systems, encryption in transit, identity, network monitoring, analysis, add-on cybersecurity services and more. This makes it cheaper to operate mesh provided services and tooling securely and reliably.

* Policy Enforcement Point
  * Sidecar, waypoint and perimeter proxies work as Policy Enforcement Points (PEPs) to secure communication between clients and servers. PEPs are a component within the mesh that serve as the gatekeeper and "front door" to a digital resource. This allows developers to move request-level policy enforcement out of the application code, trusting instead on the mesh’s assurance that requests that reach the service have been authenticated and authorized for the action that the request is attempting. The mesh can even be configured to pass proof of this to the application.
  * The service mesh tunnels service-to-service communication through the client- and server-side PEPs
  * A PEP parses the policy rules defined in the control plane and converts them into configuration parameters in the data plane module (i.e., the sidecar proxy). These policies may pertain to various functions, such as authentication, authorization, service discovery, traffic management (including load balancing), intelligent routing, blue-green deployments, and canary rollouts. It may also include configuration parameters related to resilience in the service mesh, such as timeout, retry, and circuit-breaking capabilities.
