---
title: "Modeling the Digital Map based on RFC 8345: Sharing Experience and Perspectives"
abbrev: Digital Map Modelling
docname: draft-havel-nmop-digital-map-01
category: info

stand_alone: true
submissiontype: IETF  
area: "Operations and Management"
wg: NMOP

consensus: true
v: 2

author:
  -
    fullname: Olga Havel
    org: Huawei
    email: olga.havel@huawei.com
  -
    fullname: Benoit Claise
    org: Huawei
    email: benoit.claise@huawei.com
  -
    fullname: Oscar Gonzalez de Dios
    org: Telefonica
    email: oscar.gonzalezdedios@telefonica.com
  -
    fullname: Ahmed Elhassany
    org: Swisscom
    email: Ahmed.Elhassany@swisscom.com
  -
    fullname: Thomas Graf
    org: Swisscom
    email: thomas.graf@swisscom.com

contributor:
  -
    fullname: Nigel Davis
    org: Ciena
    email: ndavis@ciena.com
  
normative:

informative:

   Catalog:
              title: YANG Impact Analysis
              author:
                org: YANG Catalog
                date: false
              target: https://yangcatalog.org/yang-search/impact_analysis/ietf-network-topology@2018-02-26

   Digital Map Hackathon:
              title: Digital Map Hackathon IETF 120 Presentation
              author:
                org: NMOP 
                date: false
              target: https://datatracker.ietf.org/meeting/120/materials/slides-120-hackathon-sessd-digital-map-hackathon-00
   
   Digital Map Hackathon Code:
              title: Digital Map Hackathon IETF 120 Code
              author:
                org: NMOP 
                date: false
              target: https://github.com/digital-map-exp/digital-map-public
      
--- abstract

This document shares experience in modelling Digital Map based on the IETF RFC 8345 topology YANG modules and 
some of its augmentations. The document identifies a set of open issues encountered during the modelling phases, 
the missing features in RFC 8345, and some perspectives on how to address them.
For definition of Digital Map concepts, requirements and use cases please refer to the 
"Digital Map: Concept, Requirements, and Use Cases" document.

Discussion Venues 

   This note is to be removed before publishing as an RFC.

   Source for this draft and an issue tracker can be found at
   https://github.com/OlgaHuawei.

--- middle

# Introduction

{{!RFC8345}} specifies a topology YANG model with many YANG augmentations for different technologies and service types. 
The modelling approach based upon {{!RFC8345}} provides a standard IETF-based API.

At the time of writing (2024) and according to the YANG catalog, there are at least 92 YANG modules that are augmenting {{!RFC8345}}; 
91 IETF-authored modules and 1 BBF-authored module. According to the YANG catalog, 19 of these modules have maturity level of 'ratified', 
18 of them have maturity level of 'adopted', 27 modules have maturity level of 'latest-approved', 
and 28 of these modules have maturity level of 'initial'.  
The up-to-date information can be found in the YANG Catalog {{Catalog}}.

From the set of IETF RFCs and I-Ds (at different level of maturity), we designed a Digital Map 
Proof of concept (PoC), with the following objectives and functionalities:

* Can the central RFC 8345 YANG module be a good basis to model a Digital Map?
+ How the different topology related IETF YANG modules fit (or not) together?
+ Modelling of Digital Map entities, relationships, and rules how to build aggregated entities and relationships. Does 
the base model support key requirements that emerge for a specific layer?
+ Modelling multiple underlay/overlay layers from Layer 2 to Layer 3 to customer service layer. To what extent it is 
easy to augment the base model to support new technologies?
- Can the base model be augmented for any new layer and technologies?


This memo documents an experience in the modeling aspects of the Digital Map, based on a PoC implementation, basically 
documenting the effort and the open issues encountered so far. During the PoC, we also identified a set of requirements 
and verified the PoC approach by demoing it iteratively.

Practically, we developed a PoC with a real lab, based on multi-vendor devices, with {{!RFC8345}} as the base YANG module.  
The PoC successfully modelled the following:

-  Layer 2 network topology (used {{!RFC8944}})
-  Layer 3 network topology (used {{!RFC8346}})
-  OSPF routing topology (aligned with [I-D.ogondio-nmop-ospf-topology])
-  IS-IS routing topology (aligned with [I-D.ogondio-nmop-isis-topology])
-  BGP routing topology
-  MPLS LDP
-  MPLS Traffic Engineering (TE) tunnels
-  SRv6 tunnels
-  L3VPN service
           

We are further verifying this PoC by implementing it iteratevely at the IETF Hackathons and engaging the wider community, 
starting with IETF120 Hackathon {{Digital Map Hackathon}}. 
The Hackathon open source is available at {{Digital Map Hackathon Code}}     
    
We delivered the IETF120 hackathon, with the plan to incrementally add more functionality in the future hackathons.
The multi-vendor operator LAB was used for this hackathon (with Huawei, Cisco, Juniper deviced).
The goal of the Digital Map Hackathons (starting from IETF120) is to demonstrate how operators can use 
the IETF Topology YANG models to represent a real carrier network. 
   
We started with one particular problem space: 
How to use IETF topology model to represent a real carrier network based on IS-IS and OSPF domains 
(target for planning/simulation purposes). The IETF120 Hackathon focused on generic topology queries, and started to 
compare IS-IS topology drafts augmenting RFC8345 versus potential RFC8345bis (gaps identified in RFC8345) approaches.
We also started analysis and prototype how to retrieve performance metrics or configuration attributes 
(defined in IS-IS Data Model for the IS-IS Protocol {{!RFC9130}} and retrieved via device API) 
northbound from the Controller via RFC8345 API and its IS-IS augmentation. 


