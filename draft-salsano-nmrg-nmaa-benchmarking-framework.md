---
title: "A Framework for Controlled Benchmarking of Network Management AI Agents on Brownfield Networks"
abbrev: "NMAA Benchmarking Framework"
category: info

docname: draft-salsano-nmrg-nmaa-benchmarking-framework-latest
submissiontype: IRTF
number:
date:
consensus: false
v: 3
workgroup: Network Management Research Group
keyword:
 - AI agents
 - network management
 - benchmarking
 - brownfield
 - agent-network interface
venue:
  group: NMRG
  type: Research Group
  mail: nmrg@irtf.org
  arch: https://mailarchive.ietf.org/arch/browse/nmrg/
  github: netgroup/draft-nmaa-benchmarking-framework
  latest: https://netgroup.github.io/draft-nmaa-benchmarking-framework/draft-salsano-nmrg-nmaa-benchmarking-framework.html

author:
 -
    fullname: Stefano Salsano
    role: editor
    organization: University of Rome Tor Vergata / CNIT
    email: stefano.salsano@uniroma2.it
 -
    fullname: Wim Henderickx
    organization: Nokia
    email: wim.henderickx@nokia.com
 -
    fullname: Andreas Kassler
    organization: Deggendorf Institute of Technology

normative:

informative:
  RFC1242:
  RFC2544:
  RFC6241:
  RFC8040:
  RFC8342:
  RFC8993:
  RFC9315:
  I-D.cui-nmrg-llm-benchmark:
  I-D.contreras-bmwg-ai-agent-benchmarking:
  I-D.wmz-nmrg-agent-ndt-arch:
  I-D.mcw-opsawg-icon-requirements:
  I-D.wnd-opsawg-icon-ps:
  I-D.feng-netconf-naim-op:
  I-D.gaikwad-llm-benchmarking-terminology:
  I-D.gaikwad-llm-benchmarking-profiles:
  I-D.yang-nmrg-mcp-nm:
  I-D.zhao-nmop-network-management-agent:
  METHODOLOGY:
    title: "Benchmarking Methodology for Network Management AI Agents on Brownfield Networks"
    author:
      - name: Stefano Salsano
    date: false
    seriesinfo:
      Internet-Draft: draft-salsano-bmwg-nmaa-benchmarking-methodology-latest
  ANIGAMUT:
    title: "ANI-Gamut: Benchmarking Agent Reliability across the Gamut of Agent-Network Interface Abstractions"
    author:
      - name: Lorenzo Bracciale
      - name: Pierpaolo Loreti
      - name: Andrea Mayer
      - name: Stefano Salsano
      - name: Wim Henderickx
    date: 2026
    refcontent: "22nd International Conference on Network and Service Management (CNSM)"
  ANIGAMUT-EXT:
    title: "ANI-Gamut: Benchmarking Agent Reliability across the Gamut of Agent-Network Interface Abstractions (extended version)"
    author:
      - name: Lorenzo Bracciale
      - name: Pierpaolo Loreti
      - name: Andrea Mayer
      - name: Stefano Salsano
      - name: Wim Henderickx
    date: 2026
    target: https://arxiv.org/abs/2609.22723
  ANIGAMUT-CODE:
    title: "ANI-Gamut: playground, scenario generator and harness (release cnsm2026)"
    author:
      - org: netgroup, University of Rome Tor Vergata
    date: 2026
    target: https://github.com/netgroup/ani-gamut
  PLAYGROUND:
    title: "A Playground for Benchmarking Agentic AI in Network Management"
    author:
      - name: Stefano Salsano
    date: 2026
    refcontent: "Mediterranean Artificial Intelligence and Networking Conference (MAIN)"
  SWE-AGENT:
    title: "SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering"
    author:
      - name: John Yang
    date: 2024
    refcontent: "Advances in Neural Information Processing Systems (NeurIPS)"
  NETCONFEVAL:
    title: "NetConfEval: Can LLMs Facilitate Network Configuration?"
    author:
      - name: Changjie Wang
    date: 2024
    refcontent: "ACM CoNEXT"
  TMF-IG1252:
    title: "Autonomous Network Levels Evaluation Methodology (IG1252)"
    author:
      - org: TM Forum
    date: false
  TS28100:
    title: "Management and orchestration; Levels of autonomous network (TS 28.100)"
    author:
      - org: 3GPP
    date: false
  ETSI-ZSM:
    title: "Zero-touch network and Service Management (ZSM); Reference Architecture"
    author:
      - org: ETSI
    date: false

