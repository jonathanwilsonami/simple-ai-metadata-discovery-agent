# AI Metadata Discovery Tutorial — Turning Aviation Documents into Machine Readable Knowledge

## Introduction 
This notebook presents an end-to-end proof of concept for transforming an unstructured aviation PDF into validated, queryable metadata with traceable provenance. While the immediate goal is educational and exploratory, the work also serves as a precursor to a future AeroFlux integration component. This notebook uses NetworkX as a lightweight in-memory knowledge graph, but AeroFlux is expected to use Neo4j for persistent graph storage along with vector databases, a larger FAA document corpus, ontology management, autonomous planning, and integration with the AeroFlux user interface.

For now, the focus is intentionally narrow: demonstrate the core metadata discovery workflow clearly and reliably before expanding it into a production-scale architecture.

## What was Built

The notebook takes an FAA aviation PDF, extracts the text, identifies useful metadata, validates it, and organizes the results into a simple knowledge graph. It then uses that structured metadata and its source evidence to answer questions, while LangGraph coordinates the overall workflow.

## Lessons Learned 

I had been wanting to build something like this for both my personal projects and my professional work, so this prototype was a useful opportunity to explore the full workflow end to end. During the build, I learned how unstructured aviation documents can be transformed into structured, validated metadata that is easier for both humans and AI systems to use. LLMs on their own can struggle to produce semantically precise answers, especially in specialized domains with their own terminology, abbreviations, and conventions, so combining LangExtract with Pydantic validation and explicit provenance makes the results much more reliable and traceable. Once that structured foundation is established, knowledge graphs can represent relationships between entities in ways that are often more expressive than storing isolated fields in traditional tables; graph databases such as Neo4j and its Cypher query language can then be used to explore those relationships directly. Finally, LangGraph and LlamaIndex demonstrate how these components can be orchestrated into a controlled workflow and queried through a grounded AI interface, providing a strong foundation for a future metadata discovery agent.

## Another Use Case 
See the markdown document titled `USECASE.md` and of course https://github.com/google/langextract/tree/main/docs/examples. 

## References

1. **Federal Aviation Administration (FAA).** *Digital Chart Supplement / Airport Facility Directory.*  
   https://www.faa.gov/air_traffic/flight_info/aeronav/digital_products/dafd/

2. **FAA.** *Northeast Chart Supplement — July 9, 2026 edition.*  
   https://aeronav.faa.gov/Upload_313-d/supplements/CS_NE_20260709.pdf

3. **Docling Project.** *Docling Quickstart and DocumentConverter API.*  
   https://docling-project.github.io/docling/getting_started/quickstart/  
   https://docling-project.github.io/docling/reference/document_converter/

4. **Google.** *LangExtract — structured information extraction with source grounding.*  
   https://github.com/google/langextract

5. **Google Developers Blog.** *Introducing LangExtract: a Gemini-powered information extraction library.*  
   https://developers.googleblog.com/introducing-langextract-a-gemini-powered-information-extraction-library/

6. **Pydantic.** *Pydantic documentation.*  
   https://docs.pydantic.dev/

7. **LangChain.** *LangGraph overview.*  
   https://docs.langchain.com/oss/python/langgraph/overview

8. **NetworkX Developers.** *NetworkX documentation.*  
   https://networkx.org/documentation/stable/

9. **LlamaIndex.** *LlamaIndex open-source framework.*  
   https://github.com/run-llama/llama_index

10. **Google AI for Developers.** *Gemini model documentation.*  
    https://ai.google.dev/gemini-api/docs/models

11. **Google AI for Developers.** *Gemini embeddings.*  
    https://ai.google.dev/gemini-api/docs/embeddings
