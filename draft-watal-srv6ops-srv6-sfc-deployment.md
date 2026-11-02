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

venue:
  group: srv6ops
  type: Working Group
  mail: srv6ops@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/srv6ops/
  github: watal/draft-watal-srv6ops-srv6-sfc-deployment

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
  RFC9603:
  I-D.draft-watal-spring-srv6-sfc-sr-aware-functions:
    target: https://datatracker.ietf.org/doc/html/draft-watal-spring-srv6-sfc-sr-aware-functions-06
    title: SRv6 SFC Architecture with SR-aware Functions
    author:
      -
        ins: W. Mishima
        name: Wataru Mishima
      -
        ins: Y. Fukagawa
        name: Yuta Fukagawa
    date: 2026-11-02
    seriesinfo:
      Internet-Draft: draft-watal-spring-srv6-sfc-sr-aware-functions-06

informative:
  RFC7426:
  RFC7665:
  RFC8664:
  RFC8955:
  RFC8956:
  RFC9552:
  RFC9862:
  I-D.draft-ietf-spring-srv6-service-programming:
    target: https://datatracker.ietf.org/doc/html/draft-ietf-spring-srv6-service-programming
    title: SRv6 Service Programming
    author:
      -
        ins: A. Abdelsalam, Ed.
        name: Ahmed Abdelsalam
      -
        ins: C. Filsfils
        name: Clarence Filsfils
      -
        ins: D. Bernier
        name: Daniel Bernier
      -
        ins: C. Li
        name: Cheng Li
      -
        ins: B. Decraene
        name: Bruno Decraene
    date: 2026-07-06
    seriesinfo:
      Internet-Draft: draft-ietf-spring-srv6-service-programming-00
  I-D.draft-ietf-idr-bgp-ls-sr-service-segments:

--- abstract

This document describes the deployment and operational experience of the SRv6 Service Function Chaining (SFC) architecture described in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}} on an academic IPv6 backbone network.

The deployed system integrates SRv6 forwarding, service function management, topology collection, path computation, and flow classification to enable on-demand provisioning of SFC services via a web-based management interface.

This document summarizes the deployment architecture and operational workflow, reports on deployment experience and lessons learned, and provides guidance for operators deploying SRv6 SFC services.

--- middle

# Introduction

Segment Routing over IPv6 (SRv6) {{RFC8986}} enables packet steering through an ordered list of instructions called a segment list.

Service Function Chaining (SFC) {{RFC7665}} can be implemented using SRv6 to steer traffic through a sequence of service functions.

The architecture of SRv6 SFC with SR-aware functions is described in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

This document reports on the deployment and operational experience of this architecture on an academic IPv6 backbone network.
It does not define any new protocols or protocol extensions.
It describes the deployment environment, system architecture, and operational workflow, and discusses the operational benefits, challenges, lessons learned, and considerations for operators deploying SRv6 SFC services.

# Terminology

## Terminology Defined in Related RFCs and Internet-Drafts

The following terms are used in this document as defined in the related RFCs and Internet-Drafts:

- SR and Segment Identifier (SID) are defined in {{RFC8402}}.
- SRv6 is defined in {{RFC8986}}.
- Headend, Color, and SR Policy are defined in {{RFC9256}}.
- SFC, Service Function, and Service Function Chain are defined in {{RFC7665}}.
- Path Computation Client (PCC) and Path Computation Element (PCE) are defined in {{RFC4655}}.
- The Path Computation Element Communication Protocol (PCEP) is defined in {{RFC5440}}.
- PCEP extensions for SR are defined in {{RFC8664}}, with SR Policy candidate path extensions further specified in {{RFC9862}}.
- BGP-LS is defined in {{RFC9552}}.
- BGP Flow Specification is defined in {{RFC8955}} and {{RFC8956}}.
- Forwarding Plane, Control Plane, Management Plane, and Application Plane are defined in {{RFC7426}}.
- NFV Infrastructure (NFVI), Virtualized Infrastructure Manager (VIM), and Virtualized Network Function Manager (VNFM) are defined in {{RFC8568}}.
- Service Segment is described in {{I-D.draft-ietf-spring-srv6-service-programming}}.
- The SRv6 SFC architecture, including the Service Function Manager (SFM) and End.AN, is described in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

## Requirements Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Deployment Objectives

- Demonstrate the SRv6 SFC architecture on an operational IPv6 backbone spanning geographically distributed sites.
- Demonstrate the feasibility of incremental deployment without modifying existing transit backbone routers.
- Evaluate the operational feasibility of on-demand SFC provisioning.

