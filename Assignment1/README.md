# Building a Datawarehouse from legacy OLTP

Approach Framework
Phase 0: Strategic Foundation (Days 1-2)
Goal: Make informed decisions, not assumed ones

OLTP Analysis → Understand business processes from schema structure

You've started this (table relationships, schema inventory)
Next: Map transactional events, not just entity relationships
Key output: 5-7 distinct business processes with clear transaction descriptions


Bus Matrix → Force yourself to think across processes

Which dimensions are truly conformed? (Customer across Sales & Service)
Which are process-specific? (Manufacturing dimensions won't cross to HR)
This step prevents building a silo


Prioritization Matrix → Justify why you pick Sales (or whatever you pick)

Score: Business Impact, Data Readiness, Feasibility, Stakeholder Demand, Architectural Value
Don't assume Sales is obvious - run the scoring, document the reasoning
Portfolio value: shows strategic thinking, not just execution skill


Architecture Decision → One clear diagram + rationale paragraph

OLTP → Staging → Dimensional Mart → Analytics
State: "Pure Kimball for this scope because [X]. Would add integration layer if [Y]."



Phase 1: Dimensional Design (Days 3-4)
Critical: Follow Kimball's 4-step process religiously
This isn't bureaucracy - it's demonstrating methodology mastery:

Business Process → "We're modeling Sales Orders, not 'sales data'"
Grain Declaration → "One row = one line item on one order as of ship date" (be this specific)
Dimensions → Customer, Product, Territory, Employee, Date (choose SCD types with business justification)
Facts → Quantity, Revenue, Cost, Margin (classify additivity explicitly)

Portfolio gold: A well-justified SCD Type 2 decision shows more depth than a perfectly normalized schema.
Phase 2: Implementation (Days 5-6)
Priorities:

Data reconciliation → Your fact table sums must match OLTP exactly
One SCD Type 2 example → Demonstrates you understand version history, effective dates
Clean, commented SQL → Reviewers read code for clarity, not cleverness

Don't over-engineer:

Skip complex orchestration (Airflow) unless it's a differentiator you want
Focus on correctness and clarity over production robustness

Phase 3: Validation & Story (Day 7)
Make the business case:

OLTP query: 8-way join with subqueries
DW query: 3-table star schema join
Results: identical, but 10x faster, 5x fewer I/O operations

Reflection essay is underrated:

This is where you show you understand why, not just how
Discuss what you'd do differently at enterprise scale
Acknowledge trade-offs you made for learning value

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