--- abstract

Large Language Model (LLM) based agents are being proposed to configure
and operate production networks. Whether a given Network Management AI
Agent (NMAA) system is safe to deploy depends not only on the model it
uses, but on the interface through which it acts, the way it is
orchestrated, the kind of request it receives, and the state of the
network it acts upon. This document describes a framework for
controlled benchmarking of NMAA systems. The system under test is
delimited by two interfaces: the human-agent interface, through which an
operator states a requirement, and the agent-network interface, through
which the system acts on the network. The framework varies one factor at
a time (the model, the abstraction level of the agent-network
interface, the agent mode, the task class) while holding the others
fixed, under explicit stress parameters (the density of pre-existing
configuration state and the rate of injected faults), on a brownfield
substrate where collateral damage to pre-existing services can actually
occur. It scores runs with a binary, zero-tolerance safety outcome and
treats statistics as part of the design. A companion document specifies
the corresponding benchmarking methodology.

--- middle

# Introduction {#intro}

Network operations are moving from commands typed by a human operator to
software agents, typically built around an LLM, that observe the running
network, change it, and confirm the result. Most of the attention in
this space has gone to the agent itself: its model, its prompting, its
planning loop. Several architectures for agentic network management have
been proposed {{I-D.wmz-nmrg-agent-ndt-arch}}
{{I-D.zhao-nmop-network-management-agent}}, and first evaluation
frameworks exist {{I-D.cui-nmrg-llm-benchmark}}
{{I-D.contreras-bmwg-ai-agent-benchmarking}}.

An operator who must decide whether to deploy such a system faces a
different set of questions. Which model should the system use for a
given class of tasks? Through which interface should it be allowed to
act: a device shell, a bounded command wrapper, model-driven
configuration, a typed service request, or an edit to a reconciled
source of truth? Does a planner-executor agent behave better than a
ReAct loop? Does the answer change when the network is densely
configured, or when faults occur while the agent acts? Each of these
questions is, in experimental terms, the same experiment with a
different treatment: the system under test is kept fixed except for one
factor, and the effect of that factor on success, safety and cost is
measured.

This document describes a framework that makes that structure explicit.
Its elements are:

* a reference model that delimits the system under test between the
  human-agent interface and the agent-network interface
  ({{model}});

* a set of controlled factors and stress parameters, and an
  experimental design (factorial, matched budget, paired runs) that
  compares settings of one factor while holding the others fixed
  ({{model}}, {{factors}});

* a brownfield substrate, densely populated with heterogeneous
  pre-existing services, together with a generator of reproducible
  initial states and the invariants that those services must keep
  satisfying ({{substrate}});

* a binary, zero-tolerance safety outcome, the damage-free run, with
  the statistics needed to bound near-zero damage probabilities
  ({{outcomes}}).

The emphasis on a brownfield substrate follows from the failure mode
that matters in operations. A configuration agent that completes its
task but silently breaks an unrelated, already-deployed service has
caused an outage. Collateral damage of this kind can only occur where
pre-existing services share resources with the target of the change:
the same devices, address space, identifier pools, or global settings.
A benchmark that runs agents on a nearly empty network cannot observe
it.

One of the controlled factors, the abstraction level of the
agent-network interface, deserves particular attention because it is
the factor that is most often fixed implicitly. In software
engineering, the interface offered to an agent, and not only the model
behind it, is known to govern how reliably the agent acts
{{SWE-AGENT}}. Operators are already climbing an abstraction ladder,
from device command line to model-driven configuration
({{RFC6241}}, {{RFC8342}}), to service-intent orchestration and
reconciled sources of truth. {{ani-levels}} describes this ladder as a
spectrum of levels that can be varied as a factor.

The framework is an outgrowth of an experimental study and of its
open-source implementation {{ANIGAMUT}} {{ANIGAMUT-EXT}}
{{ANIGAMUT-CODE}}, summarized in {{implementation}}. This document does
not report that study; it describes the framework so that other
implementations and other campaigns can be designed and compared
within it. The procedures, metrics definitions, statistics and reporting
requirements are specified in a companion benchmarking methodology
document {{METHODOLOGY}}, written in the style of the benchmarking
tradition of {{RFC1242}} and {{RFC2544}}.

This document is a product of individual contributors and is submitted
for discussion in the Network Management Research Group. It is not an
IETF product and is not a standard.


# Terminology {#terminology}

This document uses the following terms.

