# AI Metadata Discovery Tutorial — Turning Aviation Documents into Machine Readable Knowledge

## Introduction 
This notebook presents an end-to-end proof of concept for transforming an unstructured aviation PDF into validated, queryable metadata with traceable provenance. While the immediate goal is educational and exploratory, the work also serves as a precursor to a future AeroFlux integration component. This notebook uses NetworkX as a lightweight in-memory knowledge graph, but AeroFlux is expected to use Neo4j for persistent graph storage along with vector databases, a larger FAA document corpus, ontology management, autonomous planning, and integration with the AeroFlux user interface.

For now, the focus is intentionally narrow: demonstrate the core metadata discovery workflow clearly and reliably before expanding it into a production-scale architecture.
