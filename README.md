# DTN Reference Scenarios

This repository contains CCSDS DTN Reference Scenarios for use in testing,
validation, simulation, benchmarking, and evaluation of Delay/Disruption
Tolerant Networking technologies.

The scenarios are provided as open reference material using CSV and JSON
files, together with explanatory documentation.

## DTN Scenario Characteristics

###	Topology
The network topology contains the nodes and available links between those node. The topologies definitions are derived from publicly available information. A visual representation containing all the nodes of a specific scenario can aid understanding the scenario’s structure and can be used to prepare the simulation. The topology provides the basis for calculation of visibilities between assets and communication delays. Based on this visibility, the planned and actual communication contacts are derived.

###	Contacts
Contacts describe an available link from one node to another in terms of data rate and delay for a specific time period. Contacts are unidirectional and are assumed to be without errors. For simulation of errors, larger contacts are split into smaller contacts with some gap between those smaller contacts. 
For contacts, there are two different files that have different roles in the scenario execution. The Planned Contacts is the results of a planning process applied to the assets’ visibilities computed using a flight dynamics software [3] assuming that not all visibilities are utilised for communication purposes as communication assets, such as ground stations, may be shared across different users or mission objectives compete with communication time. The Planned Contacts describe the expected contact behaviour during the simulation. In a second step, an error model is applied to the Planned Contacts, to produce the Actual Contacts file that describes the real contacts which are used for communication. Here, planned contacts may be interrupted, become shorter or might be dropped entirely.
The planned contacts as expected to be used to inform Bundle Protocol when to initiate bundle forwarding via a specific (simulated) link. Planned contacts are computed starting from the flight dynamics software library [3], where asset coordinates and orbital data are inserted, to compute physical visibility and distance. After this first step, heuristic mission and ground station planning are applied to the results of the visibility computation to obtain what we call the Planned Contacts. 
While Planned Contacts inform a Bundle node when to forward bundles, the Actual Contacts determine whether this bundle is actually successfully received at the next bundle hop. This would typically be implemented by the simulation environment by dropping bundles if there is no contact in the Actual Contacts file. This file is derived from the Planned Contacts, by dropping, shortening or interrupting planes contacts based on some error model.

### Data Rates
Each contact defines a maximum data rate. This data rate is not the actual link usages, but the available maximum once the link has been established. The actual usage may be lower and depends on the application traffic volumes. For simplicity, scenarios may use the same data rate for all contacts between two assets. Data rates are defined within the Actual and Planned Contacts files for each contact.
For simulation purposes, data rates and application traffic volumes may be scaled down to facilitate simulation execution although such  might affect parameters such protocol overheads, processing or memory consumption and may lead to incorrect or less precise results. Such effects have to be carefully considered given the purpose of the individual simulation and any scaling applied should be well documented.

###	Communication Delay
The communication delay is based on the distance between to assets during a contact. For simplicity, current scenarios just consider the distance at the beginning of a contact. In addition, some fixed processing delay is added and a fixed value is assumed for terrestrial links. The communication delay for each contact is provided in the Actual and Planned Contacts files.
Simulations may be run faster or slower than real time. However, this may affect parameters like processing or memory consumption and may lead to incorrect or less precise results. Such effects have to be carefully considered given the purpose of the individual simulation and the relation between simulation and real-time should be well documented.

###	Link Error Model
A link error model is necessary to simulate data losses in the scenarios. The model is applied to every contact to determine how the real link availability is, during scenario execution. A simple probability error model with four possible outcomes for a single contact is defined: 
1.	No error – all bundles are transmitted successfully during a contact.
2.	Lost contact (P<sub>l</sub>) – no bundles are transmitted successfully during a contact.
3.	Burst errors during a single contact (P<sub>b</sub>) – a random number of burst errors with a random duration are occurring during a contact. Minimum and maximum values for the number of errors, the duration and eventual additional constraints are specified per scenario.
4.	Partial contact (late start and/or early end) (p<sub>p</sub>) – a contact starts lates or ends early. Minumum and maximum values for late start / early end and any additional constraints are specified per scenario.

