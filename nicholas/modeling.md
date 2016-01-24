# Modeling in Color

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

- accelerated domain modeling and understanding: archetypes allow you to quickly dissect a domain, leverage an existing meta-model to understand new domains, understand new domains by leverage examples for unrelated domains (e.g. this is like a partial shipment in an commerce domain)
- foundational communication tool: provides a common language to discuss domain models with your team members, similar benefits as "design patterns" provides
- agile collaboration tool: the format (e.g. color) lends itself to low-fidelity modeling techniques using post-it notes
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

## Archetypes

- "a form which all things of the same kind more or less follow" (JMICWU pg 2)
- the form being an object's responsibilities:
  - what I know
  - who I know
  - what I do
- in other words:
  - attributes
  - collaborations (links)
  - methods
  - plug-in points
  - interactions

## Events

- examples:
  - registration
  - order
  - reservation
  - subscription
- types
  - discrete
  - evolving
  - recurring
- patterns
  - follow-up
  - composite

## Roles

- Roles model the behavior of an Entity collaborating with an Event
- may also have attributes
- one of the most important archetypes in avoiding god-objects by bloating Entities with responsibilities

## Entities

- Entities play Roles
- Coad refer to Roles as hats Entities wear
- Entities can play many Roles
- parties (people, organizations), aka actors
- places
- things

## Descriptions

- "catalog-entry like description"
- toughest archetype to differentiate from Entities
- my favorite example to help understand the difference: product_description#sku vs. product#serial_number
- Entities are uniquely identifiable; e.g. product#serial_number
- Descriptions describe a group of objects; e.g. product#sku
- hence Coad's use of "catalog-entry like description"
- provide run-time inheritance for Entities, e.g. the common attributes and methods
- can also describe Events

# Footnotes

[1] [Agile Data Warehouse Design](http://www.amazon.com/Agile-Data-Warehouse-Design-Collaborative/dp/0956817203/ref=sr_1_1?s=books&ie=UTF8&qid=1453645838&sr=1-1)