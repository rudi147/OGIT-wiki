# Real life use case examples

Here we present some real life use cases based on the OGIT ontology.
[comment]: # (arago prefix is used, but neither introduced nor explained)

### Use case 1: Ticket Statistics

#### Problem description

A user wants to compile various statistics on Tickets: Find out who was the responsible agent, when and what he did,
which authorizations were required, how long it took to solve the problem. Also have a look at the request side - find out
who was the requester, which applications/components were affected. 

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| User | ogit:Person | entity | |
| Tickets | ogit.ITSM:Incident, ogit.ITSM:ChangeRequest, ... | entity | |
| to be responsible | ogit.ITSM:assignedTo | verb | |
| responsible agent | ogit:Person, ogit:Organziation | entity | individual or group of individuals |
| actions: when and what? | *n/a* | | |
| authorizations | ogit.ITSM:ApprovalTask - ogit:belongsTo -> ogit.ITSM:ChangeRequest, ... | entity, verb | |
| when was it reported | ogit.ITSM:reportedAt | attribute | specific for ogit.ITSM:Incident, other ticket types might use other fields, e.g., ogit.ITSM:openedAt |
| when was it resolved | ogit.ITSM:resolvedAt | attribute | specific for ogit.ITSM:Incident, other ticket types might use other fields, e.g., ogit.ITSM:closedAt |
| when was it closed | ogit.ITSM:Ticket - ogit.ITSM:closedBy -> ogit:Person | entity, verb | |
| requester | ogit.ITSM:requestedBy -> ogit:Person, ogit:Organization  | verb, entity| specific for ogit.ITSM:ChangeRequest |
| reporter | ogit.ITSM:reportedBy -> ogit:Person | verb, entity | specific for ogit.ITSM:Incident, ogit.ITSM:Problem |
| affected components | ogit.ITSM:reportedOn -> arago:MARSNode | verb, entity | primary CI of some incident or problem |
| affected components | arago:MARSNode <- ogit:affectedBy | verb, entity | secondary CIs of some incident or problem or CIs possibly affected by planned change |

#### Sample query

As an example we show the query needed for one specific question from our problem domain:

*Show all unresolved incident records reported by user 'sample@x.com'.*

In a Tinkerpop-based implementation, this can be done using the following Gremlin query:
```
inE.has('label','ogit.ITSM:reportedBy').outV.or(_().hasNot('ogit.ITSM:resolvedAt), _().has('ogit.ITSM:resolvedAt', ''))
```
anchored at `root=sample@x.com`.


### Use case 2: Provider Management

#### Problem description

A user wants to perform Provider Management: The user needs to know which component has which type of contract. Details
about the contracts like terms and conditions, SLAs, cost structure, penalties including history, contacts, escalation
procedures and vendors are accessed.

#### Mapping to OGIT data


| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| User, Contacts | ogit:Person, ogit:Organization | entity | individual or group of individuals |
| Component | ogit.ITSM:Asset, ogit.Software:Application | entity | |
| which Asset to which Contract | ogit.ITSM:Asset - ogit:coveredBy -> ogit:Contract | entity, verb | |
| Contract | ogit:Contract | entity | |
| which component has which type of contract | ogit.ITSM:Asset - ogit:coveredBy -> ogit:Contract | entity, verb | |
| type of contract | ogit:contractType | attribute | specifies the type of an ogit:Contract entity, e.g., "HardwareContract" |
| SLA | ogit.ITService:SLA | entity | |
| cost structure | ogit.CostModel:`*` | several entities | e.g., ogit.CostModel:Budget, ogit.CostModel:CostElement |
| penalties | ogit:Penalty | entity | |
| vendors | ogit:Organization | entity | |
| terms and conditions | ogit.ITSM:License | entity | |
| which contract of which vendor | ogit:Contract - ogit:belongsTo -> ogit:Organization | entity, verb | |
| details of contract | ogit:Contract - ogit:contains => ogit.ITSM:License, ogit.ITService:SLA | entity, verb | | 
| which penalties | ogit:Contract - ogit:defines -> ogit:Penalty | entity, verb | |
| which SLA triggers penalties | ogit.ITService:SLA - ogit:triggers -> ogit:Penalty | entity, verb | |
| ownership relations | ogit:Person - ogit:owns -> ogit.ITService:Service, ogit:Person - ogit:owns -> ogit.Software:Application, ogit:Organization - ogit:owns ->  ogit.ITSM:Asset | entity, verb | |

### Use case 3: Network Planning

#### Problem description

A user wants to plan technical networking architecture. Therefore he needs information about the network and application
structure and usage (throughput, structure, error rates). He also needs information from the vendor side about contracts
and service.

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| User | ogit:Person | entity | |
| networking architecture | ogit.Network:`*` | entity | several entities |
| vendors | ogit:Organization | entity | |
| Contract | ogit:Contract | entity | |
| SLA | ogit.ITService:SLA | entity | |
| Asset | ogit.ITSM:Asset | entity | |
| network and application structure I | ogit.Network:Port - ogit:connectsTo -> ogit.Network:NIC, ogit.Network:FCHBA | entity, verb | |
| network and application structure II | ogit.Network:Router - ogit:extends -> ogit.Network:SimpleDevice, ogit.Network:Card | entity, verb | |
| network and application structure III | ogit.Network:Shelf - ogit:has -> ogit.Network:Slot, ogit.Network:SimpleDevice - ogit:has -> ogit.Network:Port, ogit.Network:Card - ogit:has -> ogit.Network:Port | entity, verb | |