## Terminology

{::boilerplate bcp14-tagged}

Please refer to the "Digital Map: Concept, Requirements, and Use Cases" {{!I-D.havel-nmop-digital-map-concept}} for 
the definition of the following terms used in this document:

   + Topology
   + Multi-layered Topology
   + Topology layer
   + Digital Map
   + Digital Map modelling
   + Digital Map model
   + Digital Map data


# The IETF Network Topology Approaches

## IETF Network Topology

{{!RFC8345}} provides a simple generic topological model.  It defines the abstract /generic /base model for network 
and service topologies. It provides the mechanism to model networks and services as layered topologies with common 
relationships at the same layer and underlay/overlay relationships between the layers.

{{!RFC8345}} consists of two modules: 'ietf-network' and 'ietf-network-topology'.  The 'ietf-network' module defines 
networks and nodes, while 'ietf-network-topology' module adds definitions for links and termination points.

The relationships inside the layer are containment/aggregation (a network has nodes, a network has links, a node has 
termination points), source (a link has one source termination point) and destination (a link has one destination 
termination point).

The relationships between the layers are modelled via supporting relationship:

* network A is supported by network B - this may model overlay/underlay relationship

* nodes, links and termination points of network A are supported by nodes, links and termination points of network B.  
Overlay and underlay nodes, links and termination points must match underlay and overlay networks supporting it

## IETF Network Topology TE

{{?RFC8795}} defines a YANG model for representing, retrieving and manipulating traffic engineering (TE) topologies.  This is a more complex 
model which augments {{!RFC8345}} with TE topology information as follows:

* TE nodes, links, and termination points are defined using the core RFC8345 concepts

    - TE topology augments 'ietf-network' with topology identifier (provider, client and topology id), as well as other 'TE' 
  information

    - TE node augments 'node' with 'te-node-id' and other 'TE' information
    
    - TE link augments 'link' with 'TE' information
    
    - TE termination point augments termination point with 'te-tp-id' and 'TE' information

* Tunnel, tunnel termination point, local link connectivity, node connectivity matrix, and some supporting and underlay 
relationships are defined outside of the core RFC 8345 entities and relationships

## Why RFC8345 is a Good Approach for Digital Map Modelling

The main reason for selecting RFC8345 for modelling is its simplicity and that is supports majority of the core requirements. 

Please refer to the "Digital Map: Concept, Requirements, and Use Cases" {{?I-D.havel-nmop-digital-map-concept}} for 
more details of the core Digital Map requirements:

The requirements REQ-BASIC-MODEL-SUPPORT, REQ-LAYERED-MODEL, REQ-PROG-OPEN-MODEL, REQ-STD-API-BASED, REQ-COMMON-APP, 
REQ-SEMANTIC, and REQ-LAYER-NAVIGATE are automatically fulfilled with RFC8345 and extensions:

* Basic model with network, node, link and interface entity types

* Layered topology

* Open and programmable

* Standard, multi-vendor

* Multi-domain

* Semantics for layered topology

* Inter-layer and between-layer relationships

The requirements REQ-SEMANTIC for semantics and REQ-LAYER-NAVIGATE for layered topology and relationships are partially fulfilled, there may be 
need for some additional semantics.  

Other core requirements REQ-EXTENSIBLE, REQ-PLUGG and REQ-GRAPH-TRAVERSAL are not supported by {{!RFC8345}}:

*  Extensible with metadata

*  Pluggable to other YANG modules and non-YANG data

*  Optimized for graph traversal

In some cases, for REQ-PLUGG for pluggable to other YANG modules, the links are already done by augmenting 'ietf-network' 
and 'ietf-network-topology'. For others, we need to add some mechanisms to link the models and data.

# Digital Map Modelling Experience

## What Is Not in The Base Model?

Based on some shared experience, the following are listed as set of candidate extensions to {{!RFC8345}} for 
Digital Map modelling and APIs:

RFC8345-GAP-BIDIR:
: An alternate approach to model bidirectional links

RFC8345-GAP-MULTI-POINT:
: An alternate approach to multi-point connectivity

RFC8345-GAP-MULTI-DOMAIN:
: Links between domains/networks

RFC8345-GAP-SUBNETWORK:
: A network decomposition into sub-networks

RFC8345-GAP-MULTI-NETWORK:
:  Nodes, links, and termination points belonging to different networks

RFC8345-GAP-SUPPORTING:
:  Supporting relationships between different types of entities

RFC8345-GAP-SEMANTIC:
:  More network topology related semantic is needed

RFC8345-GAP-PLUGG:
:  How to connect in a generic way to other YANG modules and data

### Bidirectional Links (RFC8345-GAP-BIDIR)

   One of the core characteristics of any network topology is the link
   directionality. While data flows are unidirectional, the
   bidirectional links are also common in networking.  Examples are
   Ethernet cables, bidirectional SONET rings, socket connection to the
   server, etc.  We also encounter requirements for simplified service
   layer topology, where we want to model link as bidirectional to be
   supported by unidirectional links at the lower layer.

   {{!RFC8345}} defines all links as unidirectional, it does not support
   bidirectional links. It was done intentionally to keep the model as
   simple as possible. While simplifying the model itself, the data
   and APIs are more complex for the cases with bidirectional
   links. In such cases, there is no need to increase the amount of instances / data
   transferred via API, stored in the database, or managed/monitored by modeling
   unidirectional links.

   This document suggests to model the bidirectional
   connections as pairs of unidirectional links.

   {{?I-D.davis-opsawg-some-refinements-to-rfc8345}} further elaborates on the need for bidirectional links in the 
   network topologies and in the Digital Map.  It also proposes how RFC8345 can be augmented to support missing components.

