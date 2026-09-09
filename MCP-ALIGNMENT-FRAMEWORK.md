# Reactive MCP Alignment Framework

## Status

This document is a research and architecture baseline. It distinguishes established
protocol and provenance concepts from proposed metrics that require implementation and
empirical validation.

## Thesis

An MCP-enabled system is aligned when it can:

1. classify a request, capability, target, policy, and result;
2. establish that the requested operation is authorized and within scope;
3. execute through a compatible MCP capability;
4. validate the resulting state; and
5. preserve an auditable record of the transition.

The framework is **reactive**, not predictive. It responds to observed deviations from
a declared structural contract.

## 1. Common classification schema

Every relevant object or transition may be represented as:

```text
Phi(z) = <who, what, when, where, why, how>
```

| Field | Meaning |
| --- | --- |
| `who` | Actor, owner, authority, or responsible agent |
| `what` | Object, operation, resource, target, or result |
| `when` | Time, lifecycle phase, sequence, or state version |
| `where` | URI, directory, repository, environment, or scope boundary |
| `why` | Objective, policy reason, constraint, or acceptance condition |
| `how` | Tool, protocol, procedure, validator, or rollback mechanism |

The schema is a reusable classification interface, not a claim that six fields are a
complete ontology for every domain. Each layer defines its own semantics and validators.

## 2. Structural contract

A versioned application manifest, for example `mcp.structure.json`, can declare:

- allowed roots and paths;
- plugin identities and versions;
- six-field records;
- tool input and output schemas;
- permissions and authority boundaries;
- risk classes;
- required validators;
- provenance requirements;
- permitted reactions to observed deviations.

The manifest is an application-level contract. It is not part of the MCP protocol
itself.

## 3. Reactive control loop

Let:

```text
G_t = observed directory, repository, and dependency state
M_t = observed MCP capability and permission state
C_t = current task specification
Sigma = declared structural contract
```

The control loop is:

```text
x_t = observe(G_t, M_t, C_t)
e_t = compare(x_t, Sigma)
a_t = Policy(e_t)
```

`e_t` is an observed deviation, not a predicted future hazard. Possible reactions are
continue, inspect, repair, contain, clarify, roll back, escalate, or deny.

Recommended reaction precedence:

```text
deny/contain > escalate > clarify > repair > continue
```

This prevents an apparently repairable condition from overriding a safety or authority
failure.

## 4. Formula families

### 4.1 Capability compatibility

Represent task requirements and MCP capabilities in the same nonnegative feature space:

```text
r_t = required capability vector
c_t = available capability vector
```

Use:

```text
F(r_t, c_t) =
    (r_t dot c_t) / (||r_t||_2 ||c_t||_2 + delta)
```

`delta > 0` prevents division by zero. This is a compatibility score, not permission.
A tool can support an operation while the actor remains unauthorized to invoke it.

### 4.2 Classification completeness

For six dimensions:

```text
C(z) = sum_k omega_k * valid(W_k(z))
```

where `valid` is 0 or 1, `omega_k >= 0`, and the weights sum to 1.

For high-impact operations, required fields should be hard gates rather than
compensated by a weighted average. For example, writes may require valid `who`, `what`,
`where`, and `how`.

### 4.3 Directory quality

```text
Q(G_t, q_t) =
    w1*R_t + w2*M_t + w3*D_t + w4*P_t + w5*L_t
```

Where:

- `R_t`: retrieval quality;
- `M_t`: metadata completeness;
- `D_t`: dependency integrity;
- `P_t`: permission adequacy;
- `L_t`: layout consistency.

The weights are policy parameters and must sum to 1. Mandatory permission or dependency
failures should be applied as hard gates, not offset by good layout or metadata.

### 4.4 Retrieval entropy

If `p_j,t` is the calibrated probability that candidate `j` is correct:

```text
H_t = -sum_j p_j,t * log(p_j,t)
H*_t = H_t / log(n)
```

`H*_t` is normalized uncertainty over a candidate set of size `n`. It does not measure
all directory complexity. Depth, dependency fan-out, path length, and permission
boundaries require separate metrics.

### 4.5 Verified alignment indicator

Use the score for measurement and explanation:

```text
A_descriptive = C * F * Q * V
```

where `V` is validation readiness or verified execution quality. If all terms are in
`[0, 1]`, the result is in `[0, 1]`.

This score must not independently authorize an operation.

### 4.6 Operational distance

