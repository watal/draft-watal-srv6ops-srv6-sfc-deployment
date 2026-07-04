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

The deployed system integrates SRv6 forwarding, service function management, topology collection, path computation, and flow classification to enable dynamic provisioning of SFC services via a web-based management interface.

This document summarizes the deployment architecture, operational workflow, experience, and lessons learned, and provides guidance for network operators deploying SRv6 SFC services.

--- middle

# Introduction

Segment Routing over IPv6 (SRv6) {{!RFC8986}} enables packet steering through a set of instructions called a segment list.

Service Function Chaining (SFC) {{!RFC7665}} can be implemented using SRv6 to steer traffic through SR-aware service functions.

The architecture of SRv6 SFC with SR-aware functions is described in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

This document does not define any new protocols or protocol extensions.

Instead, it describes the deployment and operational experience of the SRv6 SFC architecture on an academic network.

On-demand instantiation of service function chains requires forwarding, control, management, and application functions to operate as a single coordinated system rather than in isolation.

This document reports on a deployment that integrates these functions on an academic backbone, and summarizes the resulting operational experience.

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
* SRv6 SFC architecture, including the Service Function Manager (SFM), defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Deployment Objectives

* Validate the SRv6 SFC architecture on a real backbone network.
* Demonstrate incremental deployment without modifying the existing IPv6 backbone.
* Evaluate the operational feasibility of on-demand SFC provisioning.
* Evaluate end-to-end service operation across forwarding, control, management, and application planes.

# Deployment Environment

Figure 1 shows the physical deployment environment.

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

The deployment was conducted on the SINET IPv6 backbone, Japan's research and education network interconnecting universities and research institutions.

The backbone provides native IPv6 connectivity among geographically distributed sites.

## Data Centers

Multiple SINET data centers are connected to the backbone.

Three geographically distributed data centers (Okinawa, Kanagawa, and Sapporo) were used to evaluate service chaining across a wide-area network.

Each data center provides computing resources with native IPv6 connectivity to the backbone.

## Video Source Sites

University-operated video source servers were used as traffic sources for the deployed service.

These servers were located at Okinawa, Kanagawa, Chiba, and Ishikawa, with native IPv6 connectivity to the backbone.

## Virtualized Network Function Infrastructure

A virtualized infrastructure was deployed at each selected data center to host SR-aware service functions.
Each site operates OpenStack as the Virtualized Infrastructure Manager (VIM).

# Deployment Architecture

The deployment follows the SRv6 SFC architecture defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

The deployed system is organized into four logical planes: the forwarding plane, control plane, management plane, and application plane.
Each plane is responsible for a distinct aspect of service deployment and operation.

The following subsections describe the role of each plane and its realization in the deployed system.

## Overall Architecture

Figure 2 illustrates the logical architecture of the deployed system.

* The application plane provides the operator interface.
* The control plane performs topology collection, path computation, and SR Policy provisioning.
* The management plane deploys and configures service functions.
* The forwarding plane forwards traffic through SR-aware service functions.

This architecture enables dynamic deployment and operation of SRv6 service function chains while preserving the existing forwarding infrastructure.

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

The forwarding plane consists of the backbone and the SR-aware service functions deployed at geographically distributed data centers.

Traffic is steered through a sequence of service functions using SRv6 segment lists.
Service functions implement the End.AN behavior to process packets and forward them to the next segment in the service chain.

The forwarding infrastructure operates without modifications to the existing backbone routers, enabling incremental deployment of SRv6 SFC services.

The forwarding plane is responsible for packet forwarding and service function execution, while service orchestration and policy decisions are handled by the upper planes.

## Control Plane

The control plane is responsible for topology collection, path computation, and SR Policy provisioning.

A Path Computation Element (PCE) provides TED management, path computation, and SR Policy provisioning.

A BGP daemon performs two functions:

* collecting topology information via BGP-LS, including Service Segment advertisements
* distributing traffic classification rules via BGP Flow Specification

## Management Plane

The management plane is responsible for deploying, configuring, and managing the lifecycle of SR-aware service functions.

The management plane comprises the following three logically distinct functions:

* VNF Manager (VNFM): defined in {{!RFC8568}}, responsible for the life-cycle management of service functions, including issuing instantiation, scaling, and termination requests.

