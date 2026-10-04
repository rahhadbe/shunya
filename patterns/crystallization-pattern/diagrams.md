# The Crystallization Pattern — Diagrams

Visual companion to [The Crystallization Pattern](crystallization-pattern.md).

| #   | Diagram                                                           | Answers                                                  |
| --- | ----------------------------------------------------------------- | -------------------------------------------------------- |
| 1   | [Architecture and control flow](#1-architecture-and-control-flow) | How does a request move through the system?              |
| 2   | [Request lifecycle over time](#2-request-lifecycle-over-time)     | What happens on a miss, a hit, and an "I don't know"?    |
| 3   | [Design decisions](#3-design-decisions)                           | What must be decided before building it?                 |
| 4   | [Position among similar ideas](#4-position-among-similar-ideas)   | How is it different from a cache, fine-tuning, or rules? |

---

## 1. Architecture and control flow

The verified map is the source of truth. The solver sits behind it and is reached only on a
miss. Nothing enters the map without passing verification.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "15px", "primaryTextColor": "#000000", "textColor": "#000000", "titleColor": "#000000", "lineColor": "#78909c", "edgeLabelBackground": "#ffffff"}}}%%
flowchart TB
    IN(["Raw input<br/><i>unbounded, free-form</i>"])
    NORM["<b>1 · Normalise</b><br/>trim · casing · punctuation<br/>canonical dates and numbers"]
    LOOK{"<b>2 · Lookup</b><br/>exact match<br/>on normalised key"}
    MAP[("<b>Verified map</b><br/>key → output<br/>verified_by · verified_at<br/>schema_version")]
    OUT(["Structured output<br/><i>finite, schema-constrained</i>"])

    subgraph SOLID["SOLID PATH · deterministic · fast · verified"]
        SERVE["Return frozen answer<br/>no model call<br/>same key ⇒ same answer"]
    end

    subgraph FLUID["FLUID PATH · probabilistic · slow · unverified"]
        SOLVE["<b>3 · Solve</b><br/>LLM with schema-constrained output<br/>may return UNKNOWN / NEED_INFO"]
        IDK{"UNKNOWN or<br/>NEED_INFO?"}
        VERIFY{"<b>4 · Verify</b><br/>human review · rule validator<br/>golden-set comparison"}
        POLICY["Miss response policy<br/>flagged answer · pending · safe default"]
    end

    subgraph WRITE["WRITE PATH · correctness asserted at write time"]
        REVIEW[["Human review queue"]]
        PROMOTE["<b>5 · Promote</b><br/>write entry + metadata"]
        DROP["Not promoted<br/>handled manually"]
    end

    GOV["<b>6 · Govern</b><br/>audit trail · review changes<br/>re-verify or invalidate on<br/>schema, vocabulary, or rule change"]

    IN --> NORM --> LOOK
    MAP -. read .-> LOOK
    LOOK == HIT ==> SERVE ==> OUT
    LOOK -- MISS --> SOLVE --> IDK
    SOLVE -.-> POLICY -.-> OUT
    IDK -- yes --> REVIEW
    IDK -- no --> VERIFY
    VERIFY -- fail --> REVIEW
    VERIFY -- pass --> PROMOTE
    REVIEW -- confirmed or corrected --> PROMOTE
    REVIEW -- not answerable --> DROP
    PROMOTE == write ==> MAP
    GOV -. governs .-> MAP

    classDef io fill:#cfd8dc,stroke:#37474f,stroke-width:2px,color:#000000
    classDef det fill:#90caf9,stroke:#0d47a1,stroke-width:2px,color:#000000
    classDef prob fill:#ffcc80,stroke:#e65100,stroke-width:2px,color:#000000
    classDef check fill:#ce93d8,stroke:#4a148c,stroke-width:2px,color:#000000
    classDef store fill:#a5d6a7,stroke:#1b5e20,stroke-width:3px,color:#000000
    classDef human fill:#f48fb1,stroke:#880e4f,stroke-width:2px,color:#000000
    classDef gov fill:#eeeeee,stroke:#424242,stroke-width:2px,stroke-dasharray:4 3,color:#000000

    class IN,OUT io
    class NORM,LOOK,SERVE det
    class SOLVE,POLICY prob
    class IDK,VERIFY check
    class MAP,PROMOTE store
    class REVIEW,DROP human
    class GOV gov

    style SOLID fill:#e3f2fd,stroke:#0d47a1,stroke-width:2px,color:#000000
    style FLUID fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#000000
    style WRITE fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px,color:#000000
```

**Reading it:** blue is deterministic, orange is probabilistic, purple is a check, green is
the authoritative store, pink involves a person. Thick arrows are the paths that matter most:
the hit path and the write-back.

---

## 2. Request lifecycle over time

The support-ticket example from the pattern, shown as three requests on different days.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "15px", "actorBkg": "#ffffff", "actorBorder": "#0d47a1", "actorTextColor": "#000000", "actorLineColor": "#90a4ae", "signalColor": "#000000", "signalTextColor": "#000000", "labelBoxBkgColor": "#ffffff", "labelBoxBorderColor": "#000000", "labelTextColor": "#000000", "noteBkgColor": "#fff59d", "noteBorderColor": "#f57f17", "noteTextColor": "#000000", "sequenceNumberColor": "#ffffff"}}}%%
sequenceDiagram
    autonumber
    actor C as Customer
    participant R as Router
    participant N as Normaliser
    participant M as Verified map
    participant L as LLM solver
    participant V as Verifier

    rect rgb(255, 204, 128)
    Note over C,V: Day 1 · first occurrence · MISS
    C->>R: "My invoice shows the wrong amount"
    R->>N: normalise(input)
    N-->>R: key = my invoice shows the wrong amount
    R->>M: get(key)
    M-->>R: not found
    R->>L: classify(input, schema)
    L-->>R: billing/disputes
    R->>V: verify proposal
    V-->>R: confirmed by team lead
    R->>M: put(key, billing/disputes, verified_by, schema_version)
    R-->>C: routed to billing/disputes
    end

    rect rgb(144, 202, 249)
    Note over C,V: Day 3 · same meaning, different surface form · HIT
    C->>R: "my invoice shows the WRONG amount!!"
    R->>N: normalise(input)
    N-->>R: key = my invoice shows the wrong amount
    R->>M: get(key)
    M-->>R: billing/disputes (verified)
    R-->>C: routed to billing/disputes
    Note over L,V: not called · deterministic · no model cost
    end

    rect rgb(244, 143, 177)
    Note over C,V: Day 4 · two intents · NEED_INFO
    C->>R: "Explain the refund policy ... and also my login fails"
    R->>N: normalise(input)
    N-->>R: key
    R->>M: get(key)
    M-->>R: not found
    R->>L: classify(input, schema)
    L-->>R: NEED_INFO (two intents)
    R-->>C: sent to manual triage
    Note over M: nothing written · "I don't know" is never promoted
    end
```

---

## 3. Design decisions

The five decisions from the pattern's design table. Each one must be answered before the
map can be trusted.

```mermaid
%%{init: {"theme": "base", "themeVariables": {"fontSize": "15px", "primaryTextColor": "#000000", "git0": "#fff176", "gitBranchLabel0": "#000000", "cScale0": "#90caf9", "cScaleLabel0": "#000000", "cScale1": "#ce93d8", "cScaleLabel1": "#000000", "cScale2": "#ffcc80", "cScaleLabel2": "#000000", "cScale3": "#a5d6a7", "cScaleLabel3": "#000000", "cScale4": "#f48fb1", "cScaleLabel4": "#000000"}}}%%
mindmap
  root((Crystallization design))
    [What is the key]
      (Exact text)
      (Normalised text)
      (Extracted fields)
      (Must stay deterministic)
    [How is an answer verified]
      (Human review)
      (Rule validator)
      (Golden set comparison)
      (The map is only as correct as this step)
    [What a caller gets on a miss]
      (Unverified answer flagged)
      (Pending or NEED_INFO)
      (Safe default)
      (Depends on harm of a wrong answer)
    [How the map is seeded]
      (Empty)
      (Golden set)
      (Historical decisions)
    [When entries are invalidated]
      (Schema change)
      (Rule change)
      (Expiry review)
```

---

## 4. Position among similar ideas

> **Qualitative positioning**, based on the "What it is not" table in the pattern.

```mermaid
quadrantChart
    title Who decides, and is the answer verified
    x-axis Unverified answer --> Verified answer
    y-axis Model decides each call --> Lookup decides
    quadrant-1 Verified and deterministic
    quadrant-2 Deterministic but unverified
    quadrant-3 Probabilistic and unverified
    quadrant-4 Verified but still probabilistic
    Crystallization: [0.88, 0.9]
    Rules engine: [0.72, 0.8]
    Cache: [0.18, 0.72]
    Semantic cache: [0.22, 0.42]
    Fine tuning: [0.35, 0.15]
    Few shot prompting: [0.2, 0.22]
```

Crystallization and a hand-written rules engine sit in the same quadrant. The difference is
where entries come from: rules are written up front by people, while crystallized entries are
discovered by the model from real traffic and confirmed by people.