The link error model does not represent only errors related to the physical properties of the link, typically expressed in bit or frame error rates and caused by e.g., atmospheric attenuation or weather effects; it also includes link interruptions or complete failure to acquire a link due to factors such as human errors, misconfiguration of communicating assets or pointing errors. The rationale is that for DTN simulations the loss of a bundle matters and not the exact reason for that loss. However, for more realistic simulations it is expected that more specific error models may be applied directly in the simulation environment, eventually without making use of the Actual Contact files.
The lost contact probability considers any error that cause a complete loss of contact. The causes can be misalignment or misconfiguration. For some contacts with very delayed start the pass can be considered completely lost for certain missions, for example if the remaining contact time is less than a threshold set by the mission.
The burst errors model is a simple error approximation to insert sporadic data loss during a contact. During real operations this kind of loss may be caused by various factors such as solar flare, weather event, interference, etc., and they are all modelled under a single probability (P<sub>b</sub>) of such event occurring and causing a burst of link unavailability. The probability are provided within the scenarios files as they depend on the scenario itself.
By applying these link error probabilities to the Planned Contacts the Actual Contacts available during the simulation are determined. Each scenario provide its own probability for the four aforementioned cases based on a simplification of empirical observations for some missions and provides motivations for the provided values. 
In cases where a more sophisticated error model is required, the users can ignore the Actual Contacts file and include dynamic runtime error directly in their simulation. Alternatively, they can derive a new Actual Contacts file applying their own heuristics to the Planned Contacts provided.

###	Application Traffic Types, Volumes and Policy
The scenarios define the generation of traffic between nodes in terms of frequency, size and other parameters in the Traffic file. A Policy file may restrict certain traffic to certain links, so that for example high-volume science data can only be transmitted on links with high data rates. 


## Files Format for Characteristics Representation

### Nodes file
The **node.json** file is composed by an array of elements with the following String fields:
-	***node_label:*** unique label used in contact files to reference this node
-	***node_name:*** descriptive node name for user convenience.
-	***node_id:*** BP Node Id in the form of an endpoint ID.

### Planned contacts, Actual contacts and visibility
The **planned_contacts.csv**, **actual_contacts.csv** and **visibility.csv**, are CSV files using the same format. They contain for every line a directional link description with the following format:

```
<source>,<destination>,<contact_start>,<contact_end>,<data_rate>,<delay>,<link_type>
```
The fields description are reported here :
-	***source:*** the link source node identifier, both source and destination are node labels mapped in the Nodes files to node IDs.
-	***destination:*** the link destination node identifier.
-	***contact_start(s):*** the start time of the contact, in seconds, relative to the start of the simulation time (zero).
-	***contact_end(s):*** the end time of the contact, in seconds, relative to the start of the simulation time (zero) or the value “-1” if the contact last until the end of the simulation time.
-	***data_rate(bps):*** the amount of bits that can flow through a link during every second. It indirectly set the amount of bundles that can flow on the link at any given time.
-	***delay(s):*** the link delay in seconds, calculated using the physical distance of the two assets plus a processing delay.
-	***link_type:*** label that can be used in the policy file to route specific application data over specific link types, e.g. using S-Band for housekeeping downlink and Ka-Band for science data; both links may be active at the same time.

### Traffic
The **traffic.json** file defines the traffic generation and traffic flow in the simulated network, from a source node id to a destination EID. The traffic file is composed by the following parameters:
-	***info:*** an informational field aiding the user in the file configuration.
-	***src:*** the source node id from which the generated traffic is transmitted.
-	***dst:*** the destination endpoint id to which the generated traffic shall be delivered.
-	***start_time:*** the data generation start time in seconds, relative to the simulation start time.
-	***end_time:*** the data generation end time in seconds, relative to the simulation start time or the value “-1” if the data generation last until the end of the simulation time.
-	***adu_size:*** the size of the bundle generated by this source endpoint id.
-	***interval:*** the rate of bundle generation in seconds.
-	***ttl:*** the generated bundles time to live in seconds.

