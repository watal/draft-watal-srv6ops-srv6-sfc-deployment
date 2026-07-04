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
  RFC2119:
  RFC4655:
  RFC5440:
  RFC8174:
  RFC8231:
  RFC8281:
  RFC8402:
  RFC8568:
  RFC8986:
  RFC9256:
  I-D.draft-watal-spring-srv6-sfc-sr-aware-functions:
    target: https://datatracker.ietf.org/doc/html/draft-watal-spring-srv6-sfc-sr-aware-functions-05
    title: SRv6 SFC Architecture with SR-aware Functions
    author:
      -
        ins: W. Mishima
        name: Wataru Mishima
      -
        ins: Y. Fukagawa
        name: Yuta Fukagawa
    date: 2026-07-04
    seriesinfo:
      Internet-Draft: draft-watal-spring-srv6-sfc-sr-aware-functions-05

informative:
  RFC7426:
  RFC7665:
  RFC8664:
  RFC8955:
  RFC9552:
  RFC9862:
  I-D.draft-ietf-spring-sr-service-programming:
  I-D.draft-ietf-idr-bgp-ls-sr-service-segments:

--- abstract

This document describes the deployment and operational experience of the SRv6 Service Function Chaining (SFC) architecture defined in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}} on an academic IPv6 backbone network.

The deployed system integrates SRv6 forwarding, service function management, topology collection, path computation, and flow classification to enable dynamic provisioning of SFC services via a web-based management interface.

This document summarizes the deployment architecture, operational workflow, experience, and lessons learned, and provides guidance for network operators deploying SRv6 SFC services.

--- middle

# Introduction

Segment Routing over IPv6 (SRv6) {{RFC8986}} enables packet steering through a set of instructions called a segment list.

Service Function Chaining (SFC) {{RFC7665}} can be implemented using SRv6 to steer traffic through SR-aware service functions.

The architecture of SRv6 SFC with SR-aware functions is described in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

This document does not define any new protocols or protocol extensions.
It documents the deployment and operational experience of the SRv6 SFC architecture on an academic network.

On-demand instantiation of service function chains requires forwarding, control, management, and application planes to operate as a single coordinated system.

This document reports on a deployment that integrates these functions on an academic backbone, and summarizes the resulting operational experience.

# Terminology

## Terminology Defined in Related RFCs and Internet-Drafts
The following terms are used in this document as defined in the related RFCs and Internet-Drafts:

* SR and Segment Identifier (SID) defined in {{RFC8402}}.
* SRv6 defined in {{RFC8986}}.
* Headend, Color, and SR Policy defined in {{RFC9256}}.
* SFC, Service Function, and Service Function Chain defined in {{RFC7665}}.
* Path Computation Client (PCC) and Path Computation Element (PCE) are defined in {{RFC4655}} and {{RFC5440}}, respectively.
* PCEP extensions for SR and SR Policy are defined in {{RFC8664}} and {{RFC9256}}, with additional SR-related extensions specified in {{RFC9862}}.
* BGP-LS defined in {{RFC9552}}.
* BGP Flow Specification defined in {{RFC8955}}.
* Forwarding Plane, Control Plane, Management Plane, Application Plane defined in {{RFC7426}}.
* NFV Infrastructure (NFVI), Virtualized Infrastructure Manager (VIM), and Virtualized Network Function Manager (VNFM) defined in {{RFC8568}}.
* Service Segment described in {{I-D.draft-ietf-spring-sr-service-programming}}.
* SRv6 SFC architecture, including the Service Function Manager (SFM) and End.AN, described in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Deployment Objectives

* Validate the SRv6 SFC architecture on a backbone network in practical use.
* Demonstrate incremental deployment without modifying existing transit backbone routers.
* Evaluate the operational feasibility of on-demand SFC provisioning.
* Evaluate end-to-end service operation across forwarding, control, management, and application planes.

# Deployment Environment

Figure 1 shows the physical deployment environment.

~~~ drawing
    +-----------------------+
    |Video Source Sites (x4)|
    +-----------+-----------+
                |        SINET IPv6 Backbone
       +--------+--------+-----------------+-----------------+
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

