# Building a Datawarehouse from legacy OLTP


### Key considerations

1. Identify high-value tables
High value for OLTP means *frequently joined tables* (high FK degree)

Two key indicators
- Row count => table volume/size
- Foreign key degree => Referential complexity (connectivity in the relationship graph)
- Business criticality

High-value tables usually have one or more of:
Signal, Why it matters
High row count, Operational importance
Large size, Storage / performance impact
Many incoming FKs, Core business entity
Frequent updates, Business activity
Time-series growth, Event / fact table

Rule of thumbs:
Pattern
Likely table type
High rows + low incoming FKs
Fact / event / log
Low rows + high incoming FKs
Dimension / reference
High rows + high incoming FKs
Very high-value (core)
Low rows + no FKs
Config / lookup / legacy


Clarification:

Data density
- Completeness density => ratio: non-nulls/total values)
- Row density => "how many rows/records a table has" (nothing to do with nulls)