The following are the candidate approaches of how we can address this limitation:

* Use the current RFC8345 approach and implement it via multiple unidirectional links 

* Don't change RFC8345, leave to different augmentations to solve the problem their own way

* Augment RFC8345 via basic approach as suggested in {{?I-D.davis-opsawg-some-refinements-to-rfc8345}}, fully backward compatible, appears simple and sufficient

* Augment RFC8345 via more sophisticated approach as suggested in {{?I-D.davis-opsawg-some-refinements-to-rfc8345}}, more complex but improves the integrity of the model, same instance structures produced

* Consider RFC8345bis

We suggest to start the work on RFC8345bis to provide the backward compatible way to support bidirectional 
links in the core topology model defined in ietf-network-topology. The starting point can be the basic approach from
{{?I-D.davis-opsawg-some-refinements-to-rfc8345}} that adds the following:

* direction-of-link with value BIDIRECTIONAL 
* direction-of-point with value BIDIRECTIONAL
* list of termination points that could be used for bidirectional links as an alternative to having source and 
destination for unidirectional (although we can also implement it via the existing source and destination when 
direction BIDIRECTIONAL)

### Multipoint Connectivity (RFC8345-GAP-MULTI-POINT)

   The RFC8345 defines all links as point to point and unidirectional, it does not support multi-point links 
   (hub and spoke, full mesh, complex).  It was done intentionally to keep the model as simple as possible. 
   The RFC suggests to model the multi-point networks via pseudo nodes.

   Same as with unidirectionality, while simplifying the model itself, we are making data and APIs more complex for 
   multi point topologies and we are increasing the amount of data transferred via API, stored in the database or 
   managed/monitored.

   One of the core characteristics of any network topology is its type and link cardinality.  Any topology model 
   should be able to model any topology type in a simple and explicit way, including point to multipoint, bus, ring, 
   star, tree, mesh, hybrid and daisy chain.  Any topology model should also be able to model any link cardinality in 
   a simple and explicit way, including point to point, point to multipoint, multipoint to multipoint or hybrid.

   By forcing the implementation of all topology types and all options for cardinality via unidirectional links and 
   pseudo nodes, we are significantly increasing the complexity of APIs and data, but also lacking full topology 
   semantics in the model.  The model cannot be fully used to validate if topology instances are valid or not.

   Note that the point to multipoint network type is also required in some cases at the OSPF layer.

   {{?I-D.davis-opsawg-some-refinements-to-rfc8345}} further elaborates on the need for multipoint connectivity in 
   network topologies and in the Digital Map, in general.  It also proposes how RFC8345 can be augmented to 
   support these missing components.

The following are the candidate approaches of how we can address this limitation:

* Use the current RFC8345 and implement via virtual nodes

* Don't change RFC8345, leave to different augmentations to solve the problem their own way 
(see how it is done in {{?RFC8795}})

* Augment RFC8345 via basic approach as suggested in {{?I-D.davis-opsawg-some-refinements-to-rfc8345}}, 
fully backward compatible, appears simple and sufficient

* Augment RFC8345 via more sophisticated approach as suggested in {{?I-D.davis-opsawg-some-refinements-to-rfc8345}}, 
more complex but improves the integrity of the model, same instance structures produced 

* Consider a RFC8345bis that provides backward compatible enhancement 
(similar to {{?I-D.davis-opsawg-some-refinements-to-rfc8345}} approach without augmentations)

We suggest to start to work on RFC8345bis to provide the backward compatible way to support multipoint connectivity 
in the core topology model defined in ietf-network-topology. The starting point can be the basic approach from 
{{?I-D.davis-opsawg-some-refinements-to-rfc8345}} that adds the following:

* list of termination points for multipoint links as an alternative to having source and destination for point to point 
links via the existing source and destination
* role-of-point
* type-of-link

###  Links Between Networks (RFC8345-GAP-MULTI-DOMAIN)

The RFC8345 defines all links as belonging to one network instance and having the source and destination as node and 
termination point only, not allowing to link to termination point of another network. This does not allow for links 
between networks in the case of multi-domains or partitioning. The only way would be to model each domain as 
node and have links between them.

In our IS-IS PoC and Hackathon (following {{?I-D.ogondio-nmop-isis-topology}}), we modelled IS-IS areas as networks 
and we needed to extend the capability to have links between different areas.  We added network reference as well to 
the source / destination of the link.  {{?RFC8795}} also augments links with external-domain info for the case of 
links that connect different domains.

The IS-IS topology {{?I-D.ogondio-nmop-isis-topology}} models Autonomous System (AS) or IS-IS domain as a network, 
and IS-IS areas as attributes of IS-IS nodes.  The RFC8345 extension can be used to model IS-IS areas as networks and 
IS-IS links between L1-2 nodes as links between two different areas.  This is not problem for OSPF, although the OSPF 
nodes can belong to multiple areas, the links can belong to only one area.