# Deployment Environment

Figure 1 shows the physical deployment environment.

~~~ drawing
 +------------+ +------------+          +------------+ +------------+
 |Video Source| |Video Source|          |Video Source| |Video Source|
 +-----+------+ +-----+------+          +-----+------+ +-----+------+
       \              |                       |              /
        \             |  SINET IPv6 Backbone  |             /
       +-+------------+--+-----------------+--+------------+-+
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

Three geographically distributed data centers in Hokkaido, Kanagawa, and Okinawa were selected to evaluate service chaining over a wide-area network.

## Video Source Sites

University-operated video source servers were located at Kanagawa, Chiba, Ishikawa, and Okinawa.

## NFV Infrastructure and Virtualized Infrastructure Manager

Each data center provides NFV Infrastructure (NFVI) to host SR-aware service functions and operates an independent OpenStack-based Virtualized Infrastructure Manager (VIM).

# Deployment Architecture

The deployment follows the SRv6 SFC architecture described in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

The deployed system is organized into four logical planes: the forwarding plane, control plane, management plane, and application plane.

The following subsections describe the role of each plane and its realization in the deployed system.

## Overall Architecture

Figure 2 illustrates the logical architecture of the deployed system.

- The application plane provides the operator interface.
- The control plane performs topology collection, path computation, SR Policy provisioning, and flow classification.
- The management plane deploys and configures service functions.
- The forwarding plane forwards traffic through SR-aware service functions.

The deployed system enables dynamic deployment and operation of SRv6 service function chains while preserving the existing forwarding infrastructure.

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

The forwarding plane consists of the existing IPv6 backbone and SR-aware service functions deployed at geographically distributed data centers.
The backbone's transit routers require no SRv6-specific modifications.

Traffic is steered through a sequence of service functions using SRv6 segment lists.
Each SR-aware service function is associated with an End.AN SID to process packets and forward them to the next segment in the service chain.

## Control Plane

The control plane is responsible for topology collection, path computation, SR Policy provisioning, and flow classification.

It consists of a Path Computation Element (PCE), a Classification Rule Controller, and a BGP daemon.

The PCE uses a Traffic Engineering Database (TED) containing topology and resource information to compute paths and provision SR Policies.

The Classification Rule Controller generates BGP Flow Specification rules based on flow classification requirements received from the application plane.

The BGP daemon provides the following functions:

- Collecting topology information via BGP-LS, including Service SID advertisements.
- Distributing BGP Flow Specification rules generated by the Classification Rule Controller to the SR source nodes.

## Management Plane

The management plane is responsible for deploying, configuring, and managing the lifecycle of SR-aware service functions.

It consists of three logically distinct functions:

- Virtualized Network Function Manager (VNFM): defined in {{RFC8568}}, responsible for managing the lifecycle of service functions, including instantiation, scaling, and termination.
- Virtualized Infrastructure Manager (VIM): defined in {{RFC8568}}, responsible for managing NFVI compute, storage, and network resources. Each data center has an independent VIM. A single VNFM manages the lifecycle of service functions across all data centers, interacting with the corresponding VIMs.
- Service Function Manager (SFM): defined in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}, responsible for configuring SRv6-specific service parameters, including Service SID allocation and End.AN behavior, after the service function instance becomes operational (Figure 5). The allocation procedure is described in Section 9.1.

Each service function instance operates its own BGP-LS speaker (e.g., an embedded agent or sidecar process).
After receiving the Service SID and End.AN configuration from the SFM, the instance advertises its Service SID to the control plane.
The instance also reports the configuration result to the SFM.

The management plane also supports service function reconfiguration and removal throughout the operational lifecycle of each service function.

## Application Plane

The application plane consists of an operator-facing web-based management interface together with the service orchestration logic.
Based on operator requests, the application plane translates service requirements into intent-based service requests and coordinates the control and management planes to realize them as deployment operations.

The application plane could be extended with closed-loop automation functions that use telemetry feedback from the control and management planes.
These functions would operate at a higher level of abstraction and remain logically separate from the control and management planes.

# Operational Workflow

This section describes the operational workflow for deploying and activating an SRv6 service function chain.
The workflow begins with an operator service request and concludes with traffic steering through the deployed service functions.

The workflow is illustrated in Figures 4-6.

The following abbreviations are used:

