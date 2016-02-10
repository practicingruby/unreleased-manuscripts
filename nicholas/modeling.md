# Modeling in Color

## Next

- [ ] flesh out descriptions of archetypes
- [ ] interweave the example domain "Transplant Management" into the notes below.
- [ ] add sequence diagrams
- [ ] add business rules
- [ ] add triggers

## To-do items

Your deliverables (modeling benefits, list of strategies + related examples, informal overview of project)

- [ ] finished the diagram, so anywhere you have `-` maybe you can fill that in
- [ ] we need a list of “benefits” for the modeling technique, so you can piece that together.
- [ ] list of a handful of strategies as well as some descriptions of examples we can relate to in the domain model e.g.:
    - "Start by thinking through events"
    - "Figure out how to separate entities from descriptions"
- [ ] come up with two more, we’d be in good shape
- [ ] A high level overview of the system.  You have a small list of features, I’d want something along those lines but in a prose format. We need this to fill in the back story around the diagram and to give the reader an understanding of the context we’re working in.
## Tips:

- "Start with too much material, then give the reader just enough"

# Benefits

- accelerated domain modeling and understanding: archetypes allow you to quickly dissect a domain, leverage an existing meta-model to understand new domains, understand new domains by leverage examples from unrelated domains (e.g. this is like a partial shipment in an commerce domain)
- foundational communication tool: provides a common language to discuss domain models with your team members, similar benefits as "design patterns" provides
- agile collaboration tool: the format (e.g. color) lends itself to low-fidelity modeling techniques using post-it notes
- color makes the archetypes easy to learn
- allows to understand foreign and complex domains

# Key Strategies

- identify the Events - what are the "hot spots" (red) in the system
- use 5 "W's" (who, what, where, when, why) to drive discovery
- keep the domain model of e-commerce in my back pocket as it provides an excellent example of many different domain patterns that can be used as a reminder of those patterns

```
order       - shipment       - return
  |                |              |
order_items - shipment_items - return-items
```
- a tool to prevent myself from going down the rabbit hole, always ask "does the system need to remember this?"
- separate Entities from Descriptions

# Process

- stakeholders, product owner typically describe the domain with the key business transaction
  - order management
  - reservations
  - transplants
- business applications manage and/or automate those transactions
- the key business transaction (event) is our starting point for discovery
- the process outlined below is only a rough estimate, rarely happens in an "exact" order
- however, the idea of separating "discovery" from "refinement" I have found quite useful

## Iteration 1: Discovery

- determine the "why": the reason and causality for the key event, this helps to flesh out the other "W's"
- they "why" may end up as a "purpose" attribute on the Event (as an enumeration or free text)
- continue with identifying the "entities" collaborating in the event
- "who" (aka actors): who collaborates in the event (parties: people, organisations)
- "where": where does the even take place (place)
- "what": what is used (things)
- determine the nature of the event, "when": when does the event occur, is it discrete, evolving, or recurring[1] (moment-interval)
- event typically don't occur in isolation
- explore if preceding and proceeding events (forming the follow-up pattern)
- build up a timeline of events, determine the "why", identify the "entities" for each
- throughout add obvious attributes and behavior

## Iteration 2: Refinement

- extract Roles from entities
- Coad always models a "Person" explicitly
- then model the Roles they play in a system (see behaviour)
- identify/extract Descriptions from Entities
- often mistakenly model objects as Entities (things) when they have responsibilities that follow a Description's archetype
- determine less obvious attributes: lifecycle and operating states, event dates
- promote attributes to objects
- demote objects to attributes

## Iteration 3: Behavior

- determine business rules: what conditions must be met for objects to collaborate
- derive feature list (interactions)  
- test model with interactions


## Object Modeling

- what is it?
- when do I use it?
- why should I use it?

## Domain

- problem area with a boundary
- domain models
   - aka business objects, problem domain (PD) objects, models
   - specific concern
   - represent objects in the problem domain
- specific concern/set of responsibilities (see Archetypes)
  - not UI/View (they display PD objects)
  - not database (they persist PD objects)
  - embedded business rules for collaborations between PD objects

## Archetypes

- "a form which all things of the same kind more or less follow" (JMICWU pg 2)
- the form being an object's responsibilities (see summary below):
  - what I know
  - who I know
  - what I do
- in other words:
  - attributes
  - collaborations (links) (see the Domain Neutral Component)
  - methods
  - plug-in points
  - interactions
- responsibilities are not exactly the same, typically a sub-set, but may include others as well specific to the domain

### Typical Responsibilities (Attributes and Behaviour)

Illustrates the typical attributes and behavior archetypes have.

![Typical Responsibilities from http://www.step-10.com/SoftwareDesign/ModellingInColour/ ](./typical_responsibilities.png)


### Typical Collaborations

Illustrates the typical collaborations and interactions archetypes have.

![Typical collaborations from http://www.step-10.com/SoftwareDesign/ModellingInColour/ ](./typical_collaborations.png)

### Domain Neutral Component (DNC)

Events with the "what", "who", and "where". Note Entity "what", connected to the Event detail.

![Domain Neutral Component from http://www.step-10.com/SoftwareDesign/ModellingInColour/ ](./dnc.png)

- archetypal pattern for object models
- just a template
- remove classes with no responsibilities
- 1-1 associations indicate opportunity to combine classes


## Events (Pink)

- required to be recorded for business or legal purposes [Coad99
- "moment" in time or over an "interval" of time
- related to an action or activity
- other archetypes describe who, what, where
- interactions between entities (parties, places, things)
- examples (activity):
  - registration (registering)
  - order (ordering)
  - reservation (reserving)
  - subscription (subscribing)
- types
  - discrete
  - evolving
  - recurring
- patterns
  - follow-up
  - composite

## Roles (Yellow)

- Roles model the behavior of an Entity collaborating with an Event (i.e participation in an event)
- may also have attributes
- one of the most important archetypes in avoiding god-objects by reducing the responsibilities of Entities
- examples:
  - customer
  - sales person
  - employee


## Entities (Green)

- Entities play different roles
- Coad refer to Roles as hats Entities wear
- Entities can play many Roles
- parties (people, organizations), aka actors
- places
- things
- examples:
  - person
  - city
  - computer

> Another name sometimes suggested for this archetype is Entity but there are two arguments against using this term. Firstly it is already in use in the data-modelling world and although our parties, places and things naturally map to entities in data modellers' Entity-Relationship diagrams, the converse is not necessarily true. Secondly, the name Party, Place, Thing neatly reminds us that this archetype comes in three flavours, a fact that is of significance in the various analysis patterns involving class archetypes.

-- http://www.step-10.com/SoftwareDesign/ModellingInColour/

## Descriptions (Blue)

- "catalog-entry like description"
- blue "labels"
- the "knowledge" layer in the "knowledge-operation layers" MARTIN
- toughest archetype to differentiate from Entities
- my favorite example to help understand the difference: product_description#sku vs. product#serial_number
- Entities are uniquely identifiable; e.g. product#serial_number
- Descriptions describe a group of objects; e.g. product#sku
- hence Coad's use of "catalog-entry like description"
- provide run-time inheritance for Entities, e.g. the common attributes and methods
- can also describe Events
- examples:
  - product description

# Footnotes

[1] [Agile Data Warehouse Design](http://www.amazon.com/Agile-Data-Warehouse-Design-Collaborative/dp/0956817203/ref=sr_1_1?s=books&ie=UTF8&qid=1453645838&sr=1-1)