The following are some benefits of lifting the RFC 8345 limitations that all links must belong to only one network 
instance:

*  IS-IS processes would be grouped in an area via the standard IETF RFC8345 network->node relationship.

*  Applications and algorithms will consume topologies based on the generic entities and relationships, 
they will not need to understand the meaning of specific IS-IS attributes.

*  The approach would be aligned with the IS-IS topology model and the IS-IS network view in manuals and documentation.

*  Provide capability to link different IGP domains and links between them.

*  Link between multiple networks/sub-networks is the common concept in network topology.

The following are the candidate approaches of how we can address this limitation:

* Use the current RFC8345 and implement domains via properties in augmentations

* Don't change RFC8345, leave to different augmentations to solve the problem their own way 
(see how it is done in {{?RFC8795}})

* Augment RFC8345 by adding some simple solution (e.g. move {{?RFC8795}} approach for multi-domain to RFC8345 digital map)

* Consider a RFC8345bis 

We suggest to start to work on RFC8345 bis to provide the backward compatible way to support links between networks 
in the core topology model defined in ietf-network-topology. The starting point can be to evaluate the approach 
from {{?RFC8795}} that adds the external domain reference to the link via the external network, node and tp reference.


###   Networks Part of Other Networks (RFC8345-GAP-SUBNETWORK)

   RFC8345 does not model networks being part of other networks,
   therefore cannot model subnetworks and network partitioning.  We
   encountered this problem with modelling IS-IS and OSPF domains and
   areas.  The initial goal was to model AS/domain with multiple areas
   so that the Digital Map model contains information about how the AS
   is first split into different IGP domains and how each IGP domain is
   split into different areas.  This is a common problem for both IS-IS
   and OSPF.

The following are the candidate approaches of how we can address this limitation:

*  Leave it as it is in RFC8345, don't model AS and IS-IS/OSPF domain directly, they would be modelled via the underlying IP network and IS-IS/OSPF enabled routers. This could be achieved via supporting relationship to L3 network and L3 nodes

* Leave it as it is in RFC8345, leave to different augmentations to solve the problem their own way 

* Leave it as it is in RFC8345, model AS, IGP domains and areas as networks and use supporting relationship to model the network topology design, with only areas having nodes. This does not describe the correct nature of the relationship, semantic is missing.

* Augment RFC8345 by adding some simple solution to support additional partitioning relationship between networks.

* Consider a RFC8345bis 

We suggest to start to work on RFC8345 bis to provide the backward compatible way to support partitioning of networks 
in the core network model defined in ietf-network. The solution needs to add a part-of relation between different 
networks, where one network (e.g. OSPF Domain) can contain multiple networks (e.g. OSPF areas)


###   Nodes, tps and links in multiple networks (RFC8345-GAP-MULTI-NETWORK)

   RFC8345 does not allow nodes and TPs to belong to multiple areas
   without splitting them into separate entities with separate keys. In
   OSPF case, we have nodes that can belong to different areas, but
   interfaces can only belong to one area. In the case of IS-IS,
   although all tutorial are stating that nodes can belong to one area
   only, the IETF, openconfig and vendor yang models and CLI allow IS-IS
   processes with all its interfaces to belong to multiple areas.

The following are the candidate approaches of how we can address this limitation:

   *  Use the current RFC8345 approach, this is not the problem for read
      but may be an issue for write for simulation as we would expect
      that the interface lists in different nodes and networks be the
      same without being able to validate.

   *  Augment RFC8345 to optionally allow nodes to be
      defined outside of network tree and enable reference as an
      alternative to the containment in the tree.  This may be a bigger
      change to the network topology approach as it would have bigger
      impact on the topology tree. Nevertheless, it can be an optional approach so would be backward compatible for 
      those augmentations that do not want to use it

   * Consider RFC8345 bis 

We suggest to work on RFC8345 bis to provide the simple backward compatible way to support both the current RFC8345 approach 
of creating multiple instances and the approach of sharing the instances. The solution needs further analysis as it has 
bigger impact on the topology tree than other enhancements.

###  Missing Supporting Relationships (RFC8345-GAP-SUPPORTING)

   RFC8345 defines supporting relationships only between the same type
   of entities.  Networks can only be supported by networks, nodes can
   only be supported by nodes, termination points can only be supported
   by terminations points and links can only be supported by links.

   During the PoC, we had a scenario  where at one layer of topology we had a link with TPs where the TPs are 
   logical and did not have underlay TP, but the only underlay connection we were able to define was to 
   underlay nodes. The same happened with nodes and networks.

   Therefore, we encountered the need to have TP supported by node
   and node supported by network.

   The RFC8795 also adds additional
   underlay relationship between node and topology and link and
   topology, but via a new underlay topology and not via the core
   supporting relationship.

The following are the candidate approaches to address this limitation:

* Use the current RFC8345 approach, leave to different augmentations to solve the problem their own way 
(see how it is done in {{?RFC8795}})

* Augment RFC8345 by adding some simple solution (e.g. move {{?RFC8795}} approach to RFC8345)

* Consider RFC8345 bis that provides backward compatible enhancement (e.g. via {{?RFC8795}} basic approach)

We suggest to work on RFC8345 bis to provide the backward compatible way to add the missing supporting relationships:
tp->supporting->node, node->supporting->network. 