The info field additional information have this format:
```
<Traffic type> <source node_label> -> <destination node_label>
```
Although just informational, the field can help having a quick overview of the traffic generated without looking up at the Nodes file.

### Policy
The **policy.json** defines policies regarding link usages. The policy file is composed by the following parameters:
-	***info:*** an informational field aiding the user in the file configuration.
-	***policy_target:*** the target node of this policy, i.e. the node executing the policy
-	***src:*** the source Node ID of the traffic the policy is applied to.
-	***dst:*** the destination EID of the traffic the policy is applied to.
-	***link_restrictions:*** defines which link types as defined by link_type in the contact files can be used for this particular traffic source and destination. If empty, any link type can be used.

The info field additional information have this format:
```
<Traffic type>
```
It provides some labelling to the policy, to distinguish types of traffic.

## Versioning
The project is structured as follows:
```
ccsds-dtn-reference-scenarios
 │ 
 ├── low-earth-orbit
 │ └── v<Major>.<Minor>.<Patch>
 │   ├── Description file
 │   ├── Nodes file
 │   ├── Visibility file
 │   ├── Planned Contact file
 │   ├── Actual Contact file
 │   ├── Traffic file
 │   └── Policy file
 │
 ├── lunar-communication
 │ └── v<Major>.<Minor>.<Patch>
 │   ├── Description file
 │   ├── Nodes file
 │   ├── Visibility file
 │   ├── Planned Contact file
 │   ├── Actual Contact file
 │   ├── Traffic file
 │   └── Policy file
 │
 └── mars-communication
   └── v<Major>.<Minor>.<Patch>
     ├── Description file
     ├── Nodes file
     ├── Visibility file
     ├── Planned Contact file
     ├── Actual Contact file
     ├── Traffic file
     └── Policy file
```
Additional scenarios may be added in the future. For each scenario multiple versions of a scenario may exist within different folders. The main branch of the repository is the “release” branch. The HEAD will contain all scenarios and all folders with the different scenario versions released so far. Any modifications and proposal for new version will be added in a “draft” branch, that can be merged into the “release branch” after CCSDS DTN Working Group approval. Any change to a scenario will result in a new version folder in the repository so that old versions are still available in the working tree if checked out. 
Any major change to a scenario (e.g. topology changes) will result in a new major version, any minor change (e.g. parameters updates) will results in a new minor version and any error corrections will result in a new patch change. When a new Major or Minor version is created a new folder will be added to the repository with the updated files and versioning, e.g. version v1.0.0 and v1.1.0 will have two separate folders coexisting. When a new patch is released to correct errors, a new folder will substitute the previous patch, e.g. v1.0.0 can be substituted by v1.0.1, the two cannot coexist as patching v.1.0.0 means it contained an error in the provided data.

## References
[3]	R.A. Mackenzie, “GODOT Flight Dynamics Infrastructure Software for operations and analysis”, 1st European Workshop on Space Flight Dynamics Services, Systems and Operations, ESA-ESOC, Darmstadt, Germany, September 2021.


## License

The DTN Reference Scenarios are licensed under the [Creative Commons
Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)
(CC BY 4.0), unless otherwise stated.

Copyright © 2026 Consultative Committee for Space Data Systems (CCSDS).

This license applies to the scenario definition material in this repository,
including CSV files, JSON files, and accompanying explanatory text.

You may use, copy, modify, redistribute, and build upon these reference
scenarios for any purpose, including commercial use, provided that appropriate
attribution is given.

Suggested attribution:

> "DTN Reference Scenarios" by the Consultative Committee for Space Data
> Systems (CCSDS), licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

If you modify the scenarios, please indicate that changes were made.

Use of these reference scenarios does not imply endorsement by CCSDS,
its member agencies, contributors, or participating organisations.

The material is provided "as is", without warranty of any kind.
Users are responsible for assessing its suitability for their intended use.