### Use case 4: Compare Architectures

#### Problem description

User wants to compare architectures and need to observe which software is used and how the software is used (users,
transaction rates, user satisfaction, errors, service disruptions.

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| User | ogit:Person | entity | |
| Software | ogit.Software:Application | |
| transaction rates | ogit.Software:Application - ogit:hasTimeseries -> ogit:Timeseries | entity, verb | |
| observe which software is used | ogit.Software:Application - ogit:appearsAs <- arago:MARSNode | entity, verb | |
| observe which software is used | ogit.Software:Application - ogit:uses <- ogit:Person | entity, verb | |
| how the software is used | ogit.Software:Application - ogit:responsibleFor <- ogit:Person | entity, verb | responsibility of the software |
| how the software is used | ogit.Software:Application - ogit:owns <- ogit:Person | entity, verb | owner of the software |
| users, transaction rates, user satisfaction, errors, service disruptions | see use case 1 | entity, verb | see Ticket stuff of use case 1 |

### Use case 5: Forecasting Technological Trends

#### Problem description

A user wants to forecast technological trends. He wants to look at which technology is used by whom, how efficiently
technology is used, how a technology is related to others, who has delivered and serviced the technology. If this data
would exist as historical data, he wants to find out who are the "early adopters" and who are the "trend spotters".

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| User | ogit:Person | entity | |
| technology | ogit.Software:Application | entity | |
| is used by whom | ogit.Software:Application - ogit:uses <- ogit:Person | entity, verb | |
| is installed by whom | ogit.Software:Application - ogit:installedBy -> ogit:Person | entity, verb | |
| exist as historical data | ogit.Software:Application - ogit:hasTimeseries -> ogit:Timeseries | entity, verb | |
| historical data | ogit:Timeseries | entity | |

### Use case 6: Data Center Planning

#### Problem description

A user wants to analyze existing and plan new data centers, therefore he needs geographical and architectural
information, information about HVAC, networking, data center assets including contract information but also information
about the security requirements of the applications and the responsible persons and organisations.

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| User | ogit:Person | entity | |
| Data Center | ogit.DCIM:Datacenter | entity | |
| more Data Center stuff | ogit.DCIM:`*` e.g., ogit.DCIM:Building | entity | |
| geographical information | ogit.DCIM:Datacenter - ogit:locatedIn -> ogit.gr:Location | entity, verb | |
| geographical information | ogit.gr:Location | entity | |
| architectural information | ogit.DCIM:`*` e.g., ogit.DCIM:Building, ogit.DCIM:Room, ogit.DCIM:Rack, ogit.DCIM:Cooling … | entity | |
| architectural information | ogit.DCIM:Datacenter - ogit:has -> ogit.DCIM:Building, ogit.DCIM:Building - ogit:has -> ogit.DCIM:Room, ogit.DCIM:Room - ogit:has -> ogit.DCIM:Section | entity, verb | and more "ogit:has" relations |
| Datacenter assets | ogit.ITSM:Asset | entity |
| contract information | ogit.ITSM:Asset - ogit:coveredBy -> ogit:Contract | |
| responsible persons | ogit.DCIM:Datacenter - ogit:responsibleFor <- ogit:Person | entity, verb | |
| responsible organisations | ogit.DCIM:Datacenter - ogit:manages <- ogit.gr:BusinessEntity - ogit:appearsAs <- ogit:Organization | entity, verb | |


### Use case 7: Application Migration

#### Problem description

A user wants to migrate applications from one infrastructure to another. He needs to find out which components are
required, which infrastructure capacities are used, if they are shared or dedicated. Further he wants to know who is
operating/servicing the applications and their components, under which contract terms and conditions.

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| User | ogit:Person | entity | |
| asset, infrastructure | ogit.ITSM:Asset | entity | |
| infrastructure nodes | arago:MARSNode | entity | |
| applications | ogit.Software:Application | entity | |
| describe infrastructure I | ogit.ITSM:Asset - ogit:describes -> arago:MARSNode, ogit.Network:Fabric, ogit.Network:SimpleDevice, ogit.Network:Card, ogit.Network:Enclosure, ogit.DCIM:Cooling, ogit.DCIM:Rack, ogit.DCIM:Device, ogit.DCIM:PDU | entity, verb | |
| describe infrastructure II | arago:MARSNode - ogit:appearsAs -> ogit.Software:Application, ogit.Software:Application - ogit:consistsOf -> ogit.Software:Component | entity, verb | |
| contract terms | ogit.ITSM:Asset - ogit:coveredBy -> ogit:Contract | entity, verb | |
| who is operating/servicing the application I | ogit.Software:Application - ogit:belongsTo -> ogit.ITService:SLA | entity, verb | |
| who is operating/servicing the application II | ogit:Person - ogit:responsibleFor -> ogit.Software:Application, ogit:Organization - ogit:manages -> ogit.Software:Application | entity, verb | |


### Use case 8: Technical Analysis of Business Processes

#### Problem description

A user wants to analyze business processes from a technical perspective. He needs to find out the relationships between
technology and business processes, esp. the usage of technology. Also he needs to learn about the business flow and
which applications, components and services are required for the business flow. Here he wants to understand who is
using the processes, which data are processes, at which rates and what results are produced. In addition the user wants
to learn what people and IT services are involved, what contracts exist and what the financial flows are.

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| User | ogit:Person | entity | |
| business processes | ogit.BPMN2:Process | entity | |
| Contract | ogit:Contract | entity | |
| Application | ogit.Software:Application | entity | |
| Service | ogit.ITService:Service | entity | |
| business process parts | ogit.BPMN2:SubProcess, ogit.BPMN2:ProcessPool, ogit.BPMN2:ProcessLane, ogit.BPMN2:Gateway, ogit.BPMN2:TextAnnotation, ogit.BPMN2:Event, ogit.BPMN2:DataObject, ogit.BPMN2:Group, ogit.BPMN2:Task, ogit.BPMN2:DataStore| entity | |
| relationships between technology and business processes I | ogit.BPMN2:Process - ogit:supports -> ogit.Software:Application, ogit.Software:Component | entity, verb | |
| relationships between technology and business processes II | ogit.BPMN2:Process - ogit:consistsOf -> ogit.Software:Application, ogit.Software:Component | entity, verb | |
| services are required | ogit.BPMN2:Task - ogit:uses -> ogit.ITService:Service | entity, verb | |
| which data are processed I | ogit.BPMN2:Process - ogit:uses -> ogit.BPMN2:DataObject | entity, verb | |
| which data are processed II | ogit.BPMN2:Process - ogit.BPMN2:read, ogit.BPMN2:write -> ogit.BPMN2:DataStore | entity, verb | |
| what contracts exist | ogit.BPMN2:Task - ogit:uses -> ogit:Contract | entity, verb | |


### Use case 9: IT Organisation Insights

#### Problem description

A user wants to gain insights into current IT organization, i.e., who is involved from the technical and the process
perspectives. What are the reporting and communication flows, what are the dependencies and contractual arrangement between
involved organizational units.

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| Person | ogit:Person | entity | |
| IT Management Role | ogit:Role | entity | |
| what is IT Management I | ogit:Person - ogit:has -> ogit:Role | entity, verb | |
| what is IT Management II | ogit:Organization - ogit:defines -> ogit:Role | entity, verb | |
| what is IT Management III | ogit:Person - ogit:manages -> ogit:Organization, ogit.Software:Application | entity, verb | |
| IT organisation | ogit:Organization | entity | |
| Contract | ogit:Contract | entity | |
| who is involved | ogit:Person - ogit:isMemberOf -> ogit:Organization | entity, verb | |
| dependencies and contractual arrangement between involved organisational units I | ogit:Organization - ogit:dependsOn -> ogit:Organization | entity, verb | |
| dependencies and contractual arrangement between involved organisational units II | ogit:Contract - ogit:belongsTo -> ogit:Organization | entity, verb | |
| dependencies and contractual arrangement between involved organisational units III | ogit:Organization - ogit:includes -> ogit:Organization | entity, verb | |
| reporting and communication flows | ogit:Organization - ogit:communicatesWith -> ogit:Organization | entity, verb | |


### Use case 10: User Ranking

#### Problem description

IT/Business wants to identify the "best" and "worst" users of a certain application. They want to find out which
application has the highest number of incidents, changes and complaints, which users are reporting tickets using which
channel. On the other hand they want to find out which agent is solving/processing the tickets/changes. For each (users,
agents and applications) they want to find out the responsibilities within the organisation and which contractors are
involved.

#### Mapping to OGIT data

| natural language term | OGIT element | OGIT type | remark |
| --- | --- | --- | --- |
| Person | ogit:Person | entity | |
| IT/Business Role | ogit:Role | entity | |
| what is IT/Business | ogit:Person - ogit:has -> ogit:Role | entity, verb | |
| application | ogit.Software:Application | entity | |
| who uses the application | ogit:Person - ogit:uses -> ogit.Software:Application | entity, verb | |
| who is operating/servicing the application I | ogit.Software:Application - ogit:belongsTo -> ogit.ITService:SLA | entity, verb | |
| who is operating/servicing the application II | ogit:Person - ogit:responsibleFor -> ogit.Software:Application, ogit:Organization - ogit:manages -> ogit.Software:Application | entity, verb | |
| time series of an application | ogit.Software:Application - ogit:hasTimeseries -> ogit:Timeseries | entity, verb | |
| number of incidents, changes and complaints, which users are reporting tickets using which channel, agent is solving/processing the tickets/changes | see use case 1 | entity, verb | all Ticket stuff |