###  Missing Topology Semantics (RFC8345-GAP-SEMANTIC)

   Many augmentations add the additional topological semantics, with same concepts modelled in a different way 
   in different augmentations. We already mentioned that some semantic is missing from the RFC8345
   topology model, like bidirectional and multi-point.  The following is
   also missing from the model:

   *  Relationship Properties.  The supporting relationship could have
      additional attributes that give more information about the
      supporting relationship.  That way we could use it for
      aggregation, underlay with primary/backup, load balancing, hop,
      sequence, etc.

   *  Termination point roles.  We are missing semantics for the common
      topology roles: uni, i-nni,e-nni, primary, backup, hub, spoke, 
   
   *  Node roles.  We are missing semantics for the common node
      roles: access, core, metro

   * Link roles. We are missing semantics for the common link 
     roles: uni, i-nni, e-nni

   *  Layers / Sublayers.  We need further analysis to determine in
      network types are sufficient to support all scenarios for layers/
      sublayers.  The network types are more related to technologies so
      in the case that the same technology is used at different layers,
      we may need some additional information in the model.  
      For further study.

   *  Tunnels and paths.  We modelled tunnels and paths via {{!RFC8345}} but
      we lost some semantics that is in {{!RFC8795}}.

   *  Node cross-connects.  We did not need to model cross connect / 
      connectivity metrices in the PoC. In the case it is needed, 
      {{!RFC8345}} does not provide the capability, semantics that 
      is in {{!RFC8795}}. For further study.

   *  Supporting or underlay.  We modelled all underlay relationships
      via supporting, further analysis is needed to determine pros and
      cons of this approach versus RFC8795 approach of using underlay
      topology.
   

The following are the candidate approaches to address this limitation:

* Use the current RFC8345 approach, leave to different augmentations to solve the problem their own way 
(see how it is done in {{?RFC8795}})

* Augment RFC8345 by adding some simple solution (e.g. move {{?RFC8795}} approach to RFC8345)

* Consider RFC8345bis

We suggest to work on RFC8345 bis to provide the backward compatible way to add the minimum semantics that the 
community agrees is required for the core topology. We need to further investigate the {{?RFC8795}} approach and 
evaluate if some parts could be moved to RFC8345. 
                   
## Plug-in other IETF YANG data models (RFC8345-GAP-PLUGG) 

We need a new RFC8345 augmentation for the purpose of connecting IETF topology modules to other IETF YANG modules. 
In some cases, this are already done by augmenting 'ietf-network'    
and 'ietf-network-topology'. For others, we need to add some mechanisms to link the IETF topology to other
models and data.

The mechanims must specify the following:

* What category? What is the category of the external YANG model (e.g. configuration, assurance, performance, ..)

* Which protocol? Is the information retrieved via netconf, restconf, ..

* Which server? Where the information can be retrieved from (the same controller or some other restconf/netconf server)

* Which instance? How to connect IETF topology module instances to other IETF YANG module instances (because we have different keys)

* Which path? What yang paths contain external data for network, node, termination-point, link

  
There are two  ways how this can be defined:

* USER DEFINED: External user defines how to connect topology to external YANG module in a generic way via some Digital Map API. 
This information is than retrieved from the application via the Digital Map API and application can automatically 
build the request for retrieval of external data.

* CONTROLLER DELEGATED: Delegated to the controller, who gives back info how to connect to external modules for each instance

The following are the candidate approaches how we can address this limitation:

* BASIC: Dont propose any new mechanism, leave to different augmentations to solve the problem their own way

* AUGMENT: Augment RFC8345 with some Digital Map plugin module

* NEW-MODULE: Create a new Digital Map plugin module that is used for building relations between the Digital Map model/data and 
other IETF models/data

We implemented the basic approach in IETF120 Hackathon, where we duplicated the properties of IS-IS at the controller API
via augmenting the RFC8345 with technology specific module. This was done just to explain the problem, knowing that
the more sofisticated solution is needed. We need to implement more advanced option for connecting ietf-l2-isis-topology 
to ietf-isis. The solution must be generic to support any other augmentation and yang files. We suggest to analyze the 
second (AUGMENT) and third (NEW-MODULE) approach in the IETF121 hackathon, together with 2 approaches how that 
information can be defined (USER DEFINED or CONTROLLER DELEGATED) and come back with the reccomendation.

##  Open Issues (for Further Analysis or Resolved)

   The following are the open issues that need further analysis or have been resolved:

   *  Do we need separation of L2 and L3 topologies?

      During the PoC/Hackathon we encountered different solutions with separate
      set of requirements.  In one solution, the L2 and L3 topology were
      the same with separate set of attributes, while in another
      solution we had difference in L2 and L3 topology (e.g.  Links
      Aggregation, Switches and Routers).

      The RFC8944 defines L2 topology and RFC8346 defines the L3
      topology.  They allow to have either one or two instances of this
      topology.

      The decision if we need separate network instances for L2 and L3
      topologies may be based on specific network topology and
      provider's preferences.

      Resolved: the RFC8345 is flexible and it can support both the same network instance with L2 + L3 augmentations 
      or separate network instances with supporting relationship between. The operator should decide what option is 
      needed for their solution.

   *  Do we need sublayers as well?  Layers versus sublayers versus
      layered instances?  

      Resolved: Layers/sublayers could be implemented via multiple network types. The new data nodes for layer are present only when the network type for the layer is present. The new data nodes for the sublayer are present when the network types for both layer and sublayer are present. The solution could also enable either single or multiple instances, like in the previous point.

      Further analysis: In the case that the same technology is used at different layers, we may need some additional information in the model, in the case that it cannot be modelled via network types + supporting relations.

   *  Same technology at service versus underlay?  BGP per VPN vs common
      BGP shared between multiple VPNs.  Different layers, same model,
      relationship define the layer. 
   
      Further analysis is needed.

   * There are potential circular dependencies in layering. For example routing can be underlay for tunnels, but tunnel interface can also be in the routing table.

      Further analysis is needed.

   * Best way to build the connection to other YANG modules for navigation.

      Further analysis and prototypes at the future Hackathons

