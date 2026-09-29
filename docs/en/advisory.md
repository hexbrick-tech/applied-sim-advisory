# Applied SIM for Advisory (Draft 0.5)

**ASG Conformance Target:** Applied SIM Standard Guidelines v0.1

## 1. Purpose

Provide third-party AI evaluation of technical investigation results produced by an enterprise (end user), or outputs from AI-based technical investigations, identify missing perspectives, and present them as advisory opinions. The user decides whether to adopt the presented content in light of the stated purpose.

## 2. Scope

- Target: investigation results and investigation outputs concerning technical information
- Out of scope: conducting the investigation itself (this practice is limited to evaluation and advisory)
- References: hexbrick-tech/sim (Foundation), hexbrick-tech/applied-sim-standard-guidelines (ASG v0.1)

## 3. Vocabulary Mapping

| SIM Concept | Definition in Advisory |
| --- | --- |
| Observation | The original text of enterprise investigation results, AI outputs, or RAG returns, before summarization or evaluation |
| Interpretation | Reading of what the original text claims or implies, including relationships and comparisons among Observations |
| Evaluation | Judgment against the purpose or criteria (Gap determination, Opinion, adoption decision) |
| Undefined | A state in which something has been observed but has not yet undergone Evaluation (Gap determination). It is a transient condition and is not persisted as an Artifact. Evaluation either records it as a Gap or completes processing with no gap identified |
| Unknown | A state in which information required for judgment is not included in the inputs |
| Semantic Boundary | A state in which a topic or position can be handled stably (Established) |
| Semantic Probe | Additional confirmation requested from the user in a form that elicits observed facts |
| Review Cycle | The evaluation execution unit under an Inquiry Frame. A Snapshot is a post-hoc Projection of this execution unit; the two have independent responsibilities. A Review Cycle exists even when no Snapshot exists |

## 4. Process

```
Step 1: Provide purpose + investigation results / AI output (establish Inquiry Frame)
        ↓
   ┌──▶ Step 2: Evaluate missing perspectives
   │        ↓
   │    Step 3: Present Opinion (supplement with RAG when needed)
   │        ↓
   │    Step 4: User decides adoption/rejection and provides gap input
   │        ↓ (optional)
   │    [Generate SnapshotReport]
   │        ↓
   └────── Feed back into Step 2
```

When Purpose, Scope, or Criteria changes, create a new Inquiry Frame. Review Cycles begin anew under that Inquiry Frame, with cycle numbering restarting from 1. Steps 2–4 repeat as long as the Inquiry Frame remains unchanged.

```
IF-001
 ├ RC-001 (Cycle 1)
 ├ RC-002 (Cycle 2)
 └ RC-003 (Cycle 3)

IF-002 (newly created due to a change in Purpose, etc.)
 ├ RC-001 (Cycle 1; numbering restarts independently)
 └ RC-002
```

## 5. Artifacts

### 5.1 Inquiry Frame

| Field | Description |
| --- | --- |
| ID | IF-001 |
| Purpose | Investigation purpose |
| Scope | Scope of the investigation |
| Judgment Criteria | Criteria for determining “sufficient” (Unknown if absent) |
| Captured At | Timestamp |

### 5.2 Review Cycle

| Field | Description |
| --- | --- |
| ID | RC-001 |
| Ref Inquiry Frame | IF-ID |
| Started | Start timestamp |
| Finished | End timestamp (Unknown while in progress) |

### 5.3 Observation Ledger

| Field | Description |
| --- | --- |
| ID | O-001 |
| Source Class | Enterprise-Research / AI-Output / RAG-Return |
| Source | Specific source |
| Query | Actual query, only for RAG-Return |
| Raw Observation | Original text, including Markdown / JSON / code, etc. |
| Captured At | Timestamp |
| Context | Generation context, or associated Gap ID for RAG-Return |

### 5.4 Gap Record

| Field | Description |
| --- | --- |
| ID | G-001 |
| Ref Review Cycle | RC-ID |
| Gap Description | Description of what is missing |
| Basis | Which of Purpose / Scope / Criteria the determination is based on |
| Origin | Model-derived in principle; exceptionally, user-input-derived |
| Status | Open / Opinion Issued / Adopted / Rejected / Partially Adopted |
| Supersedes | Previous Gap ID inherited when revisiting the topic |

The single path to an Inquiry Frame is `Ref Review Cycle → Review Cycle.Ref Inquiry Frame`. A Gap Record does not hold a direct reference to an Inquiry Frame.

**Status rules:**

