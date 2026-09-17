# Wikidata Linking for the SWARM Community Glossary

This folder contains the resources used to enrich the **SWARM Community Glossary taxonomy** with links to equivalent entities in **Wikidata**.

This phase assumes that the generated SWARM glossary Turtle data has already been loaded into a GraphDB repository, for example a repository named `swarm-glossary`.

## Goal

The objective is to identify an equivalent Wikidata entity for each SWARM glossary concept where such an entity exists.

Mappings are represented using `skos:exactMatch`; for example:

```turtle
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix swarm: <https://www.swarmcommunity.org/taxonomies/swarm-glossary/> .
@prefix wd: <http://www.wikidata.org/entity/> .

swarm:semantic-web
    skos:exactMatch wd:Q54837 .
```
> [!NOTE]
> A concept **does not** have to receive a Wikidata mapping. If no sufficiently equivalent entity is available, leaving the concept unmapped is considered a valid outcome.

## How the process works

The enrichment workflow has two main stages.

### 1. Wikidata candidate search

For each SWARM glossary concept, the preferred label is used to search Wikidata.

The Wikidata EntitySearch service returns the **top three candidate entities**, together with useful information such as:

* Wikidata Q-id
* label
* English description, when available
* search ranking

Restricting the search to three candidates gives the following LLM step a small and focused set of alternatives.

### 2. LLM-assisted candidate selection

[GraphDB's GPT magic predicates](https://graphdb.ontotext.com/documentation/11.4/gpt-queries.html) are used to evaluate the Wikidata candidates.

The LLM receives:

* the SWARM preferred label;
* the alternative label, when available;
* the three Wikidata candidates;
* their labels and descriptions.

It is instructed to select a candidate only when it represents the **same concept**, rather than something merely related, broader, or narrower.

The LLM must return either:

* one of the Wikidata Q-ids supplied in the candidate list; or
* `NONE` if none of the candidates is sufficiently equivalent.

The SPARQL query additionally validates the response so that the LLM cannot introduce a Wikidata entity that was not returned by the search.

### `queries/wikidata-candidate-search.rq`

A diagnostic query that performs only the Wikidata search and returns the top three candidates for each SWARM concept.

This is useful for checking the quality of the Wikidata search independently of the LLM.

### `queries/wikidata-llm-linking.rq`

This is the complete enrichment query which

1. retrieves the SWARM glossary concepts;
2. searches Wikidata for three top candidates;
3. prepares the candidate information for the LLM;
4. asks the LLM to select the equivalent entity or return `NONE`;
5. validates that the selected Q-id was one of the Wikidata candidates;
6. prepares the corresponding Wikidata entity IRI.

> [!TIP]
> During development, the query can be run as a `SELECT` so that the proposed mappings can be inspected before modifying the repository.

Once the results have been reviewed, the query can be adapted to insert accepted mappings as `skos:exactMatch` triples.

## Design principles

The workflow follows a few simple principles:

* **Wikidata performs the search.** The LLM does not invent or independently search for Wikidata identifiers.
* **The LLM works with a constrained candidate set.** It can select only one of the three entities returned by Wikidata.
* **No match is acceptable.** Returning `NONE` is preferable to creating an incorrect mapping.
* **Mappings can be reviewed before insertion.** The `SELECT` workflow makes proposed links visible before they are written back to the graph.
* **`skos:exactMatch` is used conservatively.** A Wikidata item should represent essentially the same concept as its SWARM glossary counterpart.