#  RFC8345 Augmentation Analysis

##  Why Analysis?

   During our PoC and Hackathon, we decided to use the RFC8345  model and API for the layered topology from the physical to the top layer, accross the domains, without the need to understand any specific information about specific domain, technologies and details required for specific network management functions / use cases. 
   
   We took the hat of application developer and used the software architecture guidelines to provide the right level of abstraction for topologies at the controller northbound interfaces. This means that the API client /application must understand and draw the topology from RFC8345 (as originally intended by this draft) without understanding all augmentations that add new topological entities / relations. 

   After determining that the RFC8345 can provide the right level of abstraction for the layered topology (and be improved by addressing gaps via backward compatible way), we wanted to understand if different IETF RFC8345 augmentations add its own topology semantics and if they address any gaps we identified.

##  What did we Analyse?
   Our goal was to do some high level analysis of all the modules that augment the RFC8345 and understand the following 2 high level points:

   * What is the purpose of augmentation?

      * Is it augmenting topology for some identified topology gaps?
      * Is it augmenting topology for some specific functionality, like TE. Functional category TE, Technology Generic.
      *	Is it augmenting topology for some specific technology, like ISIS. Technology L3 ISIS
      * Is it augmenting to connect to some other modules, like Inventory, PM?

   * How it the augmented info added?

      * By adding the attributes, events, using the types, new relations (non-topological) no impact on topology. Layered topology can be understood / drawn using the RFC8345.
      * By adding the topological entities and topological relations – full impact on topology. Layered topology cannot be understood / drawn using the RFC8345.
      * By adding some additional topological semantics – partial impact on topology (e.g. hub / spoke). Layered topology can be understood/drawn but roles will not be understood.

##  RFC8345 Augmentations that add Topology Semantics

The most important result of our analysis would be extension categories that add new topological entities and new topological relations. This means that we would not be able to use RFC8345 for understand/draw the layered topology. Examples:

* RFC8795 {{?RFC8795}}:

   * We have TE Topology and TE Nodes, LTPs, TE Links modelled using the abstract RFC8345 topology
   * we have tunnels and TTPs + underlay outside of abstract RFC8345 topology 

* RFC8542 {{?RFC8542}}:

   * we have fabric network, fabric and fabric tps modelled using the abstract RFC8345 topology
   * we have device nodes, device links, device ports + relations between them modelled outside of the abstract RFC8345 topology
 
This means that the API client /application cannot understand and draw the topology from RFC8345 (as originally intended by this draft) without understanding all augmentations that add new topological entities / relations. 

##  Summary of RFC8345 Augmentation Analysis Results

Our full analysis is on github and is still work in progress, waiting for different authors to confirm our conclusions. The full current analysis results can be found on https://github.com/ietf-wg-nmop/Misc/tree/main/Digital-Map-Analysis. 

We will present only the current summary of the RFC8345 Augmentation Analysis in this draft:

* We started our analysis by identifying the 102 YANG modules, based on the info from the YANG catalog at the moment we started analysis.
* We determined that out of those 102 YANG modules, 27 were deprecated
* We determined that out of those 102 YANG modules, 23 were expired
* We determined that out of those 102 YANG modules, 8 were Non-NMDA compliant
* That left us with 44 modules to analyze, including the 2 RFC8345 modules
* We then continued to identify the following about each of the modules, please see the subsequent sections for more details:

   * Authors
   * Functional Category
   * Use Cases (outstanding)
   * Technology
   * Network Types
   * Organization
   * Maturity Level
   * What modules are imported / augmented
   * Extensions Categories
   * What topology concepts they use or add
   
* We identified that out of 44 modules analyzed

   * 41 are importing ietf-network
   * 33 are importing ietf-network-topology
   * 12 are importing ietf-te-topology
   * there may be others using indirectly ietf-te-topology without importing the module, for example we identified that do it via import of ietf-te and te-types

### Authors

We identify authors and we added the column for them to add their comments if they aggree / disagree with our analysis, this is the work in progress.

Our hope is that the authors would be able to review our analysis of their drafts / RFCs and verify our conclusions. We will also discuss with them  specific reasons for their modelling decisions and hope to influence some changes based on the future modelling guidelines.

### Functional Category

We decided to use functional category to understand why the modules were added. We identified the following functional categories:

* TOPOLOGY
* TE
* NETWORK SLICE
* SERVICE
* VN (Virtual Network)
* TRANSPORT NETWORK CLIENT SERVICE
* PM
* INVENTORY
* OAM
* PROVISIONING
* MAP
* INCIDENT
* ENERGY
* PARTITIONING


### Use Cases

We got recommendation to also identify what Use Cases are implemented by which Module.

This is planned for the next version of the draft, to be filled in by Authors.

### Technology

We decided to use technology category to identify if the modules are generic or for specific technology. We identified the following technology categories:

