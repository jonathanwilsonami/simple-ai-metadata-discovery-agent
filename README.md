# AI Metadata Discovery Tutorial — Turning Aviation Documents into Machine Readable Knowledge

## Introduction 
This notebook presents an end-to-end proof of concept for transforming an unstructured aviation PDF into validated, queryable metadata with traceable provenance. While the immediate goal is educational and exploratory, the work also serves as a precursor to a future AeroFlux integration component. This notebook uses NetworkX as a lightweight in-memory knowledge graph, but AeroFlux is expected to use Neo4j for persistent graph storage along with vector databases, a larger FAA document corpus, ontology management, autonomous planning, and integration with the AeroFlux user interface.

For now, the focus is intentionally narrow: demonstrate the core metadata discovery workflow clearly and reliably before expanding it into a production-scale architecture.

## What was Built

The notebook takes an FAA aviation PDF, extracts the text, identifies useful metadata, validates it against our defined schema, and organizes the results into a simple knowledge graph. It then uses that structured metadata and its source evidence to answer questions, while LangGraph coordinates the overall workflow.

## Lessons Learned 

I had been wanting to build something like this for both my personal projects and my professional work, so this prototype was a useful opportunity to explore the full workflow end to end. During the build, I learned how unstructured aviation documents can be transformed into structured, validated metadata that is easier for both humans and AI systems to use. LLMs on their own can struggle to produce semantically precise answers, especially in specialized domains with their own terminology, abbreviations, and conventions, so combining LangExtract with Pydantic validation and explicit provenance makes the results much more reliable and traceable. Once that structured foundation is established, knowledge graphs can represent relationships between entities in ways that are often more expressive than storing isolated fields in traditional tables; graph databases such as Neo4j and its Cypher query language can then be used to explore those relationships directly. Finally, LangGraph and LlamaIndex demonstrate how these components can be orchestrated into a controlled workflow and queried through a grounded AI interface, providing a strong foundation for a future metadata discovery agent.