Model the environment as a directed graph:

```text
G = (N, E)
```

Nodes may represent files, directories, resources, tools, validators, policies, and
provenance records. Edges represent containment, dependency, invocation, permission
transitions, or validation transitions.

```text
d(u, v) = min over paths pi from u to v of sum_e_in_pi w(e)

w(e) = alpha*D_e + beta*P_e + gamma*U_e + delta*R_e + zeta*C_e
```

Where:

- `D_e`: traversal or structural depth;
- `P_e`: permission-boundary cost;
- `U_e`: uncertainty;
- `R_e`: operation risk;
- `C_e`: compute, latency, or tool-call cost.

This is a directed operational cost, not necessarily a mathematical metric: remediation
cost may not be symmetric.

### 4.7 Sentry authorization predicate

```text
Allow(a_t) =
    authorized(a_t)
    and within_allowed_root(a_t)
    and observed_trigger(a_t)
    and validation_plan_exists(a_t)
    and risk(a_t) <= policy_threshold
```

Optional scores such as `A_descriptive` and `d_t` may refine or prioritize permitted
actions, but cannot override this predicate.

### 4.8 Reaction reliability

```text
R_reaction =
    correct_verified_responses / eligible_detected_events
```

Track this by event class and stage:

- detection;
- classification;
- policy selection;
- execution;
- validation;
- rollback.

### 4.9 Empirical reliability

The proposed “internalization” term should be interpreted as observed reliability:

```text
I_t = average verified performance over a window W
```

This demonstrates performance under tested conditions; it does not prove literal policy
internalization.

## 5. Plugin contract

A plugin is a typed extension, not an unconstrained loadable object:

```text
P_i = <Phi(P_i), I_i, O_i, Gamma_i, Lambda_i>
```

Where:

- `Phi(P_i)`: six-field classification;
- `I_i`: accepted inputs;
- `O_i`: produced outputs;
- `Gamma_i`: authority and safety constraints;
- `Lambda_i`: lifecycle and validation requirements.

Registration is permitted only when:

```text
Register(P_i) =
    complete(Phi(P_i))
    and compatible(P_i, Sigma)
    and authorized(P_i)
```

Completeness means type-valid, policy-compatible, and operationally testable fields,
not merely the presence of six keys.

## 6. MCP mapping

| Framework concept | MCP primitive or extension |
| --- | --- |
| Structural contract | JSON resource plus application schema |
| Directory state | Resources, resource templates, or server-side index |
| Tool capability | MCP tool definition and input schema |
| Output validation | Optional MCP output schema plus application validators |
| Filesystem boundary | Client-provided MCP roots |
| Change observation | Resource subscriptions and list-change notifications |
| Sentry | Host/client/server wrapper and policy layer |
| Audit | Application resource, log, or provenance store |
| Six-field semantics | Application-level manifest and schemas |

MCP supplies interoperability primitives. It does not natively enforce the six-field
schema, risk scoring, provenance, rollback, or adaptive guardrails.

## 7. PROV-DM integration

Represent an MCP operation as an activity that uses and generates versioned entities and
is associated with an agent:

```text
Entity: request:v1
Entity: structure:v7
Activity: tool-call:structure-check:run-42
Entity: report:v1
Agent: approved-agent
```

The activity records the run and its time interval. Entities represent the data or
state involved. Material state changes should create new entity identifiers, linked by
derivation or specialization rather than mutating historical identity.

Strict-precedence cycles are invalid. Iterative refinement should therefore use
versioned entities and activities:

```text
config:v1
  -> refine activity
config:v2
  -> validate activity
config:v3
```

Influence or reference cycles are not automatically equivalent to impossible temporal
cycles; the relation type matters.

## 8. Divergence and convergence

Classify divergence rather than penalizing it uniformly:

| Type | Reaction |
| --- | --- |
| Contradiction | Block or reopen assessment |
| Scope escape | Deny or escalate |
| Ambiguity | Inspect or clarify |
| Drift | Repair or propose contract update |
| Exploration | Permit only within a safe scope |
| Innovation | Record, validate, and review |

Convergence measures stability, not truth. A system can repeatedly converge on the
wrong interpretation, so convergence requires external validation.

## 9. Validation and falsification

Benchmark conditions should include:

- clean and degraded directory structures;
- missing metadata;
- duplicate candidates;
- broken dependencies;
- changed roots;
- insufficient permissions;
- stale manifests;
- malformed tool output;
- contradictory policies.

