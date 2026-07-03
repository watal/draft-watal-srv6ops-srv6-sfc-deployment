---
title: "SRv6 Service Function Chaining Deployment"
abbrev: "SRv6 SFC Deployment"
docname: draft-watal-srv6ops-srv6-sfc-deployment-latest
category: info
submissionType: IETF

ipr: trust200902
area: Operations and Management
workgroup: srv6ops
keyword:
  - SRv6
  - Service Function Chaining
  - Operations
  - Deployment
  - IPv6
stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: W. Mishima
    name: Wataru Mishima
    organization: Kanagawa Institute of Technology
    email: mishima@nw.kanagawa-it.ac.jp
    country: Japan
 -
    ins: Y. Fukagawa
    name: Yuta Fukagawa
    organization: Kanagawa Institute of Technology
    email: fukagawa@nw.kanagawa-it.ac.jp
    country: Japan

normative:

informative:

--- abstract

This document describes the deployment and operational experience of the SRv6 Service Function Chaining (SFC) architecture defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}} on an academic IPv6 backbone network.

The deployed system integrates SRv6 forwarding, service function lifecycle management, topology collection, path computation, and flow classification to enable dynamic provisioning of SFC services via a web-based management interface.

This document summarizes the deployment architecture, operational workflow, deployment experience, lessons learned, and operational considerations, and provides operational guidance for network operators deploying SRv6 SFC services.

--- middle

# Introduction

Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.

Service Function Chaining (SFC) {{!RFC7665}} can be implemented using SRv6 by steering traffic through a sequence of SR-aware service functions.

The architecture of SRv6 SFC with SR-aware functions is described in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

This document does not define any new protocols or protocol extensions.

Instead, it describes the deployment and operational experience of the architecture on an academic network.

The deployment integrates SRv6 forwarding, service function lifecycle management, topology collection, path computation, and flow classification to enable on-demand instantiation of service function chains based on operator requests.

Based on this deployment experience, this document summarizes the deployment architecture, operational workflow, deployment experience, lessons learned, and operational considerations, and provides operational guidance for operators deploying SRv6 SFC services.

# Terminology

## Terminology Defined in Related RFCs and Internet-Drafts
The following terms are used in this document as defined in the related RFCs and Internet-Drafts:

* SR and Segment Identifier (SID) defined in {{!RFC8402}}.
* SRv6 defined in {{!RFC8986}}.
* Headend, Color, and SR Policy defined in {{!RFC9256}}.
* SFC, Service Function, and Service Function Chain defined in {{!RFC7665}}.
* Path Computation Client (PCC), Path Computation Element (PCE), and Traffic Engineering Database (TED) defined in {{!RFC5440}}.
* BGP Flow Specification defined in {{!RFC8955}}.
* NFV Infrastructure (NFVI), Virtualized Infrastructure Manager (VIM), and Virtualized Network Function Manager (VNFM) defined in {{!RFC8568}}.
* Service Segment and End.AN defined in {{!I-D.draft-ietf-spring-sr-service-programming}}.
* SRv6 SFC architecture defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Deployment Objectives

* Validate the SRv6 SFC architecture on a real backbone network.
* Demonstrate incremental deployment without modifying the existing IPv6 backbone.
* Evaluate the operational feasibility of on-demand SFC provisioning.
* Evaluate the operational integration of forwarding, control, management, and application planes.

# Deployment Environment

Figure 1 illustrates the physical deployment environment used throughout this document.


~~~ drawing
                         SINET IPv6 Backbone
       +-----------------+-----------------+-----------------+
       |                 |                 |                 |
 +------------+    +------------+    +------------+         ...
 |  SINET DC  |    |  SINET DC  |    |  SINET DC  |
 +------------+    +------------+    +------------+
       |                 |                 |
 +------------+    +------------+    +------------+
 | OpenStack  |    | OpenStack  |    | OpenStack  |
 +------------+    +------------+    +------------+
       |                 |                 |
 +------------+    +------------+    +------------+
 |  Service   |    |  Service   |    |  Service   |
 |  Function  |    |  Function  |    |  Function  |
 +------------+    +------------+    +------------+