* VIM: defined in {{!RFC8568}}, responsible for controlling and managing the underlying NFVI compute, storage, and network resources, and for fulfilling the life-cycle requests issued by the VNF Manager. Service functions are instantiated on this NFVI, as illustrated in Figure 4.

* Service Function Manager (SFM): defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}, responsible for SRv6-specific service configuration after a service function instance becomes operational, including Service SID assignment and endpoint behavior configuration, as illustrated in Figure 5.

The management plane supports reconfiguration and removal of service functions throughout their operational lifecycle.

## Application Plane

Based on user input, the application plane translates service requirements into intent-based service requests and coordinates the control and management planes to realize them as deployment operations.

The application plane MAY incorporate closed-loop automation functions that operate on telemetry feedback from the control and management planes.

Such mechanisms provide policy refinement and intent optimization based on observed service and network conditions.

These functions are logically separated from the control and management planes and operate at a higher level of abstraction.

# Operational Workflow

This section describes the operational workflow for deploying and activating an SRv6 service function chain.
The workflow begins with an operator service request and concludes with traffic steering through the deployed service functions.

The workflow is illustrated in Figures 4-6.

The following abbreviations are used:

~~~ drawing
Op   = Operator            SFM  = Service Function Manager
App  = Application         VM   = VM/VNF
VNFM = VNF Manager         Ctrl = Controller
VIM  = Virtualized         Src  = Headend
       Infrastructure      Dest = Tailend
       Manager