* GENERIC
* L3
* ISIS
* OSPF
* FABRIC
* WSON
* SAP
* PACKET
* MPLS
* ETHERNET
* MICROWAVE
* FLEXI-FRID
* L3SR
* OPTICAL
* OTN
* FGNM

### Network Types

The module augmented RFC8345 with the following network types that could be used for layers / sublayers of the Digital Map:

* l2-topology
* l3-unicast-topology
* te-topology
* isis-topology
* ospfv2-topology
* fabric-network
* te-topology
* wson-topology
* service
* sap-network
* l3-te
* packet
* mpls-topology
* te-topology
* eth-tran-topology
* mw-topology
* network-slice
* network-inventory-mapping
* nrp
* flexi-grid-topology
* srv6
* optical-impairment-topology
* otn-topology

### Organization

All except 1 module are IETF modules. Only the following modules has organization other than IETF:

* BBF module: bbf-network-map

### Maturity Level

We initially used the maturity level from YANG Catalogue, but some modules have not been shown with the correct maturity level and there is outstanding review comment to correct this in the next version of the draft.

### What Modules are Imported

We identify what modules import one or more of the following modules:

* ietf-network or ietf-network-state
* ietf-network-topology or ietf-network-topology-state
* ietf-te-topology or ietf-te-topology-state

This would be useful for the future discussion in regards to some proposals by the TEAS team to use RFC8795 ietf-te-topology as the Digital Map core model and API.

### Extension Categories

This is the most important part of our analysis, what has been added and how. We identified the following categories:

* New attributes
* New events
* New non-topological relations
* New topological entities (list of entities needed for clarification)
* New topological relations (list of relations needed for clarification)
* New topological semantics on top of the core mode (e.g. roles hub/spoke, primary backup, ..) (list of semantic needed for clarification)
* New sublayers (info)

### Topology Concepts

For each of the modules we added some information about what topological concepts were important for each of the modules and which ones are reuse of the core one and which ones are the new once.

##  RFC8345 Augmentations Analysis Conclusion

We determined after the analysis that we need to start working on the guidelines to create a Digital Map. The RFC8345 augmentations are not consistent, which makes it very hard to deploy the multi-layer digital map.

#  Digital Map Modeling Guidelines

##  Guidelines for Generic Digital Map Extensions

   Generic Digital Map extensions are the RFC8345 extensions required
   for all technologies and layers in the Digital Map. We already discussed some options for individual limitations in
   the previous sections, here is the summary:

    1. Use the current RFC8345 approach, leave to different augmentations 
    to solve the problem their own way 

    2. Augment RFC8345 network, node, link and termination point for any
    changes needed from a new digital map module

~~~~
   module: dm-network-topology
           augment /nw:networks/nw:network:
                   .... additions
           augment /nw:networks/nw:network/nw:node:
                   .... additions
           augment /nw:networks/nw:network/nt:link:
                   .... additions
           augment /nw:networks/nw:network/nw:node/nt:termination-point:
                   .... additions


   // This can be a separate draft with describing pros and cons of
   // different approaches and yang model proposal.  Add reference to
   // this draft when submitted
~~~~

    3. Make backward compatible updates to RFC8345, work on RFC8345 bis

The following are some important guidelines mentioned in the RFC8345 that should be taken into account when suggesting 
the approach:

* "The data models allow applications to operate on an inventory or topology of any network at a generic level, 
where the specifics of particular inventory/topology types are not required.
At the same time, where data specific to a network type comes into play and the data model is augmented, 
the instantiated data still adheres to the same structure and is represented in a consistent fashion.  
This also facilitates the representation of network hierarchies and dependencies between different network components 
and network types"

* “It is possible for links at one level of a hierarchy to map to multiple links at another level of the hierarchy.  
For example, a VPN topology might model VPN tunnels as links.  Where a VPN tunnel maps to a path that is composed of 
a chain of several links, the link will contain a list of those supporting links.  Likewise, it is possible for a 
link at one level of a hierarchy to aggregate a bundle of links at another level of the hierarchy."

Our recommendation on how to extend RFC8345 for Digital Map is to stay in spirit of RFC8345 and augment with 
non-topological info only. Reuse network, node, link, tp for all topological entities, reuse supporting for 
layering and add new properties/attributes and references to other modules
Therefore, we suggest to work on RFC8345 bis to provide the backward compatible way to address all 
identified limitations. 

The alternative of having the core topology augmentations in either TE modules or technology specific modules is not generic enough and is not in the spirit of having the core topology model to model topology in the consistent manner between different technologies and TE and non-TE topologies.

## Guidelines for New Technologies/Layers Extensions

   There are already drafts that support augmentation for specific
   technologies.  These drafts augment network, node, termination point
   and link, but also add different topological entities inside
   augmentations.  For example, we have examples like this:

~~~~
   module: new-module
           augment /nw:networks/nw:network/nw:network-types:
         +--rw new-topology!
                   augment /nw:networks/nw:network:
                                   ....
                   augment /nw:networks/nw:network/nw:node:
                           .... adding list of tps of other type
                                    (e.g. tunnel TPs in TE draft)
                           ... adding new supporting relationship
                                    supporting-tunnel-termination-point
                                    (te draft)
                   augment /nw:networks/nw:network/nt:link:
                                   .... adding tunnels (te draft)
                   augment /nw:networks/nw:network/nw:node/
                                                 nt:termination-point:
                                   ....