- Open does not mean unprocessed. It means that Evaluation has established the Gap and it remains unresolved or non-terminal. Open itself has reporting value; lingering is not treated as a problem.
- Revisions are allowed without limit while the status remains Opinion Issued. They are represented through Opinion.Supersedes.
- A terminal state (Adopted / Rejected / Partially Adopted) is never rolled back to Open. Revisiting the same topic is represented by a new Gap plus Supersedes.

```
Observation
  ↓
Undefined (transient condition before Evaluation)
  ↓
Evaluation
  ↓
Gap: Open (established, non-terminal)
```

### 5.5 Opinion

| Field | Description |
| --- | --- |
| ID | OP-001 |
| Ref Review Cycle | RC-ID |
| Refers To Gap | G-ID |
| Opinion Text | Specific opinion or proposal |
| Used Observations | Observation IDs used as the basis |
| Includes Model-Derived Content | Yes / No. Yes when the Opinion relies on general knowledge, industry practices, or external information outside the observed scope, beyond paraphrase or Interpretation (including relationships and comparisons) of Used Observations |
| Supersedes | Previous Opinion ID when revised within the same Gap |

**Supersedes scope:** Supersedes occurs only within the same Gap. Lineage across Gaps is followed through Gap.Supersedes; Opinions do not directly reference one another across Gaps.

If the distinction between Observation and RAG is needed, it can be derived by looking up the Source Class of each Used Observations ID, so the Opinion does not carry separate fields for it.

### 5.6 Reassessment Log

| Field | Description |
| --- | --- |
| ID | R-001 |
| New Gap | New Gap ID |
| Supersedes | Previous Gap ID being superseded |
| Trigger | New Observation ID that triggered the reassessment |

This log covers only Gap Supersedes within the same Inquiry Frame. Changes to the Inquiry Frame itself (changes in Purpose / Scope / Criteria) are represented through Inquiry Frame lineage, where multiple IFs may coexist, and are outside the scope of this log.

### 5.7 SnapshotReport (optional, generated after Step 4)

| Field | Description |
| --- | --- |
| ID | SR-001 |
| Ref Review Cycle | RC-ID |
| Generated At | Generation timestamp |
| Inquiry Frame Ref | IF-ID (reference only) |
| Gap Status Summary | Count by Status |
| Terminal Gaps (this cycle) | Gap IDs that reached a terminal state in this cycle |
| Open Gaps (cumulative) | All currently Open Gap IDs plus their age (facts only) |
| Supersedes Chains | Supersedes relationships created in this cycle |
| Basis Note | Explicitly states the items below |

**Basis Note:**

- This report is a Projection Artifact.
- It is used for Reporting purposes only.
- It is not intended for Restore.
- Observations must not be reconstructed backwards from the Snapshot.
- It may be referenced as input to a later Cycle, but it is not judgment evidence.

Past SnapshotReports are never overwritten or invalidated; they remain as historical records.

## 6. Domain-Specific Rules

- **Treatment of AI output (ASG-N3):** Fluency or plausibility of AI output is not evidence that its content is correct or an observed fact. As with other inputs, first freeze the original output as an Observation.
- **Treatment of RAG (ASG-B1, O3):** Introduce RAG only from Step 3 onward. Gap determination in Step 2 is based only on the Inquiry Frame and Step 1 inputs, without mixing in RAG. Record each RAG return individually in the Observation Ledger with its source and query; do not merge or summarize returns. Treat RAG as additional observation, not authority.
- **Derivation of missing perspectives:** In principle, AI derives them each time (Origin: Model-derived). If the user already has a prior understanding of what is missing, treat it as having already been complemented by another AI.
- **Cycle:** After Step 4 is complete, the next Step 2 begins based on new Observations and existing Gap states. Reevaluation is not forced.

## 7. ASG Requirement Mapping (Excerpt)

| ASG Requirement | Mapping |
| --- | --- |
| O1, O2 | Observation Ledger records original text without modification. Evidence is not managed redundantly in Opinion / Snapshot |
| O3, U2 | Undefined is defined as a transient condition; the existence of an Observation is not treated as establishment of a boundary |
| I1 | Interpretation (relationships and comparisons) is distinguished from Model-Derived Content |
| N2, N3 | Operational rules prevent AI output and RAG from being confused with “observed evidence.” Model-Derived Content is distinguished from Observation / Interpretation |
| E1, E2 | Basis is required for Gap Record / Opinion. SnapshotReport introduces no new Evaluation |
| U1, U3 | Gap = result of Evaluation; Unknown = absence of material required for judgment |
| B1 | An Open Gap is not treated as unestablished authority; RAG is not treated as authority |
| F1, F3 | Irreversible Gap Status plus new Gap + Supersedes represents reevaluation. The Step 4 → Step 2 cycle concretizes F1 |
