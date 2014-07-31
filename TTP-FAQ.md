#Table Type Patterns FAQ
This FAQ addresses questions related to the [OpenFlow Table Type Patterns](https://www.opennetworking.org/images/stories/downloads/sdn-resources/onf-specifications/openflow/OpenFlow Table Type Patterns v1.0.pdf)) and [OpenFlow Controller-Switch NDM Synchronization](https://www.opennetworking.org/images/stories/downloads/sdn-resources/onf-specifications/openflow/OpenFlow Controller-Switch NDM Synchronization v1.0.pdf)) specifications.
This list of Frequently Asked Questions (and answers) is intended to be a living document.  If you have a TTP-related question that is not addressed here, or if the answer you found is unclear, please help us improve this resource by forwarding your questions to the leadership of the Forwarding Abstractions WG (Curt Beckmann: beckmann@brocade.com, Ben Mack-Crane: ben.mackcrane@huawei.com) or ONF administration.

* [What are TTPs?](#What are TTPs?)
* [How are TTPs helpful?](#How are TTPs helpful?)
* [Do TTPs make OpenFlow control more complex?](#Do TTPs make OpenFlow control more complex?)
* [Do TTPs force a rigid structure on SDN, limiting programmability?](#Do TTPs force a rigid structure on SDN, limiting programmability?)
* [Do TTPs allow for optional switch capability?](#Do TTPs allow for optional switch capability?)
* [If our device supports most of the messages of a TTP, can we claim support for the TTP?](#If our device supports most of the messages of a TTP, can we claim support for the TTP?)
* [Will TTPs be required (What if I don't think they're helpful)?](#Will TTPs be required (What if I don't think they're helpful)?)
* [Who is responsible for defining TTPs?](#Who is responsible for defining TTPs?)
* [Can new TTPs be built from existing (reusable) parts?](#Can new TTPs be built from existing (reusable) parts?)
* [Who maintains a TTP?](#Who maintains a TTP?)
* [Do TTPs affect the OF-Switch protocol?](#Do TTPs affect the OF-Switch protocol?)
* [Who is responsible for maintaining the TTP description language?](#Who is responsible for maintaining the TTP description language?)
* [Are TTPs intended to describe a particular chip pipeline?](#Are TTPs intended to describe a particular chip pipeline?)
* [Do a controller and a switch need to "synchronize" with each other to use TTPs?](#Do a controller and a switch need to "synchronize" with each other to use TTPs?)
* [What's the difference between TTPs and NDMs?](#What's the difference between TTPs and NDMs?)
* [Are TTPs dependent on OF-Config protocol?](#Are TTPs dependent on OF-Config protocol?)
* [What versions of OF-Switch do TTPs support?](#What versions of OF-Switch do TTPs support?)
* [Are TTPs just for multiple flow table scenarios?](#Are TTPs just for multiple flow table scenarios?)
* [Do TTPs mean recoding of controllers and switches?](#Do TTPs mean recoding of controllers and switches?)
* [Are Multiple Flow Table use cases that important?](#re Multiple Flow Table use cases that important?)
* [Are there any TTP-related Tools available?](#Are there any TTP-related Tools available?)
* [Are TTP files sent over the wire?](#Are TTP files sent over the wire?)
* [How do TTPs impact conformance testing?](#How do TTPs impact conformance testing?)
* [How does one create a TTP?](#How does one create a TTP?)
* [Is there a repository for TTPs?](#Is there a repository for TTPs?)
* [What controllers will support TTPs?](#What controllers will support TTPs?)

##<a name="What are TTPs?"></a>What are TTPs?
TTPs are "Table Type Patterns" — templates that spell out what OF-Switch protocol features and messages a switch needs to support (and a controller needs to abide by) in a given role (for a given “use case”).

TTPs are described and their syntax and semantics are specified in the ONF Technical Specification [OpenFlow Table Type Patterns](https://www.opennetworking.org/images/stories/downloads/sdn-resources/onf-specifications/openflow/OpenFlow Table Type Patterns v1.0.pdf).

Some example TTPs are available at the ONF [TTP Repository](https://github.com/OpenNetworkingFoundation/TTP_Repository).

##<a name="How are TTPs helpful?"></a>How are TTPs helpful?
As OF-Switch has evolved to support multiple tables and more datapath protocols, it has become very feature-rich and flexible. The flexibility can make switch side implementation difficult or inefficient unless additional use-case-related information can be referenced in advance.  In particular, traditional fixed-function (e.g. ASIC-based) switches are often unable to support many OpenFlow features without that information.  More flexible platforms (e.g., CPU or NPU based) can also take advantage of additional information, e.g., in selecting optimal lookup algorithms and allocating table memory.  TTPs provide a standardized way of describing the necessary details that help product developers write applications with confidence.  Each TTP has a unique identifier and selected parameters which switches and controllers can use to ensure agreement on the details specified in the TTP.  

A switch developer can write code to support a TTP use case and be confident that only the TTP-defined messages need to be supported in that context.  This makes the coding problem tractable.  Similarly, a controller writer can develop application (or "service adaptation layer") code for a given TTP use case, with confidence that any switch that supports the TTP will support the relevant messages.  TTPs will be particularly helpful in simplifying the coding of advanced OpenFlow datapath control (where many flow tables or optional functions are needed). Because TTPs describe the expected controller/switch messaging unambiguously, they will also improve interoperability.  

TTPs will also be helpful in testing or benchmarking contexts where participants want advance notice to ensure conformance or optimize performance.  Such participants expect precise descriptions of what messages will be used during testing or benchmarking.  Because TTPs describe a set of messages, matches, actions, etc, in a standard way, they provide a great means of specifying what OpenFlow features a device must support for a particular use case.  For this reason, new use-case-oriented conformance levels are likely to be described using TTPs.

##<a name="Do TTPs make OpenFlow control more complex?"></a>Do TTPs make OpenFlow control more complex?
No.  TTPs can range from simple to complex according to the complexity of the datapath that is being controlled.  A simple TTP might describe a single flow table and require selected functionality that is specified as optional in the OF-Switch Specification. An example would be a very simple IPv6 router. Another TTP might describe two or three flow tables with only required functions, such as a layer 2 switch with ACLs and VID translation.  Yet another TTP might include six flow tables, support switching, routing, tunneling and multicast, etc.  Describing the required datapath controls in a TTP simply provides a common reference for implementers who need to support the use case.

As OpenFlow and SDN mature the usage of advanced OpenFlow features will increase.  Because multiple flow tables can often improve deployment scalability, multi-tables use cases will likely become the norm over time.   For any given use case there may be many different ways to structure the datapath control (i.e., many possible flow table patterns).  A TTP can help manage this growing complexity by focusing implementers on a common datapath model for a given use case, promoting interoperability.

##<a name="Do TTPs force a rigid structure on SDN, limiting programmability?"></a>Do TTPs force a rigid structure on SDN, limiting programmability?
No.  A TTP encodes decisions that must be made when programming an SDN (e.g., the switch datapath structure and controls).  A TTP can encode any datapath structure that OpenFlow can control.  So a TTP is a tool, used in the development process, and it does not impose any constraint on the SDN solution beyond those naturally chosen during development.  As with any other software, if a TTP includes an undesirable constraint for a given use case, it can be re-written to remove that constraint (creating a new version of the TTP or a different TTP, as desired).

A TTP (once written) is rigid in some ways, but also allows for some flexibility.  There is a “meta” level of flexibility; controllers and switches may support more than one TTP, just as they may support many use cases, allowing flexibility in the selection of a TTP.  In addition, individual TTPs can support flexibility to some degree.  A TTP can be constructed so that all the functionality described in the TTP must be supported in the switch. In such case, that particular TTP would be pretty rigid; all flow tables and flow entry types, etc, would need to be supported in the switch.  But even that TTP would not force some aspects of switch implementation, such as table size.  A controller can discover what table sizes are supported at connection time.  If large tables are required but only small tables are implemented, the controller can decide not to continue.

##<a name="Do TTPs allow for optional switch capability?"></a>Do TTPs allow for optional switch capability?
Yes.  Additional flexibility can be included in a TTP by making some flow entry types and other functionality (such as per-entry counters) optional.  When a TTP has optional functionality, a switch supporting the TTP may or may not implement the optional functionality. Sorting out whether optional functionality is available in the switch can be done at control initialization time using TTP parameters.  

Of course the idea is that a TTP should specify optional functionality only when it is not vital to the intended use case, but provides some benefit, such as monitoring or support for something that is only required in some deployments.  In situations where the “optional” functionality is required, the controller can use parameters to learn whether a switch supports it, and can raise a flag when the functionality is not available.  There may also be situations where the switch implements the optional functionality, but the controller does not need it.  TTP parameter negotiation allows the controller to inform the switch that the optional functionality won’t be needed, in case the switch can benefit from that information (e.g. by enabling some resource or performance optimizations).

TTP parameters are described in more detail in the OF-TTP spec.  Details on negotiating TTP functionality using TTP Parameters is described in the NDM Synchronization spec.

##<a name="If our device supports most of the messages of a TTP, can we claim support for the TTP?"></a>If our device supports most of the messages of a TTP, can we claim support for the TTP?
To legitimately claim support for a TTP, a switch must implement all non-optional functionality described by the TTP.  As mentioned above, a TTP may describe some functionality that is optional.  Support for the TTP can be fairly claimed even if the optional functions are not supported, though product documentation should make clear what is and is not supported.  (For consistency in the marketplace, documentation of optional functionality support should refer to the OptFunc parameter values that are or are not supported.  (See the OF-TTP specification and example TTPs for OptFunc details.)

##<a name="Will TTPs be required (What if I don't think they're helpful)?"></a>Will TTPs be required (What if I don't think they're helpful)?
TTPs are optional; many simple use cases can be supported on a variety of devices, including ASIC-based devices, using the "classic" OpenFlow framework without TTPs. In addition, complex use cases can be developed on more flexible devices without using TTPs. And so, for use cases that only require a single flow table and no optional functionality, TTPs may be less helpful.

But even where highly flexible switch platforms (such as virtual switches and NPU-based switches) are deployed TTPs are useful for switches to communicate support for a specific pipeline configuration to controllers.  Applications and controllers are often developed for general situations.  That is, they are written without knowledge of what platforms will be available.  So it may be that a controller or application is developed using a TTP, then deployed in an environment with platforms that are capable of supporting that TTP.   

##<a name="Who is responsible for defining TTPs?"></a>Who is responsible for defining TTPs?
Anyone, even non-ONF members, can define TTPs. TTP authors can have various backgrounds.

__Switch provider__: a TTP can express how a switch family can be easily controlled through OpenFlow

* Example: Broadcom’s OF-DPA

__App or controller provider__: TTPs can describe the switch behavior needed for a particular use case (i.e. a particular device role in a network architecture)

* The [simple example TTPs](https://github.com/OpenNetworkingFoundation/TTP_Repository/tree/master/Simple%20example%20TTPs) might be considered as basic flexible switch models for a MAC bridged or IPv4 network; however, TTPs can be written for switches playing more specialized roles, for example an edge switch supporting a VxLAN overlay network.

__Mixed group__: TTPs can be created by a standards group (e.g., an ONF working group) to describe requirements for a particular solution

* The Forwarding Abstraction Working Group, which developed the OF-TTP specification, has also developed some TTPs for educational purposes (see the later question regarding a TTP repository).  Though these TTPs are primarily for education, they may also have practical value.

New TTPs can be inspired by existing TTPs. For example: An app provider might notice that 3 switch-oriented TTPs all have the ability to support some interesting common functionality as a subset, and might create a TTP based on that common functionality subset.  This “converged” TTP represents functionality which is present in the original different TTPs. With support from the app provider, such a TTP could win support from the 3 original switch TTP creators, as well as from yet other switch providers, thus advancing a platform independent TTP.

##<a name="Can new TTPs be built from existing (reusable) parts?"></a>Can new TTPs be built from existing (reusable) parts?
Re-use of TTP elements can only be achieved in a limited way in the current version of the TTP framework. 

It is clear that the ability to create new TTPs quickly from existing parts that capture well-understood datapath functions would be very desirable.  Today one can cut and paste bits from existing TTPs to write a new TTP, but the TTP language still lacks some features to make this more convenient.  Stay tuned (or contribute to enhancing the TTP language to this end). For the current version of the TTP framework the possibility of improving re-use through creative TTP development tools is being explored (see the TTP tools question later in this FAQ).

##<a name="Who maintains a TTP?"></a>Who maintains a TTP?
Maintenance of a TTP is the responsibility of TTP "owner", as expressed by the TTP naming authority.  (This is a required text field within a TTP that is part of a TTP’s identifier.)  The owner would be expected to maintain a TTP by adding fixes or enhancements if additional flow entry types turn out to be needed or useful for the intended use case.  TTPs include version numbering, and a modified TTP will be given a higher version number.  (For more details on versioning, see the OF-TTP specification.)

TTP “ownership” may be transferrable.  For example, we might foresee a scenario where a member-created TTP becomes popular and the consensus of the community is transfer maintenance of the TTP to the community.  In such a case, ONF (FAWG or other WG) may clone a TTP based on an earlier TTP, giving it a new identifier to indicate the new ownership.  When ONF or other entity recreates an earlier TTP, it could typically use the original name without causing an identifier conflict because it will be in a new namespace (the naming authority field will differ).  In this way, ownership can be transferred while maintaining some continuity in naming.  If the TTP is changed in some way, it would be a “derivative work” that might only be considered to be the same TTP “in spirit”.  (Best practice may discourage similar names for markedly different TTPs.)

##<a name="Do TTPs affect the OF-Switch protocol?"></a>Do TTPs affect the OF-Switch protocol?
The answer is “no”, except in very limited ways.  The goal of defining TTPs was to create a framework for describing how to use the existing OF-Switch Protocol and not to extend it.  A TTP can require support for a feature that is defined as optional in the OF-Switch specification, but this does not change the feature itself.  TTPs can include “built-in flow mods”, which are effectively flow entries that are so central to the switch functionality that the use case only makes sense when those entries are present.  When a TTP with built-in flow mods is active, the switch will assume the entry is present in the respective table, even if the entry has not been sent. Built-in flow-mods can also cause some other minor variances from traditional OpenFlow semantics, e.g., they cannot be deleted.  There are a few other changes that could be made to the OF-Switch specification to accommodate TTPs including the following:

* Some new error messages may be defined (e.g., to indicate a flow table entry does not conform to the active TTP)
* Table Features messages are handled slightly differently (e.g., since an active TTP defines the datapath table pattern, this cannot be reconfigured by the controller)
* A protocol extension to negotiate and set the parameters for an active TTP may be added

For more details, see the “Implementation Considerations” section of the OpenFlow Table Type Patterns document.

##<a name="Who is responsible for maintaining the TTP description language?"></a>Who is responsible for maintaining the TTP description language?
The ONF owns the TTP language.  Responsibility for maintaining of the TTP description language belongs to the working group that is chartered with that task by the ONF board.  The working group chartered with that responsibility as of this writing is the Forwarding Abstractions Working Group.

##<a name="Are TTPs intended to describe a particular chip pipeline?"></a>Are TTPs intended to describe a particular chip pipeline?
TTPs were conceived and developed as a way to describe “implementation independent” switch behavior (a use case), and thus TTP were intended to describe an abstract switch architecture rather than specific physical architecture.  Such abstract architecture TTPs can be mapped onto different physical switch architectures.  The expectation is that an abstract TTP that maps onto many physical devices will have an advantage in adoption.  Application architects should therefore seek to develop such TTPs

Nevertheless, TTPs can be written to cater to a particular physical device pipeline. Shortly before the TTP standard was approved, Broadcom introduced their OF-DPA effort, based on the TTP framework but focused on their switch architecture.  (As of this writing, the TTP that represents OF-DPA capabilities is still in development, likely to complete within a few more weeks.)

If a particular switch vendor’s TTP gains an adoption advantage, one possible outcome is that other vendors may be motivated to support that TTP, which would represent a form of market-driven functional convergence at the device level.  In this way, TTPs can support an evolution toward a long-promised SDN reality.

##<a name=""></a>Do a controller and a switch need to "synchronize" with each other to use TTPs?
When TTP support is integrated into products (controllers and switches), then it is necessary to synchronize the two endpoints to enable TTP-related functionality.  This can be done by configuration or during OF Switch initialization.  At this time, the config-based approach would either use the OF-Config-based NDM Sync mechanism (see the NDM sync spec) or some proprietary (out-of-scope) mechanism of configuration such as CLI commands at both ends. The OF-Switch-based mechanism would use an OF-Switch extension mechanism that is, at the time this is written, under development as an experimenter extension by the Forwarding Abstractions WG.

##<a name="What's the difference between TTPs and NDMs?"></a>What's the difference between TTPs and NDMs?
Table Type Patterns (TTPs) describe switch models in terms developed for the current table-focused generation of the OF-Switch Protocol.  “Negotiable Datapath Model” (NDM) is a generic term for switch models of any kind, including TTPs.  Some discussions have envisioned more advanced models, though none are close to specification at this time.  The OF-Config mechanism created for negotiation of a switch model was designed to support TTPs as well as future datapath models. Because that scheme is agnostic to which switch model is used, the mechanism is called the “NDM Negotiation” scheme.
 
##<a name="Are TTPs dependent on OF-Config protocol?"></a>Are TTPs dependent on OF-Config protocol?
TTPs themselves, and products that support TTPs, are not strictly dependent on the OF-Config protocol.  However, the rich negotiation protocol described in the NDM Sync specification was defined based on the OF-Config 1.2 protocol.  In situations where OF-Config1.2 is not supported by both ends, alternative mechanisms, such as CLI commands, will typically be available to configure an active TTP on both ends of a connection

Additionally, an OF-Switch experimenter extension is being evaluated for providing basic negotiation capabilities in the absence of OF-Config 1.2 support.  This extension will be helpful in the near-term when product support for both OF-Config 1.2 and TTPs is in the early stages.  If the OF community consensus is that the extension is more broadly useful, it can be formalized as an ONF extension.

##<a name=""></a>What versions of OF-Switch do TTPs support?
The OF-TTP specification was written with OF 1.3 in mind (a long-term stable release).  The current TTP language may provide sufficient support for OF1.0, OF1.3 and OF1.4.  (OF1.1 and OF1.2 were only deployed in limited contexts.)  Future OF 1.x versions are also intended to be supported.  Some effort was invested to allow “automatic” support for future versions of OF-Switch, if versions are created without adding new protocol structures.  Support of new structures may require an upgrade to TTP language.

OF-Switch 1.0 support was included in order to make TTPs usable for describing OF1.0 test cases or benchmarking scenarios to avoid the need to develop a separate TTP-like scheme for OF1.0.

##<a name="Are TTPs just for multiple flow table scenarios?"></a>Are TTPs just for multiple flow table scenarios?
No.  TTPs also offer benefits that are relevant to single flow table datapath scenarios:

* Test and benchmarking use case profiles: TTPs describe the messages that must be supported
* Optimization: By clarifying what functions are required, switches can be optimized for a use case. (Note that these required functions may include functions identified as optional in the OF-Switch specification.)
* Single flow table interoperability: OF-Switch describes a variety of matches and actions, including many that are optional.  Capabilities messages are not expressive enough for handling combinations of required and optional features, but TTPs can represent all needed messages, thereby clarifying what optional features might be required in a given context
* Single flow tables can also involve other complex behavior, like meters, group entries, etc.  TTPs are able to express such functionality, and clarify interoperability around that functionality
* Shorthand for expressing interoperability in a market context: TTPs have names that can be used in discussions about product interoperability.  In a way, TTPs can describe a “function level agreement” (FLA)

##<a name="Do TTPs mean recoding of controllers and switches?"></a>Do TTPs mean recoding of controllers and switches?
Maybe.  There are two or three contexts for engaging with TTPs:

1. Non-product engagement: TTPs can be used as to describe unambiguous test or benchmarking scenarios.  The TTPs are used by humans to understand what messages will be used in a given test or benchmarking scenario.  Tools may be developed that can leverage TTPs in producing test stimulus or in analyzing test or benchmark results.  In these “non-product” scenarios, the product is not explicitly coded to be aware of TTPs (and thus may not support message checking or TTP negotiation, etc). 
2. No product rewrite is required for test profiles, and for “data sheet” TTP support (in the absence of negotiation).
3. For negotiated TTPs, some full function (typically soft) switches may be able to support TTPs without rewriting the switch, except, for example, adding the ability to negotiate and confirm support of specific TTPs by identifier. 
4. Some buyers may want products that include an ability to check control messages to ensure that conformance to the active TTP. This “debug mode” functionality would require specific per-TTP coding. Although some efforts are already underway to mechanically generate such message conformance check code, new TTPs would still impact switch-based code.

##<a name="Are Multiple Flow Table use cases that important?"></a>Are Multiple Flow Table use cases that important?
Making multi-table use cases more practical and interoperable was one of the primary motivators for the development of the TTP framework… But the value of those complex use cases is not always immediately clear.  Shouldn’t we just stick with solutions based on a single flow table? Not necessarily.

There are many use cases for which a single flow table cannot be used or can only be used in an inefficient sub-optimal way.

* Cross-product rule explosion happens when many combinations of exact match fields are required. One example is a router forwarding table where multiple copies may be required that differ only in the router MAC address. Multiple tables allow factoring into two tables, one for the router MAC addresses and a second for the route table.
* Protocols such as MPLS, where nested headers may need to be processed in order to determine packet disposition but OpenFlow only allows matching on the outermost header. The same applies to Q-in-Q VLAN tags. 
* Not specifically for tables but applicable to group tables, multicast replication and multipath output selection using groups can substantially reduce both the number of flow entries required and the number of interactions with the controller to place flows.

These use cases fall into two general categories which benefit from multiple flow tables: 

1. __Independent Action scenarios__. Some scenarios require that separate actions be taken based on different fields.  For example, we may want to forward packets based on the packet’s destination MAC address, and at the same time copy the packet to the controller when it comes from an unknown source MAC address.  Such scenarios are unrealistic to support in a single table using today’s OpenFlow.  Consider that a two table solution could easily support 4000 MAC addresses through the implementation of two tables, each with 4000 entries.  Attempting to support this in a single table might require millions of entries, which require significant switch resources as well as imply huge numbers of control messages.  There are many other similar scenarios where independent actions will require multiple tables.
2. __Multi-stage processing scenarios__.  Some scenarios require multiple stages of processing, which are best done in two (or more) tables.  For example, assigning VIDs by ingress port, then checking for “router VID/MAC”, followed by either switching (when no router VID/MAC match) or routing (when destination MAC is recognized as a router MAC).  As with the independent action scenarios above, attempting to implement these schemes in a single flow table is unrealistic.  The number of entries required will multiply relative to an implementation that can leverage multiple tables. 

As SDN adoption advances, the complexity of interesting scenarios will increase.  The ability to use multiple flow tables will enable scalable deployments.  Multiple flow tables may also simplify the decoupling of functions, so that one application may effectively control one flow table, while another application controls another.  

Developers may already be aware that their single table solution architecture has limited sustainability.  The availability of TTPs can provide a path forward.  Simple first generation applications may be built and even deployed using a single flow table.  Next generation applications can be prototyped in small scale environments using one flow table without TTPs, then recast into scalable multi-table solutions leveraging TTPs as the TTP ecosystem matures.

Are there any TTP-related Tools available? 

The TTPs written so far by the Forwarding Abstractions Working Group use JSON as the representation format, and many tools are available for JSON editing. In addition, some development has begun on tools for TTP validation (checking that a TTP complies with the spec and is fully self-consistent).  The Forwarding Abstractions Working Group and other TTP backers are pursuing additional tools, such as these:

* TTP-based simulation tools that can verify the intended behavior of a solution pipeline prior to deployment.
* TTP-based test stimulation generators that (with user guidance) can produce OpenFlow messages and packets for testing a device
* Ability to generate a TTP based on captured (e.g. Wireshark captures) OpenFlow messages
* TTP visualizer, that can generate a graphic representation of some aspects of a TTP
* TTP equivalence checks that check two TTPs and determine if they are equivalent, or if one is a subset of another

Please contact FAWG regarding TTP-related tools if you have interest in helping develop or define a new tool, or to request a status update on these tools.

##<a name="Are TTP files sent over the wire?"></a>Are TTP files sent over the wire?
No.
The current TTP framework does not describe any situation where OpenFlow products would pass TTP descriptions "over the wire". However, there will likely be scenarios in the future where such transfer will become interesting. For now, experimenter extensions to the OF­Switch protocol offer one way to implement the capability for providers that have interest in features that could benefit from over-the-wire TTP transfer.

##<a name="How do TTPs impact conformance testing?"></a>How do TTPs impact conformance testing?
They can provide a model against which conformance can be assessed.  Given the diversity of devices that ONF members seek to control using OpenFlow, it is impossible to identify a single set of tests that adequately captures the conformance of all OpenFlow-enabled devices.  And yet there is a strong needTTPs provide an unambiguous form for describing switch-based functionality.

##<a name="How does one create a TTP?"></a>How does one create a TTP?
A TTP describes, in OpenFlow terms, a set of switch behavior.  The first step in creating a TTP is, therefore, determining what OpenFlow-based switch behavior needs to be captured.  As mentioned earlier, there may be several paths to this set of behavior:

1. Switch roles and behavior can be determined as part of a detailed network architecture
2. Switch behavior might be defined in terms of familiar and/or standardized historical network functions such as L2 switching or L3 routing
3. The switch behavior may be an abstract representation exposing existing switch platform functionality in OpenFlow-compatible terms.
  a The Broadcom OF-DPA is an example of device-based switch behavior exposed in OF-compatible terms
4. The switch behavior may be a specific subset of some existing TTP chosen to be supportable by a particular device
  a For example, a device might support a subset of Broadcom’s OF-DPA.  Defining a TTP as a subset of OF-DPA implies that the Broadcom pipeline should also be able to support the new TTP.  Thus the new TTP might be attractive to develop on if it can be supported on many devices.

Whatever the mechanism for identifying the detailed OF-switch behavior, once it is determined, the necessary information for creating the TTP should all be known.  The next step is to represent it in the format called for in the OF-TTP specification.  At this point there are a few options.  You could type the TTP by hand, and that’s how early TTPs were developed. But because TTPs have common structures, it is much easier to begin from an existing TTP, and use copy/paste/edit liberally to get the right number of necessary structures.  Simple json syntax checkers can ensure that your resulting file is legal JSON, and upcoming tools will check that the legal JSON is also a legal TTP.  And voila, you have your first TTP.

##<a name="Is there a repository for TTPs?"></a>Is there a repository for TTPs?
The ONF has created a [GitHub TTP repository](https://github.com/OpenNetworkingFoundation/TTP_Repository) for hosting and  ONF-published TTPs and related material, as well as TTPs published by other owners.

##<a name="What controllers will support TTPs?"></a>What controllers will support TTPs?
As of this writing, the OpenDaylight SDN controller project has established [a TTP project](https://wiki.opendaylight.org/view/Table_Type_Patterns:Main) for adding several kinds of TTP functionality.  ONF and FAWG are eager to provide support to other controller efforts to support TTPs.
 