~~~
{: #fig-deployment-environment title="Deployment Environment"}

## IPv6 Backbone

The deployment was conducted on the SINET academic IPv6 backbone network, interconnecting universities and research institutions across Japan.

The backbone provides native IPv6 connectivity among geographically distributed sites.

The SRv6 SFC deployment was designed to operate without requiring modifications to the existing backbone forwarding infrastructure, enabling incremental deployment alongside academic network services.

## Data Centers

Multiple SINET data centers connected to the backbone were used to host service functions.

Three geographically distributed data centers (Okinawa, Kanagawa, and Sapporo) were selected to evaluate service chaining across a wide-area network.

Each data center provides computing resources with native IPv6 connectivity to the backbone.

## Virtualized Network Function Infrastructure

A virtualized infrastructure was deployed at each selected data center to host SR-aware service functions.

Each site operates OpenStack as the Virtualized Infrastructure Manager (VIM) {{!RFC8568}}, which manages compute, storage, and network resources of the NFV Infrastructure (NFVI) used to host SR-aware service functions.
The VNF Manager (VNFM) issues life-cycle management requests, such as instantiation and scaling, to the VIM, which provisions the required NFVI resources.

Service functions are instantiated on demand.

After instance creation, SRv6 endpoint behaviors and service-specific parameters are configured automatically.

# Deployment Architecture

The deployment follows the SRv6 SFC architecture defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

The deployed system is organized into four logical planes: the forwarding plane, control plane, management plane, and application plane. Each plane is responsible for a distinct aspect of service deployment and operation.

The following subsections describe the role of each plane and its realization in the deployed system.

## Overall Architecture

Figure 2 illustrates the logical architecture of the deployed system.
The architecture is organized into four logical planes:

* The application plane provides the operator interface.
* The control plane performs topology collection, path computation, and SR Policy provisioning.
* The management plane deploys and configures service functions.
* The forwarding plane forwards traffic through SR-aware service functions.


Together, these planes enable dynamic deployment and operation of SRv6 service function chains while preserving the existing forwarding infrastructure.

~~~ drawing
                     +----------------------+
                     |  Application Plane   |
                     +------+------+--------+
                            |      |
                            |      |
           +----------------+      +-----------------+
           |                                         |
+----------v-----------+                   +---------v---------+
|    Control Plane     |                   | Management Plane  |
+----------+-----------+                   +-------------------+
           |                                         |
           +----------------+      +-----------------+
                            |      |
                     +------v------v------+
                     |  Forwarding Plane  |
                     +--------------------+
~~~
{: #fig-plane title="Logical Architecture of the Deployed System"}

## Forwarding Plane

The forwarding plane consists of the backbone network and the SR-aware service functions deployed at geographically distributed data centers.

Traffic is steered through a sequence of service functions using SRv6 segment lists.
Service functions implement the End.AN behavior to process packets and forward them to the next segment in the service chain.

The forwarding infrastructure operates without modifications to the existing backbone routers, enabling incremental deployment of SRv6 SFC services.

The forwarding plane is responsible for packet forwarding and service function execution, while service orchestration and policy decisions are handled by the upper planes.

## Control Plane

The control plane is responsible for topology collection, path computation, and SR Policy provisioning.

Topology information collected via BGP-LS is used to populate the Traffic Engineering Database (TED).
Based on service requests and the current network topology, the Path Computation Element (PCE) computes service paths and provisions the corresponding SR Policies.

The deployed implementation uses Pola PCE, an open-source PCE implementation, for TED management, path computation, and SR Policy provisioning.

GoBGP, an open-source BGP implementation, is used to receive BGP-LS advertisements carrying topology information and Service SID information, and to distribute BGP Flow Specification rules that associate traffic flows with the corresponding SR Policies.

## Management Plane

The management plane is responsible for deploying, configuring, and managing the lifecycle of SR-aware service functions.

Consistent with the roles defined in {{!RFC8568}}, the management plane comprises two logically distinct functions:

* VNF Manager (VNFM): responsible for the life-cycle management of service functions, including issuing instantiation, scaling, and termination requests.
* Virtualized Infrastructure Manager (VIM): responsible for controlling and managing the underlying NFVI compute, storage, and network resources, and for fulfilling the life-cycle requests issued by the VNF Manager.

Service functions are instantiated on virtualized infrastructures managed by OpenStack, which acts as the VIM.
After a service function instance becomes operational, the management plane configures SRv6 endpoint behaviors, Service SIDs, and other service-specific parameters required for traffic processing.

The management plane supports reconfiguration and removal of service functions throughout their operational lifecycle.

## Application Plane

Based on user input, the application plane translates service requirements into intent-based service requests and coordinates the control and management planes to realize them as deployment operations.

The application plane MAY incorporate closed-loop automation functions that operate on telemetry feedback from the control and management planes.

Such mechanisms provide policy refinement and intent optimization based on observed service and network conditions.

These functions are logically separated from the control and management planes and operate at a higher level of abstraction.

# Operational Workflow

This section describes the operational workflow for deploying and activating an SRv6 service function chain.
The workflow begins with an operator service request and concludes with traffic steering through the deployed service functions.

To keep each diagram within a readable line width, the workflow is presented as three figures: Figure 3a covers network function instantiation, Figure 3b covers service segment configuration and topology advertisement, and Figure 3c covers SFC activation and traffic steering.
The Service Function Manager (SFM) shown in Figure 3b is defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.
The following abbreviations are used:

~~~
Op   = Operator            SFM  = Service Function Manager
App  = Application         VM   = VM/VNF
VNFM = VNF Manager         Ctrl = Controller
VIM  = Virtualized         HE   = Headend
       Infrastructure      TE   = Tailend
       Manager
~~~
{: #fig-wf-legend title="Abbreviations Used in Figures 3a-3c"}

Figure 3a shows service function instantiation.
Consistent with {{!RFC8568}}, the VNF Manager issues the life-cycle request and the VIM allocates and provisions the underlying NFVI resources.

~~~ drawing
 Op           App         VNFM          VIM          VM
  |            |            |            |            |
  |  Service   |            |            |            |
  |  Request   |            |            |            |
  |----------->|            |            |            |
  |            |  Deploy    |            |            |
  |            |    NF      |            |            |
  |            |----------->|            |            |
  |            |            |Instantiate |            |
  |            |            |----------->|            |
  |            |            |            |Instantiate |
  |            |            |            |----------->|
  |            |            |Instance OK |            |
  |            |            |<-----------|            |
  |            |            |            |            |
~~~
{: #fig-wf-3a title="Figure 3a: NF Instantiation"}

Figure 3b shows the configuration of SRv6-specific service parameters after the service function instance becomes operational, and the subsequent advertisement of the corresponding Service Segment.

~~~ drawing
 App               SFM               VM               Ctrl
  |                 |                 |                 |
  |     Deploy      |                 |                 |
  |     Segment     |                 |                 |
  |---------------->|                 |                 |
  |                 |    Configure    |                 |
  |                 |   Service SID   |                 |
  |                 |---------------->|                 |
  |                 |                 |    Advertise    |
  |                 |                 |     Segment     |
  |                 |                 |---------------->|
  |                 |                 |                 | Update TED
  |                 |                 |                 |
~~~
{: #fig-wf-3b title="Figure 3b: Service Segment Configuration"}

Figure 3c shows SFC activation, consisting of path computation, SR Policy provisioning, Flow Specification installation, traffic steering, and monitoring.

~~~ drawing
 App              Ctrl               HE                TE
  |                 |                 |                 |
  |   Request SFC   |                 |                 |
  |---------------->|                 |                 |
  |                 | Compute Path    |                 |
  |                 |    Provision    |                 |
  |                 |    SR Policy    |                 |
  |                 |---------------->|                 |
  |     Install     |                 |                 |
  |    Flow Rule    |                 |                 |
  |---------------->|                 |                 |
  |                 |    FlowSpec     |                 |
  |                 |---------------->|                 |
  |                 |                 |   SFC Traffic   |
  |                 |                 |    Steering     |
  |                 |                 |---------------->|
  |                 |   Monitoring    |                 |
  |                 |     Status      |                 |
  |<----------------------------------------------------|
  |                 |                 |                 |
~~~
{: #fig-wf-3c title="Figure 3c: SFC Activation and Traffic Steering"}


## Service Request

The operational workflow begins when an operator submits a service request through the web-based management interface.

A service request typically includes:

* ingress and egress nodes;
* optional traffic classification rules;
* the required sequence of service functions; and
* optional service constraints, such as latency requirements.

The application plane translates these service requirements into deployment requests for the control and management planes.

## Network Function Deployment

If one or more requested service functions are not currently available, new service functions are deployed.

Deployment may be triggered either by an explicit operator request or automatically based on operational policies, such as resource utilization or closed-loop service management.

The VNF Manager requests the VIM to allocate the necessary compute, storage, and network resources and to instantiate the required virtual machines or virtual network functions (VNFs).
The VIM provisions the corresponding NFVI resources and returns the instantiation result to the VNF Manager.
The deployment phase completes after the service functions become operational.

## Service SID Allocation

After service function deployment, a Service SID is assigned to each service function.

Service SID allocation MAY be performed dynamically by the management plane or explicitly specified by the application.

When Service SIDs are allocated dynamically, the management plane MUST ensure that allocated Service SIDs are unique within the SR domain.
The TED MAY be referenced to identify available SID space and avoid address conflicts.

## Topology Collection

Topology information is continuously collected via BGP-LS independently of individual service requests.

After a deployed service function is configured with an End.AN behavior and has completed all required initialization and health verification procedures, the corresponding node advertises its Service SID information via BGP-LS.

This ensures that only operationally ready service functions are included in topology dissemination and path computation.

The control plane updates the TED accordingly, allowing newly deployed service functions to be considered during subsequent path computations.

## Path Computation

When a service request is received, the control plane computes an SR Policy satisfying the requested service chain based on the current network topology and the available service functions stored in the TED.

Path computation considers both network topology and the locations of deployed service functions.

## SR Policy Provisioning

After path computation, the computed SR Policy is provisioned to the SR source node acting as the PCC using PCEP.

The installed SR Policy specifies the SRv6 segment list representing the selected service function chain.

## Flow Classification

If traffic classification is requested, the control plane installs BGP Flow Specification rules on the SR source node.

Each Flow Specification rule associates a traffic flow with the corresponding SR Policy by referencing its Color value.

This mechanism enables only the selected traffic flows to traverse the deployed service function chain.

## Traffic Steering

Once both the SR Policy and the corresponding Flow Specification rules have been installed, the SR source node begins steering matching traffic through the selected sequence of service functions.

Traffic traverses each service function according to the SRv6 segment list until reaching the final destination.

## Monitoring

Operational status is continuously monitored throughout the service lifecycle.

Typical monitoring targets include:

* service function availability
* SR Policy status
* BGP session status
* Virtualized Infrastructure Manager (VIM) status
* service function health

In addition, operators MAY verify correct packet forwarding using SR path tracing or in-band telemetry mechanisms.

## Service Update and Removal

The deployed system supports updates throughout the service lifecycle.

Typical operations include:

* adding service functions to an existing service chain
* removing service functions
* redeploying failed service functions
* updating SR Policies
* removing service chains

Service updates are performed while maintaining consistency between the forwarding, control, management, and application planes.

# Deployment Experience

This section describes the deployment and operational experience of the SRv6 SFC architecture on the backbone.

The deployment used the infrastructure described in Section 4 to support a remote video production service requiring dynamic deployment of distributed service functions.

The following subsections describe the infrastructure, the architecture, the service, and the operational observations.

## Infrastructure Deployment

The deployment utilized the backbone described in Section 4.1 and the distributed OpenStack-based data centers described in Section 4.2.

## Architecture Deployment

The deployed system implemented the four-plane architecture described in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

The application, control, management, and forwarding planes were realized as follows.

The application plane was implemented as a web-based management interface through which operators requested service deployments.

The control plane consisted of Pola PCE for topology management, path computation, and SR Policy provisioning, together with GoBGP for BGP-LS topology collection and BGP Flow Specification distribution.

Within the management plane, a VNF Manager component issued life-cycle requests to OpenStack, which served as the VIM responsible for allocating compute, storage, and network resources.
Ansible was used for the automatic configuration of SRv6 endpoint behaviors after instantiation, including End.AN, Service SIDs, and other service-specific parameters.

The forwarding plane consisted of the backbone and SR-aware service functions deployed at the distributed data centers.

## Service Deployment

The deployed system was evaluated using a remote video production service.

Video streams captured at production sites in Okinawa, Kanagawa, Chiba, and Ishikawa were dynamically steered through a sequence of SR-aware service functions deployed in the Kanagawa and Sapporo data centers.

The service functions performed video switching, transcoding, and caption insertion before forwarding the processed streams to the production system.

Service chains were created through the web-based management interface without requiring manual router configuration.

Distributed service functions were instantiated, and their Service SID information was advertised via BGP-LS and incorporated into service chains without manual configuration of backbone routers.

## Operational Benefits

The deployment demonstrated several operational benefits.

* Existing backbone routers required no software modifications.
* Service functions were deployed on demand using existing cloud infrastructure.
* SR Policies and Flow Specification rules were automatically generated.
* Operators primarily interacted through the application plane, reducing operational complexity.
* Newly deployed service functions became available for path computation after their Service SID information was advertised via BGP-LS.

## Scalability Considerations

Additional service function nodes were added by connecting new OpenStack sites to the backbone.

Service SID information is distributed via BGP-LS, enabling newly deployed service functions to be used in path computation without manual controller reconfiguration.

The architecture supported scaling by increasing the number of service function nodes and data centers while maintaining centralized path computation and service orchestration.

# Lessons Learned

During the deployment of the SRv6 SFC system over the backbone, several operational issues and design insights were identified.
This section summarizes key observations obtained from real-world operation.

## Service SID Allocation and Address Space Management

A centralized Service SID allocation mechanism was required to avoid address conflicts across distributed service functions.

In the implemented system, the management plane interacts with the control plane by referencing the TED to identify available SID space before assigning Service SIDs.

This approach ensured uniqueness of Service SIDs within the SR domain and reduced the risk of inconsistent state across distributed sites.

## Service Verification and Observability

Operators require end-to-end visibility to verify that SRv6 SFC services are deployed and operating as intended.

SRv6 Path Tracing and on-path telemetry (e.g., flow-based or in-band telemetry) can be used to confirm that traffic traverses the expected sequence of service functions according to the installed SRv6 policies.

In addition to verifying that traffic follows the expected path, service-level verification is necessary to confirm that service functions process traffic as intended.
Achieving this requires integration of both network-level and cloud-level telemetry.

Network telemetry includes SR Policy status, BGP session state, and SRv6 forwarding behavior, while cloud telemetry includes service function instance status and resource utilization across multiple VIM domains.

A unified observability framework is RECOMMENDED to correlate service-level behavior with the underlying network and infrastructure state, enabling rapid fault identification and end-to-end service verification.

## Latency-Aware Service Function Placement

Because data centers are geographically distributed, inter-site latency has a significant impact on service performance.

For latency-sensitive applications such as real-time video processing, cumulative path latency across multiple sites is an important consideration for service function placement.

In this deployment, VNF placement was determined manually.
However, it is recommended that the application plane integrate latency measurement and topology-aware placement optimization to automate this process.

# Operational Considerations

## Service SID Allocation

As described in Section 6.3, Service SID allocation MUST ensure uniqueness within the SR domain.
A centralized allocation mechanism SHOULD be used, where the management plane coordinates with the control plane (TED) to identify available SID space prior to assignment, thereby preventing address collisions and simplifying multi-site service deployment.

## Controller Considerations

Controller placement and architecture have a significant impact on system performance and scalability due to frequent interactions between:

* application plane and management plane
* VNF Manager and Virtualized Infrastructure Manager (VIM) within the management plane
* control plane and SR Headend nodes

To reduce control-plane latency and operational overhead, controller placement SHOULD minimize latency between control components and service endpoints while considering network topology constraints.

In the described deployment, the application plane and associated management components were co-located at a single data center to reduce inter-domain communication latency.

A logically centralized controller architecture SHOULD be used to ensure consistent path computation and policy enforcement across the SR domain.

For large-scale deployments, a hierarchical controller model MAY be used to improve scalability while preserving global policy consistency.

## Flow Classification and SR Policy Synchronization

Flow classification and SR Policy provisioning are tightly coupled in operational behavior.

Inconsistent timing between BGP Flow Specification installation and SR Policy activation MAY result in transient traffic misclassification or blackholing.

Implementations SHOULD ensure that SR Policy provisioning and Flow Specification installation are performed in a coordinated manner, such that traffic steering is only enabled after both components are fully operational.

## Failure Recovery

Failure recovery follows the mechanisms defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

When a network function becomes unavailable, its associated Service SID is withdrawn from topology information so that it is no longer considered during subsequent path computation.

If an Anycast SID {{!RFC8402}} is used, traffic MAY be redirected to alternative service instances.
If no alternative exists, packets are dropped and an ICMPv6 Destination Unreachable message is generated.

Fast reroute mechanisms operate at the forwarding plane and MAY maintain connectivity; however, service state consistency is not guaranteed during rerouting events.

Therefore, service functions MUST implement mechanisms to handle potential state inconsistencies, such as buffering, re-synchronization, or idempotent processing.

## Observability

A multi-layer observability framework SHOULD include:

* SRv6 topology and SR Policy state
* Flow classification and traffic steering behavior
* Service function health and availability
* Virtual infrastructure resource utilization

This SHOULD be supported by a unified telemetry framework across both network and cloud domains to ensure consistency.

Correlation between SR Policy state, flow classification, and service function health is essential for rapid fault detection and diagnosis.

In service scenarios involving content modification (e.g., video processing), application-layer verification MAY also be required to compare input and output streams.

## Operational Sequencing

Automated orchestration SHOULD ensure correct sequencing of:

1. Service function instantiation
2. Service SID assignment
3. BGP-LS advertisement (after service readiness verification)
4. SR Policy provisioning
5. Flow Specification installation

Out-of-order execution MAY result in transient traffic disruption.

Operators SHOULD implement consistency checks and readiness verification before enabling traffic steering.

# Security Considerations

The deployment described in this document relies on existing security mechanisms provided by SRv6 and the associated control and management protocols.

Operators SHOULD ensure that controller communications, management interfaces, and service provisioning mechanisms are appropriately authenticated and protected.

The security considerations discussed in the referenced specifications remain applicable to this deployment.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the reviews and input from Mitsuru Maruyama, Katsuhiro Sebayashi, Taisei Tanabe, Ryuta Futami, and Takashi Kurimoto.
This work was partially supported by JST CRONOS No. JPMJCS24N9.