~~~~
   There is a need to agree some guidelines for augmenting IETF network
   topology, so that additional topological information is not added in
   the custom way.  There is also need to categorize the current augmentations and the impact of RFC 8345 bis 
   based on what has been added for different technologies:

* new properties/attributes (e.g. ietf-l2-topology, ietf-l3-unicast-topology, ietf-isis-topology)
* new events (e.g. ietf-l2-topology)
* new topological entities (e.g. ietf-te-topology, ietf-dc-fabric-topology)
* new topological relations (e.g. ietf-te-topology)
* type reuse (e.g. ietf-dots-telemetry, ietf-dc-fabric-types, ietf-dc-fabric-topology)

<--
This can be a separate draft.  Guidelines with examples?  Add reference to this draft when submitted
-->


## Guidelines for Digital Maps Connections to Other Components

Digital Map must be pluggable:
  
* We must connect to other YANG modules for inventory, configuration, assurance, etc 
* Not everything can be in YANG, we need to connect digital map YANG model with other modelling mechanisms, 
both southbound, northbound and internally

Also, there are already some modules that connect network topology to other YANG modules. 
We will investigate different approaches and propose the best practices. 
The following are some existing approaches proposed in IETF:
 
* How to connect network topology to interface {{?I-D.draft-ietf-ccamp-if-ref-topo-yang}}
* How to connect network topology to hardware inventory {{?I-D.ietf-ccamp-network-inventory-yang}}
* How to connect network topology to ivy inventory {{?I-D.draft-wzwb-ivy-network-inventory-topology}}
* How to connect network topology to performance monitoring {{?RFC9375}}

We did the initial implementation during the IETF120 Hackathon that focused on showing the problem via duplicating the 
attributes of IETF-ISIS in RFC8345 augmentations. We need better solution and will continue analysis and prototype 
in the future Hackathons.

### How to connect YANG models with other modelling mechanisms
There is need to connect YANG network topology to models and data outside of YANG, for example BMP, IPFIX, logs, etc.

## Digital Map APIs

   This will include hierarchical APIs for cross-domain figure, IETF
   YANG Based API (read and write, change subscription and notify) and
   Query API

##  Digital Map Knowledge

   The following knowledge was needed to build Digital Map:

   *  Abstract IETF Entities and Relationships as in {{!RFC8345}}

   *  {{!RFC8345}} Augmentations for a)Layers/sublayers b)Entities
      (services and subservices), c)Properties

   *  Rules for aggregating entities

   *  Rules for instantiating relationships (inter-layer and intra-
      layer)

   *  Mapping from devices and controllers

   What can be achieved with existing RFC8345 YANG:

   *  Entities (base class IETF Network, IETF Node, IETF Link, IETF TP)

   *  Properties

   *  Relationships

   Next steps

   *  How to support temporal

   *  How to support spacial

   *  How to support historical

# Conclusions

Digital Map Modelling and Data are needed for the current Operator Use Cases, and are also basis for the Digital Twin. 
During our PoC and Hackathon we have proven that Digital Map can be modelled using {{!RFC8345}}.  
Nevertheless, we proposed some extensions/augmentations to {{!RFC8345}} to support Digital Maps. 

After analysing all augmentations of RFC8345 modules, we determined that there must exist some constraints in 
regards to how to augment the core Digital Map model for different Layers and Technologies in order to support the 
approach recommended in RFC8345 and implemented in our PoC/Hackathon. Therefore, we recommend that IETF adopts the 
guidelines how to augment the core RFC8345 modules for Digital Map. 
All entities should augment IETF node, IETF network, IETF link or IETF Termination Point and augmentation can 
only include new properties, events, non-topological references. 

We suggest to start the work on RFC8345 bis to provide the backward compatible way to support the following basic 
topological features:

* bidirectional links
* multipoint connectivity
* cross-domain links via links between networks
* multi-domain via network partitioning
* shared topological entities between different domains
* additional supporting relations
* additional semantics required for core topologies

We also suggest to start working on a new draft that would define the RFC8345 augmentation or new modules for the purpose of
* connecting IETF topology module to other IETF YANG modules (what yang paths are connected to node, termination-point)
* defining what IETF topology module instances are related to the IETF YANG module instances (because we have different keys)
* avoid duplication the properties in RFC8345 augmentations

# Security Considerations

This section uses the template described in Section 3.7 of {{?I-D.ietf-netmod-rfc8407bis}}.

The YANG modules cited in this document define a schema for data
that is designed to be accessed via network management protocols such
as NETCONF {{!RFC6241}} or RESTCONF {{!RFC8040}}.  The lowest NETCONF layer
is the secure transport layer, and the mandatory-to-implement secure
transport is Secure Shell (SSH)  {{!RFC6242}}.  The lowest RESTCONF layer
is HTTPS, and the mandatory-to-implement secure transport is TLS
 {{!RFC8446}}.

The Network Configuration Access Control Model (NACM)  {{!RFC8341]
provides the means to restrict access for particular NETCONF or
RESTCONF users to a preconfigured subset of all available NETCONF or
RESTCONF protocol operations and content.

The specifications that define these modules call out both
sensitive and vulnerable writable and readable data nodes. These
considerations are not reiterated here.

# IANA Considerations

This document has no actions for IANA.


--- back

# Acknowledgments
{:numbered="false"}

Many thanks to Mohamed Boucadair for his valuable contributions, reviews, and comments.
Many thanks to Bo Wu for her review of augmentation analysis and for the recommendations she shared.

