# AI-Assisted Metadata Intelligence and Data Resolution

This is a strong use case for extending the metadata discovery approach beyond simple extraction. LangExtract would not be the entity-resolution engine itself. Instead, it would be one way to capture institutional knowledge—rules, mappings, naming conventions, identifier semantics, exceptions, and relationships—from documents and examples. That knowledge could then be combined with existing structured metadata to infer missing fields, resolve ambiguous records, and explain why a particular resolution was made.

## High-Level Architecture

```text
             Institutional Knowledge
        ┌──────────────┬───────────────┐
        │              │               │
     Manuals       Taxonomies      Team Rules
     Wikis          Mappings       ID Patterns
        │              │               │
        └──────────────┴───────────────┘
                       │
                LangExtract / ETL
                       │
                       ▼
                Knowledge Model
                       │
              ┌────────┴────────┐
              │                 │
         Knowledge Graph     Rule Store
              │                 │
              └────────┬────────┘
                       │
                 Resolution Engine
                       │
          ┌────────────┼─────────────┐
          │            │             │
      Exact rules   Similarity    LLM/ML inference
          │            │             │
          └────────────┴─────────────┘
                       │
                       ▼
                Resolved Metadata
              + confidence
              + provenance
              + explanation
```

## Example: Resolving a Missing `sensor_type`

Suppose an imagery record contains:

```text
image_id:     ABC_20260809_X12_0045
platform:     PLATFORM_7
sensor_type:  NULL
collection:   NIGHT
resolution:   0.30
```

The organization may already know things such as:

```text
Image IDs containing X12 usually come from Sensor A.

Platform 7 can carry Sensor A or Sensor B.

Night collections from Platform 7 at 0.30 resolution
are almost always Sensor A.

Older records sometimes omit sensor_type.

The X12 identifier was replaced by X14 after 2024.
```

Today, that knowledge may exist in people's heads, spreadsheets, mapping tables, documentation, code, wikis, or old email threads.

The relationships could instead be represented explicitly:

```text
(ImageIDPattern:X12)
    ──INDICATES_SENSOR──> (Sensor:A)

(Platform:7)
    ──CARRIES_SENSOR──> (Sensor:A)

(Platform:7)
    ──CARRIES_SENSOR──> (Sensor:B)

(Sensor:A)
    ──SUPPORTS_RESOLUTION──> (Resolution:0.30)

(ImageIDPattern:X12)
    ──REPLACED_BY──> (ImageIDPattern:X14)
```

When a record has:

```text
sensor_type = NULL
```

the resolution system could reason over the available evidence:

```text
Record
  │
  ├─ image_id matches X12
  ├─ platform = 7
  ├─ resolution = 0.30
  └─ collection = NIGHT

Evidence:
X12 → Sensor A
Platform 7 → Sensor A or B
0.30 resolution → Sensor A

Result:
sensor_type = Sensor A
confidence = 0.94
```

Rather than simply writing the inferred value back to the record, the system should retain metadata about the inference:

```json
{
  "sensor_type": "A",
  "resolution_method": "inferred",
  "confidence": 0.94,
  "evidence": [
    "image_id pattern X12",
    "platform 7",
    "resolution 0.30"
  ],
  "rule_version": "sensor_rules_v4"
}
```

This creates **metadata about the metadata**, including:

```text
value
source
confidence
inference_method
rule used
timestamp
version
supporting evidence
```

## Where LangExtract Fits

For imagery records that are already structured database rows, LangExtract would probably not be run against every record. Instead, it could be used to extract institutional knowledge from sources such as:

- technical manuals
- sensor documentation
- naming-convention documents
- team SOPs
- data dictionaries
- mapping documents
- wiki pages
- collection documentation

For example, a document might state:

> Prior to 2024, imagery collected from Platform 7 using payload configuration X12 corresponds to Sensor A.

LangExtract could convert that statement into structured knowledge such as:

```text
Platform 7
    HAS_CONFIGURATION → X12

X12
    MAPS_TO_SENSOR → Sensor A

X12
    VALID_BEFORE → 2024
```

Pydantic could then validate the extracted information before it becomes part of the institutional knowledge base.

The pipeline from the notebook would evolve from:

```text
Document
  ↓
LangExtract
  ↓
Pydantic
  ↓
Knowledge Graph
```

to:

```text
Institutional documents
          ↓
      LangExtract
          ↓
       Pydantic
          ↓
 Institutional Knowledge Graph
          │
          │
Structured imagery metadata ─────┐
                                 │
                                 ▼
                        Resolution Engine
                                 │
                                 ▼
                        Enriched Metadata
```

## Entity Resolution Is Only One Application

The knowledge graph should not be designed only around entity resolution. A broader goal would be to represent:

> **What does our organization know about its data?**

Entity resolution then becomes one application of that shared knowledge.

The same system could eventually answer questions such as:

```text
What sensor produced this image?

Why do we believe it was Sensor A?

What does X12 mean in an image identifier?

Which datasets use the old identifier convention?

Which fields can be inferred from image IDs?

What changed about this metadata after 2024?

Which platforms can carry Sensor B?

What records may be mislabeled?

What metadata fields are dependent on platform configuration?

Which mappings are authoritative versus inferred?

What documentation supports this mapping?

If I change this taxonomy, which datasets are affected?
```

This begins to look less like a simple entity-resolution tool and more like an **institutional data intelligence layer**.

## Why a Knowledge Graph Is Useful

A relational table is excellent for storing records such as:

```text
record_id | sensor | platform | date
```

However, this problem contains many relationships:

```text
Sensor
  IS_INSTALLED_ON → Platform

Platform
  USES_CONFIGURATION → Configuration

Configuration
  IMPLIES → Sensor

ImageIDPattern
  ENCODES → CollectionType

Field
  DERIVED_FROM → OtherField

TaxonomyTerm
  SAME_AS → LegacyTerm

TaxonomyTerm
  REPLACED_BY → NewTerm

Rule
  APPLIES_TO → Dataset

Rule
  SUPPORTED_BY → Document

Record
  RESOLVED_USING → Rule
```

Those relationships are well suited to graph representation.

The graph can also capture how knowledge changes over time. For example:

```text
X12 → Sensor A
```

may only have been valid during a certain period.

A graph database such as Neo4j could represent that relationship with attributes:

```text
(X12)-[:MAPS_TO {
    valid_from: 2018,
    valid_to: 2023,
    confidence: 1.0,
    source: "Payload Configuration Manual v3"
}]->(SensorA)
```

This type of temporal, contextual, and provenance-rich relationship becomes increasingly difficult to manage using simple lookup tables alone.

## Use Multiple Resolution Strategies

An LLM should not be responsible for deciding every ambiguous value. A safer resolution hierarchy would be:

1. **Authoritative mappings**

   ```text
   exact mapping table says X12 = A
   ```

2. **Deterministic rules**

   ```text
   if platform=7 and configuration=X12 → A
   ```

3. **Graph reasoning**

   ```text
   record → platform → configuration → sensor
   ```

4. **Statistical or similarity-based inference**

   ```text
   98% of otherwise similar records are Sensor A
   ```

5. **LLM inference**

   ```text
   interpret ambiguous documentation or naming conventions
   ```

6. **Human review**

   ```text
   confidence is too low or evidence conflicts
   ```

This is much safer and more explainable than simply asking an LLM to fill in missing fields.

## Capturing Tacit Institutional Knowledge

One of the most valuable aspects of this approach is capturing knowledge that currently exists only through years of experience.

For example, a senior analyst might explain:

> If the fourth section of the ID starts with Q, it is usually Sensor B—but only for Platform 3. Before 2019 it meant something different. If the collection type is Z, you also need to check the resolution because some historical records were migrated incorrectly.

That information could be modeled explicitly:

```text
Rule
 ├── condition → ImageID section 4 starts Q
 ├── condition → Platform = 3
 ├── exception → date < 2019
 ├── exception → CollectionType = Z
 ├── evidence → Analyst SOP
 ├── source → Senior Analyst
 ├── confidence → 0.88
 └── predicts → Sensor B
```

An AI system could then explain a resolution rather than simply returning a value:

```text
Sensor B is inferred with 88% confidence.

The image identifier matches the Q-pattern used for Platform 3 after 2019.
The record also matches the expected resolution.

This inference is based on rule IMG-SENSOR-017 and supporting institutional
documentation.
```

This turns tacit organizational knowledge into something that can be searched, validated, reasoned over, reused, and transferred to future analysts and AI systems.

## How This Connects to the Notebook

The aviation notebook already demonstrates most of the architectural foundation:

```text
Docling
    document understanding

LangExtract
    institutional knowledge extraction

Pydantic
    knowledge validation

LangGraph
    controlled reasoning workflow

Knowledge Graph
    institutional memory

LlamaIndex
    retrieval

LLM
    natural-language interface
```

A good next proof of concept would use:

```text
20–50 imagery metadata records
+
1 page describing identifier conventions
+
1 CSV of existing mappings
+
3–5 deliberately NULL sensor_type fields
```

The system could then produce something like:

```text
Original:
sensor_type = NULL

Resolved:
sensor_type = Sensor A

Confidence:
0.93

Reason:
Image ID X12 pattern + platform mapping + resolution

Supporting knowledge:
Mapping table row 17
Identifier guide §3.2
```

That would move the project from **AI metadata extraction** toward **AI-assisted metadata intelligence and entity resolution**.

The larger opportunity is not simply an agent that reads metadata, but a system that captures and operationalizes the **institutional semantics, mappings, relationships, exceptions, provenance, and historical knowledge surrounding the data**. 


## See also
https://github.com/google/langextract/blob/main/docs/examples/medication_examples.md 