~~~ drawing
 Op   = Operator            SFM  = Service Function Manager
 App  = Application         VNF  = Virtualized Network Function
 VNFM = VNF Manager         Ctrl = Control Plane
 VIM  = Virtualized         Src  = Headend
        Infrastructure      Dest = Endpoint
        Manager
~~~
{: #fig-wf-legend title="Abbreviations Used in Figures 4-6"}

In Figures 5 and 6, Ctrl represents the control-plane components described in Section 5.3 collectively.
The BGP daemon handles BGP-LS topology collection and BGP Flow Specification distribution, while the PCE handles path computation and SR Policy provisioning.

Figure 4 shows service function instantiation.
Consistent with {{RFC8568}}, the VNFM issues the lifecycle request, and the VIM allocates and provisions the underlying NFVI resources.

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

Figure 5 shows SRv6-specific service configuration after the service function becomes operational, followed by Service SID advertisement.

~~~ drawing
 App               SFM               VNF              Ctrl
  |                 |                 |                 |
  |    Configure    |                 |                 |
  |   Service SID   |                 |                 |
  |---------------->|                 |                 |
  |                 |    Configure    |                 |
  |                 |   Service SID   |                 |
  |                 |---------------->|                 |
  |                 |                 |    Advertise    |
  |                 |                 |   Service SID   |
  |                 |                 |---------------->|
  |                 |                 |                 | Update TED
  |                 |Configuration OK |                 |
  |                 |<----------------|                 |
  |Configuration OK |                 |                 |
  |<----------------|                 |                 |
  |                 |                 |                 |
~~~
{: #fig-wf-5 title="Service SID Configuration and Advertisement"}

Figure 6 shows SFC activation, including path computation and SR Policy provisioning, followed by optional Flow Specification installation and traffic steering.

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
  |  Request Flow   |                 |                 |
  | Classification  |                 |                 |
  |---------------->|                 |                 |
  |                 |    FlowSpec     |                 |
  |                 |---------------->|                 |
  |                 |                 |   SFC Traffic   |
  |                 |                 |    Steering     |
  |                 |                 |---------------->|
  |                 |                 |                 |
~~~
{: #fig-wf-6 title="SFC Activation and Traffic Steering"}


## Service Request

The operational workflow begins when an operator submits a service request through the web-based management interface.

A service request typically includes:

- headend and endpoint nodes
- optional flow classification rules
- the required sequence of service functions

The application plane translates these service requirements into deployment requests for the control and management planes.

## Network Function Deployment

If one or more requested service functions are not currently available, new service functions are deployed.
Deployment is triggered by an explicit operator request.

## Topology Collection

Topology information is continuously collected via BGP-LS, independently of individual service requests.

After a service function instance becomes operational, the management plane configures its Service SID and End.AN behavior.
Once the configuration has been applied, the instance advertises its Service SID via the BGP-LS extension described in {{I-D.draft-ietf-idr-bgp-ls-sr-service-segments}}.
The instance also reports the configuration result to the SFM.

A service function is included in the TED only after its advertisement is received and is excluded when the advertisement is withdrawn.
The SFM monitors service function availability and eligibility for traffic steering.
When an instance becomes unavailable or is no longer eligible for traffic steering, the SFM SHOULD initiate withdrawal of the corresponding Service SID advertisement without delay.

For an Anycast Service SID, the affected node SHOULD withdraw only its own advertisement.
The shared SID SHOULD remain available while at least one eligible instance advertises it.

Stale or delayed Service SID updates may cause incorrect path computation and steer traffic to unavailable or invalid service functions.

## Path Computation

When a service request is received, the control plane computes an SR Policy satisfying the requested service chain based on the current network topology and the available service functions stored in the TED.

## SR Policy Provisioning

As illustrated in Figure 6, the computed SR Policy is provisioned to the SR source node acting as the PCC, using PCEP {{RFC9603}}.
The SR Policy specifies the SRv6 segment list representing the selected service function chain.

## Flow Classification

If flow classification is requested, the application plane submits the classification requirements to the Classification Rule Controller in the control plane.

The Classification Rule Controller generates the corresponding BGP Flow Specification rules, which are distributed to the SR source node by the BGP daemon.

The rules associate matching traffic with the intended SR Policy by Color and Endpoint, as shown in Figure 6.

## Traffic Steering

Once the SR Policy is operational, the SR source node can steer traffic through the selected service functions.
When flow classification is requested, traffic steering begins after the corresponding Flow Specification rules are installed.
Only traffic matching the installed rules is steered through the service function chain.

## Monitoring

Operational status is continuously monitored throughout the service lifecycle using telemetry collected from both the network and infrastructure.

Operators may also verify traffic steering using SR path tracing or in-band telemetry mechanisms.

For content-modifying services, application-layer verification may be required in addition to network and infrastructure monitoring, as discussed in Sections 8.1 and 9.3.

## Service Update and Removal

The deployed system supports updates throughout the service lifecycle.

Typical operations include:

- modifying or removing service function chains
- redeploying failed service functions
- removing service functions

When a service function becomes unavailable, the affected Service Function Node MUST stop processing packets using its Service SID, as specified in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.
The SFM SHOULD withdraw the corresponding Service SID advertisement as described in Section 6.3.

Service updates require coordination among the forwarding, control, management, and application planes to maintain operational consistency.
The ordering and verification of update operations are discussed in Section 9.4.

# Deployment Experience

This section describes the deployment and operational experience of the SRv6 SFC architecture.
The system was deployed to support remote video production.

The following subsections describe its architecture, the service it supported, and the operational behaviors and design insights observed during the deployment.
This section does not specify requirements or recommendations.

## Deployed System Architecture

The deployment used the environment described in Section 4.

The deployed system implements the following components:

- Application plane: a web-based management interface and service orchestration component.
- Control plane: Pola PCE for path computation and SR Policy provisioning, a Classification Rule Controller for generating flow classification rules, and GoBGP for BGP-LS topology collection and BGP Flow Specification distribution.
- Management plane: a VNFM, OpenStack as the VIM, and Ansible as the SFM.
- Forwarding plane: the existing backbone and distributed SR-aware service functions.

## Service Deployment

Video streams from the source sites described in Section 4.3 were dynamically steered through SR-aware service functions deployed in the Hokkaido, Kanagawa, and Okinawa data centers.

The service functions performed video switching, transcoding, color correction, and caption insertion before forwarding the processed streams to the production system.

Operators created service function chains through the web-based management interface.

The management plane instantiated the distributed service functions, after which Service SIDs were assigned to the instances and the corresponding information was advertised via BGP-LS.

## Operational Benefits

The deployed system demonstrated several operational benefits.

- No modifications to the existing backbone infrastructure were required for deployment or operation.
- Service functions were deployed on demand using the existing NFV infrastructure.
- SR Policies and Flow Specification rules were automatically generated.
- Operators manage services through an intent-based interface without configuring low-level network details.

## Scalability Considerations

This deployment demonstrated that additional SR-aware service functions can be introduced without changing the overall control architecture.

Once instantiated, configured, and advertised via BGP-LS, these service functions become available for path computation without requiring manual updates to the PCE's service-function configuration.

# Lessons Learned

This section presents operational observations and design insights gained from the deployment.
It focuses on service verification, state correlation, and orchestration timing and consistency.

## Service Verification and Observability

Verifying end-to-end service correctness requires more than monitoring SR Policy status or the operational state of service functions.

For the deployed video processing service, network-layer monitoring alone could not verify processing correctness; application-layer comparison of input and output video streams was needed.

For service chains containing multiple content-modifying service functions, checking only the original source and final destination was insufficient to identify which function caused an incorrect result.
This experience demonstrated the value of per-function application-layer verification (see Section 9.3).

## Multi-domain State Correlation Limitations

The deployed system spans multiple VIM domains across geographically distributed data centers.

Operational state was monitored independently across the network, infrastructure, and application layers.
No unified mechanism was available to correlate SR Policy state, service function status, and application-layer verification results.

As a result, troubleshooting required manual correlation of information from multiple sources, increasing the effort needed to identify the cause of service failures.

## Service Orchestration Timing and Consistency Issues

Correct service operation depends on the ordering and timing of service function readiness, Service SID advertisement, SR Policy provisioning, and Flow Specification installation.

Delays in BGP-LS propagation and TED synchronization may cause discrepancies between the actual service function state and the state used for path computation.

Installing Flow Specification rules before the corresponding SR Policy becomes operational may leave the intended traffic-steering state unavailable.
As specified in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}, a PCC does not activate the associated traffic-steering state until the SR Policy is available.

Therefore, provisioning and verifying the SR Policy before distributing the corresponding Flow Specification rules is RECOMMENDED to reduce transient inconsistencies and simplify operational verification.

These observations motivate the operational sequencing recommendations described in Section 9.4.

# Operational Considerations

SRv6 SFC deployments require coordination among the control, management, and application planes to ensure consistent service operations.

## Service SID Allocation

Service SID allocation MUST ensure that distinct service functions within the same SRv6 Locator do not inadvertently use the same Function value.

When multiple service function instances are intentionally configured to share an Anycast Service SID, the shared SID MUST be treated as a single logical service endpoint.
Such sharing MUST be explicitly coordinated to ensure that all instances provide compatible service behavior.

Reachability to the node hosting a service function is provided by the SRv6 Locator assigned to that node and advertised through normal IGP/BGP routing.
Transit nodes along the path only need to maintain reachability to this Locator; they are not required to be aware of the Service SIDs corresponding to individual service functions providing the End.AN behavior.

In this deployment, the SFM checked which Function values were already in use and selected an available value before configuring the corresponding End.AN behavior.

This approach reduces the risk of collisions with already allocated Service SIDs and provides a consistent allocation procedure across the distributed data centers.

## Failure Recovery

Failure recovery follows the mechanisms defined in {{I-D.draft-watal-spring-srv6-sfc-sr-aware-functions}}.

Fast reroute at the forwarding plane can maintain network connectivity, but does not necessarily preserve service-level state.
During failover, traffic may be redirected to a different service function instance that does not share the processing state of the original instance.

For stateful service functions using Anycast SIDs, operators need to consider session affinity and service-state consistency across instances.
The required mechanisms depend on the service function and its processing model.
Where failover is expected to preserve session or processing state, the deployment MUST provide appropriate mechanisms.

## Observability

Effective observability for SRv6 SFC involves monitoring multiple aspects of the system, including:

- SRv6 topology and SR Policy state
- Flow classification and traffic steering behavior
- Service function health and availability
- Virtual infrastructure resource utilization
- Application-level service processing results

For content-modifying services, application-layer verification of individual service functions SHOULD compare each function's input and output (see Section 8.1).

Packet mirroring capabilities at SRv6 endpoints may provide additional information for service-level monitoring and troubleshooting.
The applicability of such capabilities to SRv6 SFC deployments requires further investigation.

## Operational Sequencing

Automated orchestration SHOULD ensure that service functions are instantiated and configured before their Service SIDs are advertised via BGP-LS.
The SFM SHOULD monitor and record the configuration results reported by service function instances.

The corresponding SR Policy SHOULD be provisioned and verified before Flow Specification rules are installed to steer traffic through the service chain.

When an existing SR Policy is modified or replaced, flow classification rules SHOULD be updated only after the replacement policy has been successfully provisioned and verified.

These procedures help avoid transient traffic misclassification and blackholing during service deployment and updates.

# Security Considerations

The security considerations in {{RFC8402}}, {{RFC8986}}, and {{RFC9256}} apply to this deployment, which relies on the SR domain trust model described in {{RFC8402}}.
Operators MUST ensure that SRv6 packets originating outside the trusted SR domain are not processed as SRv6 traffic at domain boundaries.

This deployment provisions SR Policies to SR source nodes directly via PCEP, consistent with the PCE-initiated LSP model described in {{RFC8231}} and {{RFC8281}}.
Without adequate protection, an attacker could inject or modify PCEP messages to provision unauthorized SR Policies.
Operators SHOULD ensure that PCEP sessions used for SR Policy provisioning are protected using appropriate authentication, authorization, and integrity protection mechanisms.

Because service functions are instantiated dynamically and become eligible for path computation after Service SID advertisement (see Section 6.3), operators SHOULD ensure that Service SID information is advertised only for authenticated and authorized service functions.

The management plane SHOULD verify the identity and integrity of a service function instance before advertising its Service SID into the SR domain.
If this verification is not performed, a rogue or compromised service function could be selected during path computation, resulting in traffic being steered to an unauthorized function.

Operators SHOULD ensure that topology and Service SID information advertised via BGP-LS is protected against unauthorized modification or injection.

Operators SHOULD ensure that BGP-LS topology and Service SID information is distributed only to authorized consumers.

Management interfaces SHOULD be protected using mutually authenticated secure transport protocols.

Operators SHOULD ensure that flow classification rules and Color values used to associate them with SR Policies are protected against unauthorized modification or injection using appropriate authentication, authorization, and integrity protection mechanisms.

Unauthorized modification or compromise of flow classification rules, Color values, or SR Policies may result in unintended traffic steering, service misbehavior, or service disruption.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}
The authors would like to thank Mitsuru Maruyama, Katsuhiro Sebayashi, Taisei Tanabe, Ryuta Futami, and Takashi Kurimoto for their valuable reviews.

This work was partially supported by JST CRONOS (No. JPMJCS24N9).