The deployment was conducted on SINET, Japan's research and education network interconnecting universities and research institutions.

SINET provides native IPv6 connectivity among geographically distributed sites.

## Data Centers

Three geographically distributed data centers in Sapporo, Kanagawa, and Okinawa were selected to evaluate service chaining over a wide-area network.
Each data center provides NFV Infrastructure (NFVI).

## Video Source Sites

University-operated video source servers were located at Kanagawa, Chiba, Ishikawa, and Okinawa.

## NFV Infrastructure and Virtualized Infrastructure Manager

A virtualized infrastructure was deployed at each selected data center to host SR-aware service functions.
Each site operates OpenStack as the Virtualized Infrastructure Manager (VIM).

# Deployment Architecture

The deployment follows the SRv6 SFC architecture defined in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

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

The forwarding infrastructure operates on existing backbone routers without requiring modifications, enabling incremental deployment of SRv6 SFC services.

The forwarding plane is responsible for packet forwarding and service function execution, while service orchestration and policy decisions are handled by the upper planes.

## Control Plane

The control plane is responsible for topology collection, path computation, and SR Policy provisioning.

A Path Computation Element (PCE) uses a Traffic Engineering Database (TED) for topology and resource information.

A BGP daemon performs two functions:

* collecting topology information via BGP-LS, including Service Segment advertisements
* distributing traffic classification rules via BGP Flow Specification

## Management Plane

The management plane is responsible for deploying, configuring, and managing the lifecycle of SR-aware service functions.

The management plane consists of the following three logically distinct functions:

* Virtualized Network Function Manager (VNFM): defined in {{RFC8568}}, responsible for the lifecycle management of service functions, including issuing instantiation, scaling, and termination requests.

* VIM: defined in {{RFC8568}}, responsible for controlling and managing the underlying NFVI compute, storage, and network resources, and for fulfilling the lifecycle requests issued by the VNFM. Service functions are instantiated on this NFVI, as illustrated in Figure 4.

* Service Function Manager (SFM): defined in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}, responsible for SRv6-specific service configuration after a service function instance becomes operational, including Service SID assignment and endpoint behavior configuration, as illustrated in Figure 5.

The management plane supports reconfiguration and removal of service functions throughout their operational lifecycle.

## Application Plane

The application plane consists of an operator-facing web-based management interface together with the service orchestration logic.
Based on operator requests, the application plane translates service requirements into intent-based service requests and coordinates the control and management planes to realize them as deployment operations.

The application plane MAY include closed-loop automation functions that operate on telemetry feedback from the control and management planes.

Such mechanisms provide policy refinement and intent optimization based on observed service and network conditions.

These functions are logically separated from the control and management planes and operate at a higher level of abstraction.

# Operational Workflow

This section describes the operational workflow for deploying and activating an SRv6 service function chain.
The workflow begins with an operator service request and concludes with traffic steering through the deployed service functions.

The workflow is illustrated in Figures 4-6.

The following abbreviations are used:

~~~ drawing
Op   = Operator            SFM  = Service Function Manager
App  = Application         VNF  = Virtualized Network Function
VNFM = VNF Manager         Ctrl = Controller
VIM  = Virtualized         Src  = Headend
       Infrastructure      Dest = Tailend
       Manager
