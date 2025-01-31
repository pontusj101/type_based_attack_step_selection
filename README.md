# Quasi-realistic agent design

The KTH simulator exposes attack graphs to agent programs. These agents can,
based on their internal policy, take an action at each time step of the
step-wise simulation. The internal policy can either be learned (e.g. using
Reinforcement Learning techniques) or hardcoded using rule-based methods. The
attack graph these agents work with is generated based on an instance model and
a MAL (Meta Attack Language) spec.

The Quasi-realistic agent (QR agent) is a rule-based agent that exhibits
a somewhat realistic behavior derived from real data. While it is hard to
quantify the degree of realism, statistical properties of real logs of attacks
can be extracted and then fitted into parameterized rules comprising the agent's
policy. For the QR agent, the set of rules will be defined at the meta level,
i.e. in the MAL file. The extent of realism of the QR agent will depend
on the granularity of the rules and how well and to what extent they can capture
the statistical properties of real logs.

## Rules and priority

Rules consist of three parts: a query, a target step name and a priority; rules
apply the priority to the set of attack steps that match the query.

Queries in the MAL file will be defined in the relational domain using 1st order
logic. This definition will share similarities (operators and semantics) with
the existing step expression definition syntax already provided by "MAL 1.0"
(the current, unversioned MAL language specification being used). The existing
syntax supports set operations and limited selector logic; in particular,
transitive and type-of selections are supported and may need extension. Queries
will return a set of assets the target step of which will get the priority
defined in the rule. While rules are defined at the metalevel (in the MAL spec),
queries may use selectors that apply on the instance model (for example, queries
that search for assets with a fan-out larger than a number).

Priority is a real number. For our use case, we consider it to be a non-negative
real number; priority 0 will have the agent ignore corresponding steps and
higher priority means it is more probable for the agent to select matching
steps. Priority can also be termed preference. Priority needs to be translated
to probabilities. For this, the softmax function can be used on the
probabilities of the attack steps in the attack horizon.

The target step name gives the attack step in each of the returned assets
matched by the query that will be assigned the related priority. The target
attack step should exist in every asset returned.

Further:

- To better support rules, new selectors can be added. Also, new syntax to
  define prioritization may be needed. These will be backwards compatible MAL extensions.

- In case of multiple rules assigning different priorities to the same attack
  step on an asset, priority should be resolved. We consider using the max value
  assigned by any rule as the resolution method.

- Priority will be evaluated at runtime before attack graph state is passed to
  the agent. This will enable selectors such as "TTC-spent/left",
  "assets-with-%-of-attack-steps-compromised", etc.

- Priority rules suffice to describe agent policy. But, to fully describe an
  agent profile, objectives and resources should also be specified. For the
  latter, TTCs in the MAL spec do not suffice as these are system-dependent
  (e.g. throughput). For a proposal on defining agent skills see:
  https://github.com/pontusj101/type_based_attack_step_selection/blob/main/Separating_attacker_skills_from_TTCs.pdf.


## 1st iteration

- The new MAL syntax needs to be defined. The first implementation will provide
  queries that build on the existing MAL operators and selectors. For example,
  all "list" steps of an asset type will have a priority of 5, and all "get"
  steps will have a priority of 3.

- Support for the new syntax will be added to the MAL tool ecosystem.

- Conversion to probabilities will also be implemented.

## 2nd iteration

The goal of the 2nd iteration will be to extend the ruleset and its
expressiveness.

- We will extract the most important behaviors observed in real data.

- New syntax will be added to help write rules that can describe the above
  behaviors.

- We will fit parameters derived from the real data to rule parameters using
  log-likelihood or other methods. See:
  https://github.com/pontusj101/type_based_attack_step_selection/blob/main/type-based%20attack%20step%20selection.pdf

- The aspect of time should also be modeled.

## Ideas for next iterations

- Use of transformers for producing priorities

- Agent inheritance

## Evaluation of the QR agent

See https://github.com/pontusj101/type_based_attack_step_selection/blob/main/Comparing_agent_policies_from_logs.pdf
for a proposal on how to evaluate the realism of logs produced by the QR agent.

Real data will be collected observed pen-testers than real attacks.

For evaluation, the detector-extension of MAL (referred to as MAL 2.0) is needed
so that agent actions produce the synthetic logs that will be compared with the
real data.

## Issues open for discussion

- The exact syntax should be defined.

- Should it be possible to combine rules? For example, can a new rule reference
  another rule, use the latter's assigned priority and produce a new priority
  based on some calculation?