~~~
{: #fig-wf-legend title="Abbreviations Used in Figures 4-6"}

Figure 4 shows service function instantiation.
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
  |            |Instance OK |            |            |
  |            |<-----------|            |            |
  |            |            |            |            |
~~~
{: #fig-wf-4 title="NF Instantiation"}

Figure 5 shows SRv6-specific service configuration after the service function becomes operational, followed by Service Segment advertisement.

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
  |                 | Service SID OK  |                 | Update TED
  |                 |<----------------|                 |
  |     Deploy      |                 |                 |
  |   Segment OK    |                 |                 |
  |<----------------|                 |                 |
  |                 |                 |                 |
~~~
{: #fig-wf-5 title="Service Segment Configuration"}

Figure 6 shows SFC activation, consisting of path computation, SR Policy provisioning, Flow Specification installation, and traffic steering.

~~~ drawing
 App              Ctrl               Src              Dest
  |                 |                 |                 |
  |   Request SFC   |                 |                 |
  |---------------->|                 |                 |
  |                 | Compute Path    |                 |
  |                 |    Provision    |                 |
  |                 |    SR Policy    |                 |
  |                 |---------------->|                 |
  |                 |                 |                 |
  |                 |  SR Policy OK   |                 |
  |                 |<----------------|                 |
  |     Request     |                 |                 |
  |     SFC OK      |                 |                 |
  |<----------------|                 |                 |
  |     Install     |                 |                 |
  |    Flow Rule    |                 |                 |
  |---------------->|                 |                 |
  |                 |    FlowSpec     |                 |
  |                 |---------------->|                 |
  |                 |                 |   SFC Traffic   |
  |                 |                 |    Steering     |
  |                 |                 |---------------->|
  |                 |   FlowSpec OK   |                 |
  |                 |<----------------|                 |
  |   FlowSpec OK   |                 |                 |
  |<----------------|                 |                 |
  |                 |                 |                 |
~~~
{: #fig-wf-6 title="SFC Activation and Traffic Steering"}


## Service Request

The operational workflow begins when an operator submits a service request through the web-based management interface.

A service request typically includes:

* headend and tailend nodes
* optional traffic classification rules
* the required sequence of service functions
* optional service constraints, such as latency requirements

The application plane translates these service requirements into deployment requests for the control and management planes.

## Network Function Deployment

If one or more requested service functions are not currently available, new service functions are deployed.

Deployment may be triggered either by an explicit operator request or automatically based on operational policies, such as resource utilization or closed-loop service management.

## Service SID Allocation

After service function deployment, a Service SID is assigned to each service function.

Service SID allocation MAY be performed dynamically by the management plane or explicitly specified by the application plane.

See Section 9.1 for operational considerations.

## Topology Collection

Topology information is continuously collected via BGP-LS independently of individual service requests.

Once the service function has completed initialization and health verification, and the Service SID has been configured with the corresponding End.AN behavior, it advertises its Service SID information via BGP-LS.

Until this advertisement is received, the service function is not included in the TED and is therefore not considered during path computation.

## Path Computation

When a service request is received, the control plane computes an SR Policy satisfying the requested service chain based on the current network topology and the available service functions stored in the TED.

## SR Policy Provisioning

As illustrated in Figure 6, the computed SR Policy is provisioned to the SR source node acting as the PCC, using PCEP.
The SR Policy specifies the SRv6 segment list representing the selected service function chain.

## Flow Classification

If traffic classification is requested, BGP Flow Specification rules are installed, as shown in Figure 6, associating each traffic flow with its SR Policy by Color, to ensure that only the selected traffic flows traverse the deployed service function chain.
Installation ordering relative to SR Policy provisioning is discussed in Section 9.3.

## Traffic Steering

Once the SR Policy and Flow Specification rules shown in Figure 6 are installed, the SR source node begins steering matching traffic through the selected service functions.

## Monitoring

Operational status is continuously monitored throughout the service lifecycle using telemetry collected from both the network and infrastructure.

Monitoring data is used to verify service availability and support operational troubleshooting.

In addition, operators MAY verify correct traffic steering using SR path tracing or in-band telemetry mechanisms.

## Service Update and Removal

The deployed system supports updates throughout the service lifecycle.

Typical operations include:

* modifying service chains
* removing service chains
* redeploying failed service functions
* removing service functions

Service updates are performed while maintaining consistency between the forwarding, control, management, and application planes.

# Deployment Experience

This section describes the deployment and operational experience of the SRv6 SFC architecture.

The deployed system supported a remote video production service.

The following subsections describe the deployed architecture, the deployed service, and the resulting operational observations.

## Architecture Deployment

The deployment utilized the deployment environment described in Section 4.

The deployed system implements the following components:

* Application plane: a web-based management interface.
* Control plane: Pola PCE for path computation and SR Policy provisioning, together with GoBGP for BGP-LS topology collection and BGP Flow Specification distribution.
* Management plane: a VNF Manager, OpenStack as the VIM, and Ansible as the SFM.
* Forwarding plane: the existing backbone and distributed SR-aware service functions.

## Service Deployment

Video streams from the video source sites described in Section 4.3 were dynamically steered through SR-aware service functions deployed in the Kanagawa and Sapporo data centers.

The service functions performed video switching, transcoding, and caption insertion before forwarding the processed streams to the production system.

Service chains were created through the web-based management interface.
The distributed service functions were instantiated, their Service SID information was advertised via BGP-LS, and the resulting chains were incorporated into forwarding without any manual configuration of backbone routers.

## Operational Benefits

The deployed system demonstrated several operational benefits.

* Existing backbone routers required no software modifications.
* Service functions were deployed on demand using existing cloud infrastructure.
* SR Policies and Flow Specification rules were automatically generated.
* Operators primarily interacted through the application plane, reducing operational complexity.
* Newly deployed service functions became available for path computation once Service SID advertisement (Section 6.4) was complete.

## Scalability Considerations

This deployment demonstrated that the architecture can scale incrementally by deploying additional SRv6-capable service function nodes without changing the overall control architecture.

Once the application, control, and management components are deployed, additional SR-aware service functions can be instantiated or removed using the VIM's native scaling mechanisms.

These service functions become available for path computation without requiring manual updates to the controller configuration.

# Lessons Learned

During the deployment of the SRv6 SFC system over the backbone, several operational issues and design insights were identified.
This section summarizes key observations obtained from real-world operation.

## Service Verification and Observability

Verifying end-to-end service correctness required more than monitoring SR Policy status or the operational state of service functions.
In the deployed video processing service, it also required application-layer verification, such as comparing input and output video streams, to confirm that traffic had been processed correctly.

Furthermore, because service functions were distributed across multiple VIM domains, effective troubleshooting required correlating network-layer, cloud-layer, and application-layer observations across these domains.

During the deployment, however, these types of operational information were monitored using separate tools, and no unified observability framework was available.
As a result, diagnosing service failures required manual cross-referencing between SR Policy state, service function status, and application-layer observations, making troubleshooting time-consuming and operationally complex.

This experience led to the operational recommendations in Section 9.5, which describe a unified multi-layer observability framework.

## Latency-Aware Service Function Placement

This section addresses the placement of SR-aware service functions.
For control-plane component placement, see Section 9.2.

Because data centers are geographically distributed, inter-site latency has a significant impact on service performance.

For latency-sensitive applications such as real-time video processing, cumulative path latency across multiple sites is an important consideration for service function placement.

In this deployment, VNF placement was determined manually.
The application plane SHOULD integrate latency measurement and topology-aware placement optimization to automate this process.

# Operational Considerations

SRv6 SFC deployments require coordination among the control, management, and application planes to ensure consistent service operations.

## Service SID Allocation

Building on the Service SID allocation described in Section 6.3, allocation MUST ensure uniqueness within the SR domain.

A centralized allocation mechanism SHOULD be used, where the management plane coordinates with the control plane (TED) to identify available SID space prior to assignment, thereby preventing address collisions and simplifying multi-site service deployment.

## Controller Considerations

Controller placement and architecture have a significant impact on system performance and scalability due to frequent interactions between:

* the control plane and the application plane, which issues service requests and receives status updates
* the control plane and the management plane, which coordinates Service SID allocation and topology updates
* the control plane and SR source nodes, which receive SR Policy and Flow Specification provisioning

To reduce control-plane latency and operational overhead, controller placement SHOULD minimize latency between control components and service endpoints while considering network topology constraints.

In the described deployment, the application plane and associated management components were co-located at a single data center to reduce inter-domain communication latency.

A logically centralized controller architecture SHOULD be used to ensure consistent path computation and policy enforcement across the SR domain.

For large-scale deployments, a hierarchical controller model MAY be used to improve scalability while preserving global policy consistency.

## Flow Classification and SR Policy Synchronization

When an SR Policy is intended to steer only selected traffic (e.g., a specific flow or VPN), it SHOULD be associated with a Color so that it is applied only to the intended traffic.

The corresponding BGP Flow Specification rules SHOULD be installed only after the SR Policy has been successfully provisioned and is operational.

Otherwise, traffic MAY be classified to an SR Policy that is not yet operational, resulting in transient traffic misclassification or blackholing.

## Failure Recovery

Failure recovery follows the mechanisms defined in {{!I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

In operational deployments, fast reroute at the forwarding plane can maintain connectivity, but service-level state consistency is not guaranteed during failover events.

Therefore, service functions MUST be designed to handle potential state inconsistencies (e.g., buffering, re-synchronization, or idempotent processing).

## Observability

Based on the deployment experience described in Section 8.1, a multi-layer observability framework SHOULD include:

* SRv6 topology and SR Policy state
* Flow classification and traffic steering behavior
* Service function health and availability
* Virtual infrastructure resource utilization

This SHOULD be supported by a unified telemetry framework spanning network and cloud domains, correlating the layers above to enable rapid fault detection and diagnosis.

For content-modifying services (e.g., video processing), application-layer verification MAY also be required to compare input and output streams.

## Operational Sequencing

Automated orchestration SHOULD ensure correct sequencing of:

1. Service function instantiation
2. Service SID assignment and service configuration
3. Readiness verification
4. BGP-LS advertisement
5. SR Policy provisioning
6. Flow Specification installation

Out-of-order execution MAY result in transient traffic disruption.

Operators SHOULD implement consistency checks and readiness verification before enabling traffic steering.

# Security Considerations

The deployment described in this document relies on existing security mechanisms provided by SRv6 and associated control and management protocols, including BGP-LS, PCEP, BGP Flow Specification, and NFV management interfaces.

Management interfaces SHOULD be protected using mutually authenticated secure transport protocols.

Operators MUST ensure that SR Policies, topology information, traffic classification rules, and Service SID information are protected against unauthorized modification or injection, using appropriate authentication, authorization, and integrity protection mechanisms.

Compromise of these components may result in incorrect traffic steering, service misbehavior, or service disruption.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}
The authors would like to acknowledge the reviews and input from Mitsuru Maruyama, Katsuhiro Sebayashi, Taisei Tanabe, Ryuta Futami, and Takashi Kurimoto.
This work was partially supported by JST CRONOS No. JPMJCS24N9.