Network Management AI Agent (NMAA) system:
: A software system that receives a configuration or operational
  requirement from a human operator, or from another system acting on
  the operator's behalf, and acts on a network to satisfy it, using one
  or more AI models (typically LLMs) to decide its actions. The term
  covers single agents and multi-agent systems.

Human-Agent Interface (HAI):
: The interface through which a requirement is given to the NMAA
  system: a natural-language request with zero or more attached files
  (for example, a configuration file, a topology description, a
  spreadsheet of sites).

Agent-Network Interface (ANI):
: The interface through which the NMAA system observes and changes the
  network. The term was introduced in {{I-D.cui-nmrg-llm-benchmark}},
  where a single ANI is defined; this document treats the ANI as a
  factor that can take different abstraction levels ({{ani-levels}}).

System Under Test (SUT) boundary:
: The boundary that encloses the NMAA system between the HAI and the
  ANI. The model, the orchestration logic and the ANI adapter are
  inside; the network substrate, the generator, the fault injector and
  the observer are outside. The SUT boundary corresponds to the
  "Compound System" boundary of {{I-D.gaikwad-llm-benchmarking-terminology}}.

Brownfield:
: A network substrate that, before the agent acts, already carries a
  dense set of heterogeneous services sharing devices and resources.

Service instance:
: One deployed service (for example, one tunnel, one firewall policy
  set for a tenant), the unit of collateral damage.

Service class:
: A type of service (for example, SRv6 east-west tunnels, firewall
  policy, underlay routing, floating IP); a brownfield carries instances
  of several service classes.

Baseline invariant:
: A machine-checkable condition that a pre-existing service instance,
  or the network as a whole, satisfies before the agent acts and must
  keep satisfying afterwards.

Collateral damage:
: The violation of at least one baseline invariant of a pre-existing
  service instance, or of a global invariant, after the agent has acted.

Damage-free run:
: A run after which no baseline invariant is violated.

