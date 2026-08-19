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

Concept IRIs are generated from normalized versions of their preferred labels, for example:

```text
Knowledge Graph
```

becomes:

```turtle
swarm:knowledge-graph
```

## Running the mapping

From the project root, run:

```bash
java -jar sparql-anything-v1.2.0.jar \
  -q mapping/swarm-glossary.rq \
  -v "file=input/SWARM Community Glossary.xlsx" \
  -f TTL \
  -o output/swarm-glossary.ttl
```

> [!IMPORTANT]
> The `sparql-anything-v1.2.0.jar` file should be placed in the project root directory. It is intentionally excluded from version control (gitignored) and must be available locally to execute the mapping.

> [!NOTE]
> The Excel file path is passed to the SPARQL Anything query as a parameter, so the file location is not hard-coded inside the mapping.

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

## Requirements

* Java
* SPARQL Anything v1.2.0
* The SWARM Community Glossary Excel workbook

## Notes

The current taxonomy focuses specifically on the **Knowledge Graphs** category, but the same approach can later be extended to additional glossary categories.
