# SWARM Community Glossary Taxonomy

This project transforms selected terms from the **SWARM Community Glossary** Excel workbook into a lightweight SKOS taxonomy using [SPARQL Anything](https://github.com/SPARQL-Anything/sparql.anything).

## What it does

The mapping reads the **Vocabulary Terms** worksheet and processes only rows where:

```text
Category = Knowledge Graphs
```

For each matching row, it creates a `skos:Concept` with:

* `skos:prefLabel`
* `skos:altLabel`, when available

All other spreadsheet columns are ignored.

The generated concepts use the following base namespace:

```text
swarm : https://www.swarmcommunity.org/taxonomies/swarm-glossary/
```

Concept IRIs are generated from normalized versions of their preferred labels, for example `Knowledge Graph` becomes `swarm:knowledge-graph`.

## Running the mapping

From the project root, run:

```bash
java -jar sparql-anything-v1.2.0.jar \
  -q mapping/swarm-glossary.rq \
  -v "file=input/SWARM Community Glossary - Vocabulary Terms.xlsx" \
  -f TTL \
  -o output/swarm-glossary.ttl
```

> [!NOTE]
> 1. The `sparql-anything-v1.2.0.jar` file should be placed in the project root directory. It is intentionally excluded from version control (gitignored) and must be available locally to execute the mapping.
> 2. The Excel file path is passed to the SPARQL Anything query as a parameter, so the file location is not hard-coded inside the mapping.

## Output

The generated Turtle file is written to:

```text
output/swarm-glossary.ttl
```

Example output:

```turtle
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix swarm: <https://www.swarmcommunity.org/taxonomies/swarm-glossary/> .

swarm:knowledge-graph
    a skos:Concept ;
    skos:prefLabel "Knowledge Graph" ;
    skos:altLabel "KG" .
```

## Wikidata linking

A second phase enriches the generated SWARM glossary concepts with links to equivalent **Wikidata** entities.

For each glossary concept, the process:

1. searches Wikidata for the top three candidate entities;
2. provides those candidates, together with their labels and descriptions, to an LLM through GraphDB's GPT magic predicates;
3. asks the LLM to select an equivalent Wikidata entity, or no match when none of the candidates is appropriate;
4. validates that the selected entity was one of the candidates returned by Wikidata.

The resulting link can then be stored as a `skos:exactMatch`, for example:

```turtle
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix swarm: <https://www.swarmcommunity.org/taxonomies/swarm-glossary/> .
@prefix wd: <http://www.wikidata.org/entity/> .

swarm:semantic-web
    skos:exactMatch wd:Q54837 .
```

The queries and documentation for this phase are kept separately under [`wikidata-linking/`](wikidata-linking/README.md).

## Requirements

* Java
* [SPARQL Anything v1.2.0](https://github.com/SPARQL-Anything/sparql.anything/releases/tag/v1.2.0)
* The SWARM Community Glossary Excel workbook
* [Graphwise GraphDB](https://graphwise.ai/components/graphdb/) for the Wikidata-linking phase
* [GraphDB GPT integration](https://graphdb.ontotext.com/documentation/11.4/gpt-queries.html) when LLM-based candidate selection is used

## Notes

- The mapping uses the spreadsheet's actual column headers rather than positional column numbers. 
- The current taxonomy focuses specifically on the **Knowledge Graphs** category, but the same approach can later be extended to additional glossary categories. 
- The Wikidata-linking workflow is intentionally conservative: a concept may remain unmapped when no sufficiently equivalent Wikidata entity is found.