Task class:
: A family of requirements stated on the HAI by a fixed prompt
  template and file structure (for example, "onboard a new tenant
  site").

Task variant:
: A concrete instance of a task class (tenant, prefixes, target nodes),
  placed in a freshly generated brownfield.

Controlled factor:
: A property of the SUT whose settings are compared in a campaign
  (model, ANI level, agent mode, task class).

Stress parameter:
: A property of the environment that is swept to test whether a
  difference between factor settings persists, grows or disappears
  (state density, fault rate).

Run:
: One execution of the SUT on one task variant, with one setting of
  every factor and stress parameter, under a fixed budget.

Campaign:
: A set of runs designed to compare settings of one controlled factor.


# Relationship to Other Work {#related}

NetConfBench {{I-D.cui-nmrg-llm-benchmark}} defines an evaluation
framework for LLM agents performing intent-driven network configuration
in an emulated environment, with a single Agent-Network Interface
(get-topology, get-running-cfg, update-cfg, execute-validation) and
process metrics. The present framework adopts its notion of ANI and
generalizes the setting in three directions: the ANI becomes one of
several controlled factors; the environment is a brownfield in which
collateral damage can be observed; and the safety outcome is binary and
zero-tolerance, with statistics sized for near-zero probabilities.
Task sets developed for NetConfBench could be ported onto a brownfield
substrate and run within this framework.

{{I-D.contreras-bmwg-ai-agent-benchmarking}} proposes a benchmarking
methodology for AI agents in network operations, focused on
task-oriented performance, effectiveness, efficiency, robustness and
coverage metrics, and requires the tool interfaces and guardrails of
the SUT to be documented. The present framework is complementary: it
makes those interfaces and guardrails a factor to be varied, and adds
the brownfield substrate and the collateral-damage outcome. The
companion methodology document
{{METHODOLOGY}} is intended to be
aligned with that work.

{{I-D.gaikwad-llm-benchmarking-terminology}} and
{{I-D.gaikwad-llm-benchmarking-profiles}} define terminology and
profiles for benchmarking LLM serving systems, including a Compound
System profile with task success rate. The SUT boundary used here is a
Compound System boundary; the damage-free run rate and the ANI level
can be seen as extensions of that profile for network management.

{{I-D.wmz-nmrg-agent-ndt-arch}} describes an architecture for
AI-driven network operations and identifies agent benchmarking, as
opposed to model benchmarking, as an open challenge, calling for
process-oriented evaluation. ICON {{I-D.wnd-opsawg-icon-ps}}
{{I-D.mcw-opsawg-icon-requirements}} specifies requirements for
observing, controlling and intervening on network management agents
(scope restriction, approval, interruption, rollback). Those mechanisms
surround the agent; this framework measures what they, and the
abstraction of the interface itself, buy in terms of success and
safety. Operation-level intermediate representations such as
{{I-D.feng-netconf-naim-op}} and MCP-based exposures of network
operations {{I-D.yang-nmrg-mcp-nm}} are concrete interfaces that can be
placed on the abstraction spectrum of {{ani-levels}} and compared.

NetConfEval {{NETCONFEVAL}} compares translation targets of different
abstraction as independent static tasks; here the abstraction level is
varied for the same task, carried out by an agent acting in a closed
loop on a live substrate.

Autonomy levels for networks are defined by TM Forum {{TMF-IG1252}},
3GPP {{TS28100}} and assumed by ETSI ZSM {{ETSI-ZSM}}. They classify
how much of the decision loop is owned by the system rather than by a
human. The abstraction level of the ANI is a different and orthogonal
quantity: a system at any autonomy level still acts on the network
through some interface. To avoid confusion with those levels, and with
protocol layer numbering, the ANI levels in this document are named
A0 to A4. Intent-based networking {{RFC9315}} and autonomic networking
{{RFC8993}} provide the conceptual background for the higher levels.


# Reference Model and Experimental Design {#model}

## System Under Test {#sut}

~~~ ascii-art
              +--------------------+
              |  Human operator    |
              +---------+----------+
                        | requirement (prompt + files)
   =====================|============================ HAI
     SUT boundary       v
   +------------------------------------------------+
   |  NMAA system                                   |
   |   model(s)  +  agent mode  +  ANI adapter      |
   +----------------------+-------------------------+
   =======================|========================== ANI (level A0..A4)
                          v
   +------------------------------------------------+
   |  Brownfield substrate                          |
   |  (pre-existing services, shared resources)     |
   +------------------------------------------------+
        ^                         ^
        | fault injection         | out-of-band observation
   +----+-----+             +-----+------------------+
   | injector |             | observer (invariants,  |
   +----------+             | task test cases)       |
                            +------------------------+
~~~
{: #fig-sut title="Reference model"}

The SUT is the NMAA system between the HAI and the ANI
({{fig-sut}}). The level of the ANI is an internal choice of the NMAA
system, invisible to the operator: the same HAI request can be met by
an agent acting through a raw shell or through a typed service
interface. The substrate, the scenario generator, the fault injector
and the observer are part of the test environment, outside the SUT.

## Controlled Factors and Stress Parameters

A campaign compares settings of one controlled factor; the others are
held fixed. The controlled factors are:

* the model, or the set of models, used by the NMAA system;

* the abstraction level of the ANI ({{ani-levels}});

* the agent mode: single-turn generation, ReAct-style loop,
  planner-executor, with or without a recovery step;

* the task class, including the form of the HAI request.

Two stress parameters are swept across campaigns, to test whether a
difference between factor settings is robust:

* the state density of the brownfield: the number of service classes
  and of instances per class populated before the agent acts;

* the fault rate lambda: the rate of faults injected in the substrate
  while the agent acts.

## Task Classes and Variants

A task class is defined on the HAI by a prompt template and a file
structure. For each cell of the design, the scenario generator draws S
variants of the task class; each variant fixes the concrete request and
is placed in a freshly generated brownfield. The prompt text and file
structure are identical across the variants of a class, so that what
varies between runs is the situation and not the wording. Two task
classes that request the same reconfiguration in different words can be
used to separate the effect of expression from the effect of the
situation.

## Matched Budget

Settings of a factor are compared under a matched budget. The execution
budget (model, system prompt, agent mode, step and token caps, retry
policy, observation interface) is held constant except for the factor
under study. The development budget is matched as well: when the ANI
level is varied, a single shared agent is used, with no level-specific
tuning beyond exposing each interface. Cost (tokens, messages, steps,
wall-clock time) is recorded as a dependent variable, so that a factor
setting can be found to reach a safe state at lower cost, not only to be
safer at equal cost.

## Paired Runs and Common Random Numbers

Each task variant bundles the request, the generated brownfield and the
fault seed, and is replayed across the settings of the factor under
study. This makes the comparison paired: the same physical event (the
same fault, at the same point of the same scenario) is presented to
every setting. Faults are injected in the substrate, below the ANI, so
that the stimulus is identical for every setting and the comparison
measures how the effect propagates through each SUT.

## Out-of-Band Observation

Success and safety are measured by the test environment, not by the
SUT. The observer checks the test cases of the requested service and
the baseline invariants of every pre-existing service instance after the
run, through a channel that does not depend on the ANI level, so that
every setting is scored identically.

## Outcomes and Statistics {#outcomes}

The primary safety outcome of a run is binary: the run is damage-free
or it is not. The primary safety statistic of a cell is the
damage-free run rate. The completion outcome is the functional success
of the requested service. Each run also receives an outcome label:
clean success, dangerous success (the service works but damage
occurred), clean failure, destructive failure. Cost, recovery after a
fault, and the magnitude of damage when it occurs are secondary
outcomes.

Because the target regime is a damage probability near zero, the
statistics are part of the design and not an afterthought: proportions
are estimated with interval methods valid near zero (Wilson,
Clopper-Pearson), the number of runs per cell is sized from the bound
to be asserted (by the rule of three, zero damaged runs out of S give an
approximate 95% upper bound of 3/S), and paired comparisons use tests
on discordant pairs (McNemar). Definitions and procedures are given in
{{METHODOLOGY}}.


# Factor Spaces {#factors}

## Abstraction Levels of the Agent-Network Interface {#ani-levels}

The ANI levels form a spectrum from raw device access to reconciled
service automation ({{tab-levels}}).

| Level | Interface | Key safety or transactional property |
|-------|-----------|--------------------------------------|
| A0 | raw device shell | none; unbounded action space |
| A1 | bounded per-device command wrapper | generic guardrail (allowlist), snapshot and manual rollback |
| A2 | model-driven configuration (NETCONF/YANG) | device-level candidate datastore, validation, confirmed commit |
| A3-T | typed transactional service intent | cross-device workflow transaction, active validation, rollback |
| A3-R | reconciled source of truth | versioned desired state, drift detection and reconciliation |
| A4 | service intent and source of truth combined | both properties of A3-T and A3-R |
{: #tab-levels title="ANI abstraction levels"}

A0 to A2 form an ordered base, ordered by how far the interface moves
the agent from raw mutation and how much safety it interposes. A2
corresponds to standardized model-driven configuration with candidate
datastores and confirmed commit {{RFC6241}} {{RFC8342}} {{RFC8040}}.

Above A2 the levels form a plane spanned by two axes: scope, from
per-device configuration to cross-device service intent, and state and
control model, from one-shot actions to a persistent, versioned desired
state continuously reconciled against the network. A3-T advances the
first axis, A3-R the second; they are not ordered with respect to each
other. A4 combines both, and is where several commercial service
orchestrators operate.

A level denotes the abstraction exposed to the agent, not the stack used
to enforce it, and a level need not be built on the one below. A typed
service intent (A3-T) can be realized by serializing the intent into
device commands, without A2. This decoupling is what allows the same
task to be exposed at several levels on one substrate.

A level can be complemented by ad-hoc measures that do not change its
abstraction, such as a blocklist of known destructive commands added to
A1. Such measures are treated as ablations within a level, so that the
contribution of a structural interface and of hand-tuned prohibitions
can be separated.

## Models

The model factor covers the choice of model, model size and version,
and inference settings. A campaign that fixes the ANI level and varies
the model answers the question of which model is adequate for a task
class at a given level; one that varies both can test whether a higher
level compensates for a weaker model.

TODO: reporting of model identity and version for closed models;
handling of model updates during a campaign.

## Agent Modes

TODO: single-turn, ReAct, planner-executor, multi-agent; with and
without recovery; what is held fixed when the agent mode is the factor.

## Task Classes and the Form of the HAI

TODO: task taxonomy (onboarding, change-set on a live tenant,
decommissioning, troubleshooting); HAI forms from terse ticket to
structured intent; held-out targets placed to interact with existing
state.


# Brownfield Substrate and Scenario Generator {#substrate}

This section outlines the requirements on the test environment;
details are left to future revisions.

* Multi-service substrate. Instances of several service classes coexist
  on the same devices and share namespaces, address space, identifier
  pools and global settings, so that resource contention, and therefore
  collateral damage, is possible. Topology size and state density are
  independent scales: density is configuration state, not additional
  nodes, and can be swept on a modest topology.

* Layered, seeded scenario generator. The initial state is described by
  a manifest that stacks service-class layers on a topology base. Each
  layer establishes its prerequisites, generates its instances as real
  device state together with their baseline invariants (so that the
  oracle is emitted with the state), and verifies them. The manifest
  and a seed determine the scenario. Every pre-existing service passes
  its invariants before the run.

* Realism of the initial state. Service sizes and node load follow
  skewed distributions; shared-resource dependencies and prefix
  adjacency or overlap occur at a tunable rate, so that shadowing and
  collisions are genuine traps; the target of the task is placed to
  interact with existing state.

* Substrate-level fault injection. Faults are injected at node and
  link level (loss, latency, link down, frozen node, daemon restart,
  held lock, concurrent writer, transient pool exhaustion), identically
  for every setting, never inside an ANI adapter. Partial application,
  double application on retry and lockout are measured outcomes, not
  injected stimuli.

TODO: minimum requirements for a conforming substrate; how the
generator's realism can be assessed.


# Example Campaigns {#campaigns}

This section illustrates how operational questions map onto campaigns
of the framework.

## Fixed Model, Varying ANI Level

Question: through which interface should an agent be allowed to act?
The model, agent mode, task class and budget are fixed; the ANI level is
the treatment. A pilot of this campaign is reported in
{{ANIGAMUT}}, on a multi-tenant SRv6 brownfield of about twenty-five
nodes with on the order of ten service classes. On a change-set task on
a live 25-site tenant (two additions, two moves, one removal), an agent
acting at A0 completed 0 of 6 paired variants and the same agent at
A3-T completed 6 of 6 (exact McNemar test on the six discordant pairs,
p = 0.031), using about 3 steps and 42k tokens per run against 40
steps (the cap) and about 375k tokens. Under injected faults
(lambda = 0.3) on an onboarding task, A3-T completed 4 of 4 runs and A0
none. The damage-free run rate was 1.00 at every level: the agents
failed safe and the substrate kept tenants in separate VRFs, so
collateral safety was determined by the isolation of shared state
rather than by the ANI level. This illustrates why the substrate must
contain services whose shared state isolation does not protect, if
collateral damage is to be measured at all.

## Fixed ANI Level, Varying Model

Question: which model is adequate for a task class at a given level?
The ANI level, agent mode, task class and budget are fixed; the model
is the treatment. The paired design is unchanged: each task variant is
replayed across models.

## Varying Agent Mode

Question: does the orchestration of the agent matter more than the model
or the interface? The agent mode is the treatment, with model and ANI
level fixed.

## Stress Sweeps

Any of the campaigns above can be repeated across values of the state
density and of the fault rate. A difference between settings that grows
with density or with the fault rate indicates a property that small,
sparse testbeds would hide.


# Operational Considerations {#operational}

TODO: how an operator can use results to place an agent at a level for
a given service type; limits of emulated substrates; transfer of
results to production; cost of campaigns.


# Security Considerations {#security}

Benchmarking NMAA systems requires granting agents the ability to change
network configuration. Campaigns need to be run on isolated substrates
that cannot reach production networks or management systems; this
applies in particular to the lowest ANI levels, which give the agent an
unbounded action space.

Results of a campaign characterize a SUT on a given substrate and
budget. They do not establish that a system is safe to deploy; a
damage-free run rate of 1.00 over S runs only bounds the damage
probability, approximately, below 3/S at 95% confidence.

The framework measures accidental collateral damage by agents acting in
good faith. Adversarial inputs (prompt injection through the HAI or
through data read from the network, compromised tools) are out of scope
of this version and are addressed by other work. Generic guardrails and
blocklists can make a level appear safe by tuning to the benchmark; the
framework treats them as explicit ablations so that this effect can be
measured.


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# ANI-Gamut: A Reference Implementation {#implementation}

ANI-Gamut {{ANIGAMUT}} {{ANIGAMUT-EXT}} is an open-source implementation
of the framework {{ANIGAMUT-CODE}} (release tag cnsm2026). It provides:

* a harness with per-level ANI adapters for A0, A1 and A3-T, A1 with an
  optional blocklist as an ablation, A2 and A3-R being defined but not
  yet implemented;

* a multi-service SRv6 substrate on Containerlab with Linux and FRR
  nodes, using a NEXT-CSID data plane, with firewall, underlay routing,
  floating-IP and addressing services coexisting on the same nodes;

* a layered, seeded scenario generator with co-emitted invariants;

* substrate-level fault injection and out-of-band observation.

It builds on the benchmarking playground described in {{PLAYGROUND}}.
A project page is available at <https://netgroup.github.io/ani-gamut/>.


# Acknowledgments
{:numbered="false"}

The framework originates from work carried out with Lorenzo Bracciale,
Pierpaolo Loreti and Andrea Mayer. The authors thank Frank Loh and
Papasega Wade for discussions.