Compare:

1. no directory index;
2. index without six-field records;
3. six-field records without provenance;
4. fixed guardrails;
5. adaptive guardrails;
6. reactive sentry with operational distance;
7. reactive sentry plus provenance validation.

Measure:

- event detection precision and recall;
- reaction reliability;
- unauthorized-operation rate;
- false repair rate;
- validation-pass rate;
- rollback success;
- latency and tool-call count;
- human-intervention rate;
- provenance completeness;
- contract-drift rate.

The hypothesis is weakened if the schema does not improve verified reaction reliability,
if the scores do not correlate with outcomes, or if adaptive controls reduce safety.

## 10. Boundaries of the claim

Established or directly implementable:

- MCP resources, tools, roots, schemas, and notifications;
- graph-based state representation;
- typed plugin contracts;
- bounded compatibility scores;
- hard authorization predicates;
- directed remediation costs;
- versioned provenance records;
- validation benchmarks.

Proposed and requiring evidence:

- six fields as a broadly reusable classification core;
- alignment score correlation with execution success;
- operational distance as a useful remediation predictor;
- productive divergence measurement;
- empirical reduction of external scaffolding without safety loss.

Not valid as originally written:

- `Geodesic = gradient of alignment`;
- reducing guardrails necessarily improves alignment;
- an unnormalized intent-capability dot product as a bounded alignment score;
- a single compensatory score as an authorization mechanism.

## 11. Research addendum: identity, typing, and provenance safeguards

### MCP primitives are not interchangeable

MCP distinguishes three control layers:

- **Prompts** are user-controlled;
- **Resources** are application-controlled;
- **Tools** are model-controlled.

A prompt is not a tool, a resource is not an activity, and a static tool declaration is
not the same object as a tool invocation. A tool invocation may be represented as a
PROV activity, while the tool definition is an interface or capability declaration.

### MCP resource identity is not snapshot identity

An MCP URI identifies a logical resource. Its contents may change while the URI remains
the same. For provenance, distinguish:

```text
logicalResourceId = MCP URI
snapshotId        = URI + content digest or explicit revision
```

Do not use a mutable URI alone as an immutable PROV entity identity.

### Tool annotations are hints, not guarantees

MCP tool annotations such as read-only or destructive behavior are untrusted hints
unless the server is trusted. A sentry must not infer safety solely from:

```text
readOnlyHint = true
```

Actual behavior requires independent authorization, scope checks, execution monitoring,
and post-action validation.

### Qualified identity

A durable identity should include enough context to avoid collisions between servers
and revisions:

```text
identity =
  (protocolVersion,
   serverAuthority,
   primitiveKind,
   localName,
   schemaDigest,
   snapshotOrRevision)
```

A local tool name is not a globally unique identity. A content digest or explicit
revision is also needed when a resource's contents are version-sensitive.

### PROV cycles and identity constraints

Not every cycle is invalid in PROV. In particular:

- `alternateOf` is an equivalence relation and may participate in cycles;
- `specializationOf` is a strict partial order and must not be reflexive;
- an ordering cycle is invalid when it contains a `strictly-precedes` edge;
- normalization termination is a separate concern from graph acyclicity.

Entity attributes alone do not necessarily establish identity. A stable identifier,
qualified name, URI, digest, or explicit key is required when identity must persist
across versions.

### Hard conformance gate

Use deterministic conformance gates before advisory scoring:

```text
G =
  typeOK
  and identityOK
  and schemaOK
  and authorizationOK
  and temporalOK
  and provenanceOK
```

If `G` is false, the operation is non-conformant regardless of its numerical alignment
score. Only after the gate passes should an advisory score summarize evidence quality.

Unknown values should remain explicitly unknown rather than silently becoming zero.
Contradiction should be represented separately from missing evidence.

## References

- Model Context Protocol specification: <https://modelcontextprotocol.io/specification/2025-06-18>
- MCP resources: <https://modelcontextprotocol.io/specification/2025-06-18/server/resources>
- MCP tools: <https://modelcontextprotocol.io/specification/2025-06-18/server/tools>
- MCP roots: <https://modelcontextprotocol.io/specification/2025-06-18/client/roots>
- W3C PROV-DM: <https://www.w3.org/TR/prov-dm/>
- W3C PROV constraints: <https://www.w3.org/TR/prov-constraints/>
- W3C PROV-O: <https://www.w3.org/TR/prov-o/>