~~~
{: #fig-wf-legend title="Abbreviations Used in Figures 4-6"}

In Figures 5 and 6, Ctrl represents the control-plane components described in Section 5.3 (the PCE and the BGP daemon) collectively.
The specific component responsible for each message is identified by the corresponding function (e.g., topology/segment advertisement is handled by the BGP daemon, while path computation and SR Policy provisioning are handled by the PCE).

Figure 4 shows service function instantiation.
Consistent with {{RFC8568}}, the VNFM issues the lifecycle request and the VIM allocates and provisions the underlying NFVI resources.

~~~ drawing
 Op           App         VNFM          VIM           VNF
  |            |            |            |             |
  |  Service   |            |            |             |
  |  Request   |            |            |             |
  |----------->|            |            |             |
  |            |   Deploy   |            |             |
  |            |    VNF     |            |             |
  |            |----------->|            |             |
  |            |            |Instantiate |             |
  |            |            |----------->|             |
  |            |            |            |Instantiate  |
  |            |            |            |------------>|
  |            |            |            |Instantiated |
  |            |            |            |<------------|
  |            |            |Instance OK |             |
  |            |            |<-----------|             |
  |            |Instance OK |            |             |
  |            |<-----------|            |             |
  |            |            |            |             |
~~~
{: #fig-wf-4 title="NF Instantiation"}

Figure 5 shows SRv6-specific service configuration after the service function becomes operational, followed by Service Segment advertisement.

~~~ drawing
 App               SFM               VNF              Ctrl
  |                 |                 |                 |
  |     Deploy      |                 |                 |
  |     Segment     |                 |                 |
  |---------------->|                 |                 |
  |                 |    Configure    |                 |
  |                 |   Service SID   |                 |
  |                 |---------------->|                 |
  |                 |                 |    Advertise    |
  |                 |                 |   Service SID   |
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
  |     Request     |                 |                 |
  |   Path Compute  |                 |                 |
  |---------------->|                 |                 |
  |                 | Compute Path    |                 |
  |                 |                 |                 |
  |                 |    Provision    |                 |
  |                 |    SR Policy    |                 |
  |                 |---------------->|                 |
  |                 |                 |                 |
  |                 |  SR Policy OK   |                 |
  |                 |<----------------|                 |
  |   SR Policy OK  |                 |                 |
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

After service function deployment and service function initialization, Service SID allocation is performed by the management plane before BGP-LS advertisement, as described in Section 9.1.

## Topology Collection

Topology information is continuously collected via BGP-LS independently of individual service requests.

Once the service function has completed initialization and health verification, and the Service SID has been configured with the corresponding End.AN behavior, it advertises its Service SID information via the BGP-LS extension defined in {{I-D.draft-ietf-idr-bgp-ls-sr-service-segments}}, which is currently under standardization.

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

* modifying service function chains
* removing service function chains
* redeploying failed service functions
* removing service functions

Service updates are performed while maintaining consistency between the forwarding, control, management, and application planes.

# Deployment Experience

This section describes the deployment and operational experience of the SRv6 SFC architecture.

The deployed system supported a remote video production service.

The following subsections describe the deployed architecture, the deployed service, and the resulting operational observations.

## Deployed System Architecture

The deployment used the environment described in Section 4.

The deployed system implements the following components:

* Application plane: a web-based management interface and service orchestration component.
* Control plane: Pola PCE for path computation and SR Policy provisioning, together with GoBGP for BGP-LS topology collection and BGP Flow Specification distribution.
* Management plane: a VNFM, OpenStack as the VIM, and Ansible as the SFM.
* Forwarding plane: the existing backbone and distributed SR-aware service functions.

## Service Deployment

Video streams from the video source sites described in Section 4.3 were dynamically steered through SR-aware service functions deployed in the Sapporo, Kanagawa, and Okinawa data centers.

The service functions performed video switching, transcoding, and caption insertion before forwarding the processed streams to the production system.

Operators created service function chains through the web-based management interface.

The management plane instantiated the distributed service functions, after which Service SIDs were assigned.

Service SID information was advertised via BGP-LS after service SID assignment.


## Operational Benefits

The deployed system demonstrated several operational benefits.

* No modifications to the existing backbone infrastructure were required for deployment or operation.
* Service functions were deployed on demand using existing cloud infrastructure.
* SR Policies and Flow Specification rules were automatically generated.
* Operators manage services through an intent-based interface, without requiring awareness of low-level network details, thereby reducing operational complexity.
* Newly deployed service functions became available for path computation once Service SID advertisement (Section 6.4) was completed.

## Scalability Considerations

This deployment demonstrated that the architecture can scale incrementally by deploying additional SRv6-capable service function nodes without changing the overall control architecture.

Once the application, control, and management components are deployed, additional SR-aware service functions can be instantiated or removed using the VIM's native scaling mechanisms.

These service functions become available for path computation without requiring manual updates to the controller configuration.

# Lessons Learned

During the deployment of the SRv6 SFC system over the backbone, several operational issues and design insights were identified.
This section summarizes key observations obtained from real-world operation.

## Service Verification and Observability

Verifying end-to-end service correctness required more than monitoring SR Policy status or the operational state of service functions.
In the deployed video processing service, it also required application-layer verification, such as comparing input and output video streams, to confirm that traffic was processed correctly.

Application-layer verification was performed by comparing streams at the video source sites in Chiba and Kanagawa with the corresponding output from the service function chain.

Furthermore, because service functions were distributed across multiple VIM domains, effective troubleshooting required correlating network-layer, cloud-layer, and application-layer observations across these domains.

During the deployment, however, these types of operational information were monitored using separate tools, and no unified observability framework was available.
As a result, diagnosing service failures required manual cross-referencing between SR Policy state, service function status, and application-layer observations, making troubleshooting time-consuming and operationally complex.

This experience led to the operational recommendations in Section 9.5, which describe a unified multi-layer observability framework.

## Latency-Aware Service Function Placement

This section addresses the placement of SR-aware service functions.
For control-, management-, and application-plane component placement, see Section 9.2.

Because data centers are geographically distributed, inter-site latency has a significant impact on service performance.

For latency-sensitive applications such as real-time video processing, cumulative path latency across multiple sites is an important consideration for service function placement.

In the deployed system, service function placement was determined manually based on operator knowledge of the network topology and latency characteristics.
While the underlying architecture supports incremental scaling of service function nodes (Section 7.4), this manual placement decision process itself does not scale well as the number of deployment sites increases.

This experience led to the operational recommendations in Section 9.2, which describe latency-aware and topology-aware approaches to service function placement.

# Operational Considerations

SRv6 SFC deployments require coordination among the control, management, and application planes to ensure consistent service operations.

## Service SID Allocation

Building on the Service SID allocation described in Section 6.3, Service SID uniqueness within the SR domain MUST be ensured.
Uniqueness is ensured at two distinct levels, corresponding to the Locator and Function fields of an SRv6 SID {{RFC8986}}.

Reachability to the node hosting a service function is provided by the SRv6 Locator assigned to that node and advertised through normal IGP/BGP routing.
Transit nodes along the path need only maintain reachability to this Locator; they are not required to be aware of the Service SIDs corresponding to individual service functions providing the End.AN behavior.

Within a given Locator, however, the Function field associated with a specific End.AN behavior MUST be uniquely assigned so as not to collide with other service functions sharing the same Locator.
Because this Function value assignment is not visible to the routing and forwarding plane, it SHOULD be coordinated by the management plane (e.g., the SFM) prior to advertisement, and verified against the current TED maintained by the control plane via BGP-LS, to prevent collisions across data centers and service functions.

A centralized allocation mechanism SHOULD be used, where the management plane coordinates with the control plane to identify available Function space prior to assignment, thereby preventing address collisions and simplifying multi-site service deployment.

## System Component Placement

Because the deployment spans geographically distributed data centers, the placement of both service functions and control-, management-, and application-plane components has a significant impact on system performance and scalability.

Service function placement affects both path latency and the distribution of traffic load across the SR domain.
As discussed in Section 8.2, placement decisions SHOULD be based on measured inter-site latency between ingress points (e.g., video source sites), data centers, and egress points (e.g., the production system), rather than on static assumptions.
Placement decisions SHOULD also take into account the resource availability of the underlying NFVI at each data center and the current load distribution among already-deployed service functions, to avoid concentrating traffic onto a subset of data centers.
The VIM's native scaling mechanisms (Section 7.4) MAY be used to instantiate or remove service function instances in response to such placement decisions.

Control-plane, management-plane, and application-plane components similarly benefit from careful placement, though the relevant consideration differs by the frequency of interaction.
Interactions between the application plane and the control and management planes (e.g., service function deployment and configuration requests, path computation and SR Policy provisioning requests) occur repeatedly throughout the service lifecycle.
In contrast, interactions between the operator and the application plane (e.g., service requests and status queries) are comparatively infrequent and can tolerate greater physical separation.
Co-locating the application plane with the control and management planes SHOULD therefore take precedence over placing the application plane close to operators or video source sites.

In the described deployment, the application plane, the VNFM, the SFM, and the control plane (Pola PCE and GoBGP) were co-located at a single data center (Sagamihara) to minimize the latency of these frequent interactions.

In this deployment, the VIM centrally managed NFVI resources across the participating data centers.

A logically centralized controller and management architecture SHOULD be used to ensure consistent path computation, service configuration, and policy enforcement across the SR domain.
For large-scale deployments, a hierarchical controller and management model MAY be used to improve scalability while preserving global policy consistency.

## Flow Classification and SR Policy Synchronization

When an SR Policy is intended to steer only selected traffic (e.g., a specific flow or VPN), it SHOULD be associated with a Color so that it is applied only to the intended traffic.

The corresponding BGP Flow Specification rules SHOULD be installed only after the SR Policy has been successfully provisioned and is operational.

Otherwise, traffic MAY be classified to an SR Policy that is not yet operational, resulting in transient traffic misclassification or blackholing.

## Failure Recovery

Failure recovery follows the mechanisms defined in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

In operational deployments, fast reroute at the forwarding plane can maintain connectivity, but service-level state consistency is not guaranteed during failover events.

Therefore, service functions MUST be designed to handle potential state inconsistencies (e.g., buffering, re-synchronization, or idempotent processing).

## Observability

A multi-layer observability framework SHOULD include:

* SRv6 topology and SR Policy state
* Flow classification and traffic steering behavior
* Service function health and availability
* Virtual infrastructure resource utilization

This framework SHOULD include a unified telemetry system spanning both network and cloud domains, enabling cross-layer analysis for rapid fault detection and diagnosis.

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

The security considerations described in {{RFC8402}}, {{RFC8986}}, and {{RFC9256}} apply to this deployment.

This deployment relies on the SR domain trust model described in {{RFC8402}} and {{RFC8986}}.
Operators MUST ensure that SRv6 packets originating outside the trusted SR domain are not accepted for SRv6 processing at domain boundaries.

This deployment provisions SR Policies to SR source nodes directly via PCEP without the use of a hop-by-hop signaling protocol such as RSVP-TE, consistent with the PCE-initiated LSP model described in {{RFC8231}} and {{RFC8281}}.
As noted in those documents, if the security mechanisms defined therein are not used, an attacker able to inject or modify PCEP messages could provision unauthorized SR Policies without the additional verification that a signaling protocol would otherwise provide.
Operators MUST ensure that PCEP sessions used for SR Policy provisioning are protected using appropriate authentication, authorization, and integrity protection mechanisms.

Because service functions are instantiated dynamically and become eligible for path computation after Service SID advertisement (see Section 6.4), operators SHOULD ensure that Service SID information is advertised only for authenticated and authorized service functions.
The management plane SHOULD verify the identity and integrity of a service function instance before its Service SID is advertised into the SR domain.
Otherwise, a rogue or compromised service function could be selected during path computation, resulting in traffic being steered to an unauthorized function.

The export of topology and traffic engineering information via BGP-LS, as described in {{RFC9552}}, may expose commercially sensitive network information.
Operators SHOULD ensure that only trusted consumers are configured to receive this information, and that topology and Service SID information advertised via BGP-LS is protected against unauthorized modification or injection.

Management interfaces SHOULD be protected using mutually authenticated secure transport protocols.

Traffic classification rules and the Color values used to associate them with SR Policies MUST also be protected against unauthorized modification or injection, using appropriate authentication, authorization, and integrity protection mechanisms.
Incorrect association between traffic classification rules, Color values, and SR Policies may result in unintended traffic steering.

Compromise of these components may result in incorrect traffic steering, service misbehavior, or service disruption.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}
The authors would like to thank Mitsuru Maruyama, Katsuhiro Sebayashi, Taisei Tanabe, Ryuta Futami, and Takashi Kurimoto for their valuable reviews.

This work was partially supported by JST CRONOS (No. JPMJCS24N9).
