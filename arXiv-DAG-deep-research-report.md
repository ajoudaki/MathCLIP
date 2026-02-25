# Statement-Level Graphs for Mathematical Papers: Open Projects Organizing arXiv Content into Dependency DAGs

## Executive summary

A small number of public/open projects attempt what you described—turning mathematical papers into **graph-structured (often DAG-like) networks of statements** (theorems/lemmas/definitions/proofs) with **directed edges** encoding “uses/depends-on/cites/generalizes”-type relations. The two most directly aligned and publicly accessible efforts found are:

**ArxiTex + MathXiv + ArxiGraph**: an actively developed, open-source pipeline that parses LaTeX sources from entity["organization","arXiv","preprint server"] papers, extracts theorem-like “artifacts” (theorem/lemma/definition/corollary/remark/example, often with proofs), and constructs a **per-paper directed dependency graph** from explicit references plus optional “inferred dependencies” via LLM-based steps; it exports one JSON graph per paper to a public entity["organization","Hugging Face","ml platform"] dataset, and provides a web UI for interactive exploration. citeturn8view0turn10view0turn13view0turn15view0turn31view0

**TheoremKB**: an open-source research effort aimed at extracting results and proofs and linking them into a **cross-paper theorem dependency graph**. Public code exists (MIT), but its arXiv-derived extracted content is not redistributed in the repo; published work around it evaluates components such as theorem/proof extraction and relationship detection, and a later paper attributes to TheoremKB a dependency graph with **>6 million nodes** extracted from “nearly all mathematics papers” on arXiv (claim from that later paper). citeturn5search0turn35view0turn5search3turn37view3

Outside these, many related projects fall into one of three adjacent categories:

1) **Markup-assisted LaTeX dependency graphs** (explicit `\uses{}` / `\proves{}` or label-reference extraction), which can be applied to arXiv sources but typically require author/curator annotation (example: KnowTeX; also concepts similar to Lean Blueprint / plasTeX-based tools). citeturn41search0turn41search3turn41search4  
2) **Curated statement corpora with explicit IDs and dependency graphs** (e.g., the Stacks Project’s tagged results and dependency graph). citeturn32search31turn32search3turn33search15  
3) **Paper-level (and sometimes section-level) scholarly graphs** (citations/metadata) like OpenAlex, Semantic Scholar, OpenCitations, zbMATH Open; these are powerful infrastructure but typically do not provide statement-level theorem/lemma nodes out of the box. citeturn40search0turn40search1turn40search3turn40search6turn40search2

## What a “statement dependency DAG” requires

A “statement graph” for mathematical literature usually needs three layers of structure:

**Statement segmentation**: identify boundaries of mathematical units (definition/theorem/lemma/proof/example/remark) from LaTeX/PDF/HTML (e.g., theorem-like environments, `\newtheorem` declarations, proof blocks). citeturn8view0turn15view0turn36view0

**Normalization / identity**: assign stable IDs to statements inside a paper (“Theorem 3.2”, `\label{thm:main}`, etc.) and, if going cross-paper, resolve external references (bib entries, arXiv IDs, DOIs, title-author matching). citeturn8view0turn15view0turn11view2

**Edge semantics**: define directed edges such as:
- **Internal reference edges**: proof of A references statement B via `\ref{}` or similar.
- **External-citation edges**: statement A cites external paper X; optionally resolve to a referenced theorem in X if possible.
- **Inferred dependency edges**: “A uses B” even without explicit reference, typically from NLP/LLM inference. citeturn15view0turn8view0

Whether the resulting directed graph is a strict **DAG** depends on design constraints. Many pipelines *intend* “dependency” edges to be acyclic within a paper, but explicit citations and inference can introduce cycles, so most systems conservatively describe the output as a *directed dependency graph* rather than guaranteeing DAG-ness. (This is an inference from how edges are constructed and described, not a guarantee stated in the docs.) citeturn8view0turn15view0

```mermaid
flowchart TD
  arxiv[(arXiv LaTeX/PDF/HTML)] --> ArxiTex[ArxiTex pipeline]
  ArxiTex --> MathXiv[MathXiv per-paper JSON graphs]
  ArxiTex --> ArxiGraph[ArxiGraph UI (Next.js)]

  arxiv --> TheoremKB[TheoremKB extraction + linking]
  TheoremKB --> TKGraph[Cross-paper theorem dependency graph]

  LaTeX[(Any LaTeX project)] --> KnowTeX[KnowTeX markup-assisted graphing]

  Stacks[Stacks Project (tagged results)] --> NP[NaturalProofs dataset]
  ProofWiki[ProofWiki (theorems/proofs)] --> NP

  OpenAlex[OpenAlex works/citations graph] --> ArxiTex
  S2AG[Semantic Scholar Academic Graph API] --> Apps[Downstream graph apps]
```

## Projects that organize statements into graphs

The “most on-target” candidates are the per-paper extraction/graph pipelines for arXiv LaTeX and the cross-paper theorem-dependency efforts. The descriptions below highlight the attributes you requested; when a detail is not available in primary sources found, it is marked **unspecified**.

**ArxiTex (pipeline)** builds “a structured, machine-readable knowledge graph representing the logical dependencies and symbolic definitions within a paper,” extracting theorem-like environments via regex/LaTeX parsing, linking statements via explicit `\ref` and bibliography/citations, and optionally adding LLM-based enhancements for definition extraction and dependency inference. It can export each paper’s graph to JSON intended for a Hugging Face dataset, and includes workflow tooling to discover and process batches of matching arXiv papers. citeturn8view0turn22view0turn11view2  
It also includes an optional step to resolve external bibliography entries to arXiv IDs by detecting explicit arXiv identifiers or querying the arXiv API via heuristic title/author extraction. citeturn11view2  
The repository shows commits in January 2026, indicating active development. citeturn10view0

**MathXiv Knowledge Graphs (dataset)** hosts “per-paper knowledge graphs extracted from mathematical arXiv papers using the ArxiTex pipeline,” explicitly describing nodes (artifacts) and directed edges expressing dependency relations, plus a per-paper “definition bank” and a mapping from artifacts to the terms they use. citeturn15view0  
The dataset is organized as “one JSON file per paper under `data/`,” with an explicit JSON schema sketch including `node_count` and `edge_count` statistics per paper. citeturn15view0  
The dataset’s file tree indicates the `data/` directory is on the order of **hundreds of MB** (784 MB shown at one revision), implying a multi-thousand-paper scale in practice, but the dataset card does not publish an explicit total paper count (so exact coverage is **unspecified**). citeturn31view0turn15view0  
The dataset card warns users to respect rights/terms for underlying papers and notes that ArxiTex code is MIT, but the dataset’s own overall licensing for redistributed derived statement text is **not specified as a single license** in the card excerpted here. citeturn15view0

**ArxiGraph (visualization UI)** is a Next.js frontend “for exploring ArxiTex outputs (document graphs + definition banks)” and can browse JSON exports (e.g., from the Hugging Face dataset) and/or call an ArxiTex backend to process papers on demand. citeturn13view0  
Its repo indicates MIT licensing and commits in January 2026. citeturn13view0turn14view0

image_group{"layout":"carousel","aspect_ratio":"16:9","query":["ArxiGraph ArxiTex knowledge graph UI screenshot","Stacks Project dependency graph visualization tag","Lean blueprint dependency graph document screenshot","KnowTeX LaTeX dependency graph DOT TikZ"],"num_per_query":1}

**TheoremKB (code + research artifacts)** is described as targeting “the unit of information of use… are the mathematical results… and how they rely on each other,” with the project goal of turning the literature “from a collection of papers to a knowledge base of mathematical results.” citeturn36view0  
Its public repository is MIT licensed and in its README describes a dataset of “4400” arXiv articles used for experiments; however, it states it “cannot share” the article content and instead provides paper links metadata (so public raw statement/proof text extracted from arXiv is **not redistributed** via the repo). citeturn5search0turn35view0  
Repository activity shows the latest commits in late 2024, suggesting the open repo is not currently fast-moving (as of the evidence captured). citeturn35view0  
Published evaluation work associated with the project reports “first steps” toward extracting theorems/proofs and identifying references between them from arXiv papers, using a mix of style-based, computer-vision (layout), and NLP approaches. citeturn5search3turn5search0  
A later paper (Connected Theorems) explicitly characterizes TheoremKB as “construct[ing] a theorem dependency graph with over six million nodes extracted from nearly all mathematics papers on arXiv.” citeturn37view3  
(That scale claim is **attributed by the later paper**; it is not independently verified in the repository documentation excerpted above.)

**KnowTeX (LaTeX-first dependency graphs)** is a standalone tool that “analyzes LaTeX projects to construct knowledge dependency graphs among mathematical statements and proofs,” extracting labeled environments and visualizing dependencies via explicit `\uses{...}` and `\proves{...}` annotations. citeturn41search0turn41search4  
The accompanying arXiv writeup describes outputs such as a DOT graph and TikZ-based LaTeX rendering. citeturn41search3  
This is not arXiv-specific but can be applied to exported arXiv source trees if you are willing to add/maintain `\uses`/`\proves` markup (so, extraction is “semi-manual/annotation-based”). citeturn41search0turn41search4

**AutoMathKG (paper-described system; openness unclear)** proposes a “directed graph composed of Definition, Theorem, and Problem entities” with reference relationships as edges, integrating multiple sources including ProofWiki, textbooks, and arXiv papers, using LLMs for augmentation/updates and a vector database for similarity search and fusion. citeturn41search2turn41search1  
However, primary sources found here are the paper and summaries; a public dataset/API/repo for AutoMathKG itself is **unspecified** based on the evidence retrieved. citeturn41search2turn41search1

## Comparison table of candidate projects

The table focuses on projects that (a) explicitly represent mathematical statements as nodes and (b) provide directed edges representing some dependency/usage/citation relation, even if the scope is broader than arXiv.

| name | URL | license | scope | node types | edge types | extraction method | coverage | active? |
|---|---|---|---|---|---|---|---|---|
| **entity["organization","ArxiTex","math statement graphs"]** | GitHub repo citeturn8view0 | MIT citeturn8view0 | arXiv (LaTeX sources), paper-internal graphs; can also resolve external citations to arXiv IDs citeturn8view0turn11view2 | “artifacts” including theorem/lemma/definition/corollary/claim/etc; optional proof text; also external_reference nodes citeturn8view0turn11view2 | explicit internal `\ref` + external `\cite`; optional inferred deps (e.g., used_in/generalizes) citeturn15view0turn8view0 | regex + LaTeX scanning (`\newtheorem`) + bibliography parsing; optional LLM for definition extraction + dependency inference citeturn8view0turn15view0 | depends on what you process; exports one JSON per paper; “discover/process” workflow for batches citeturn8view0turn22view0 | yes (commits Jan 2026) citeturn10view0 |
| **entity["organization","MathXiv Knowledge Graphs","hf dataset per arxiv paper"]** | Hugging Face dataset card citeturn15view0 | dataset-level license: unspecified; code generating it is MIT; underlying papers from arXiv citeturn15view0 | arXiv (mathematics) per-paper graphs citeturn15view0 | artifacts (theorem/lemma/definition/remark/example/…) + optional proof; definition_bank + artifact_to_terms_map citeturn15view0 | directed dependency edges with fields for internal/external reference and inferred dependency_type citeturn15view0 | produced by ArxiTex pipeline (regex/LaTeX + optional LLM) citeturn15view0turn8view0 | one JSON per paper; `data/` directory shown as 784 MB at one revision; total file/paper count unspecified citeturn31view0turn15view0 | yes (dataset commits shown “Jan 7”) citeturn42view0 |
| **entity["organization","ArxiGraph","arxitex web ui"]** | GitHub repo citeturn13view0 | MIT citeturn13view0 | visualization + pipeline runner; loads JSON exports from HF or local backend citeturn13view0 | n/a (viewer for ArxiTex graphs) citeturn13view0 | n/a (visualizes edges already in graph) citeturn13view0 | n/a (frontend) citeturn13view0 | depends on loaded dataset/backends citeturn13view0 | yes (commits Jan 2026) citeturn14view0 |
| **entity["organization","TheoremKB","theorem dependency graph"]** | GitHub repo citeturn5search0 | MIT (repo) citeturn35view0 | arXiv-focused extraction/linking; cross-paper theorem dependency graph goal citeturn36view0turn5search0 | results (theorem/lemma/claim/…) and proofs (exact type schema: partially specified) citeturn36view0turn5search0 | internal/external references between results (goal: dependency graph) citeturn36view0turn5search3 | mixture of style-based/CV/NLP per associated publication; also LaTeX-source instrumentation used in early stages citeturn5search3turn36view0 | repo mentions experiments on 4,400 arXiv articles but does not redistribute content citeturn5search0; later paper attributes >6M theorem nodes (claim external) citeturn37view3 | unclear; repo last commits in Nov 2024 citeturn35view0 |
| **entity["organization","KnowTeX","latex uses proves graphs"]** | GitHub repo / arXiv paper citeturn41search0turn41search3 | unspecified (repo license not captured in retrieved excerpt) citeturn41search0 | LaTeX projects in general (can include arXiv source trees) citeturn41search0turn41search4 | labeled statement/proof environments citeturn41search0turn41search3 | explicit `uses/proves` annotation edges citeturn41search0turn41search4 | manual markup + parser/expander; outputs DOT and TikZ citeturn41search0turn41search3 | per-project; no global corpus citeturn41search3 | yes (paper Dec 2025/Jan 2026; repo exists) citeturn41search3turn41search0 |
| **entity["organization","AutoMathKG","llm math knowledge graph"]** | arXiv paper citeturn41search2turn41search1 | unspecified (public code/data not identified here) citeturn41search2 | multi-source: ProofWiki + textbooks + arXiv + TheoremQA citeturn41search2turn41search1 | Definition / Theorem / Problem entities citeturn41search2 | reference relationships; (edge taxonomy details unspecified in abstract) citeturn41search2 | LLM-based augmentation and update mechanisms + vector DB retrieval citeturn41search2 | “wide coverage” (quantitative corpus size not in abstract snippet) citeturn41search2 | unspecified |
| **entity["book","The Stacks Project","algebraic geometry reference"]** | project site citeturn33search19 | GNU FDL 1.2+ (stated in project PDF intro) citeturn33search15turn33search3 | curated textbook/reference (not arXiv) citeturn33search25 | tagged items (sections/lemmas/theorems/…) citeturn32search31turn32search3 | explicit dependency graph between tags citeturn32search3turn32search7 | manual authoring + cross-reference tracking (project-maintained) citeturn32search7turn32search3 | very large; dependency metrics show thousands of tags in some dependency closures citeturn32search3 | yes (continuously maintained project) citeturn33search25 |

## Related efforts and enabling infrastructure

The projects above live in a broader ecosystem of tools and datasets that can be combined to approximate “theorems-as-a-service,” but typically stop short of a full, automatically extracted DAG of arXiv mathematical statements.

**Curated natural-language theorem corpora**  
entity["organization","ProofWiki","mathematical proofs wiki"] is an online wiki of mathematical definitions and proofs; its pages are explicitly statement-like, and the site indicates content is available under a Creative Commons Attribution-ShareAlike license (version not specified on the main page excerpt). citeturn34search2turn34search5  
The NeurIPS dataset paper “Mathematical Theorem Proving in Natural Language” reports using a ProofWiki XML dump and states ProofWiki is licensed under CC BY‑SA 3.0, while also using a snapshot of the Stacks Project (licensed under GNU FDL). citeturn34search27turn33search15  
This illustrates a recurring pattern: statement corpora exist, but the “graph” structure is often implicit (hyperlinks/citations), requiring downstream mining to obtain explicit dependency edges. citeturn34search27turn34search2

**Formal Abstracts and “theorems-as-a-service” direction**  
entity["organization","Formal Abstracts","formal theorem statement service"] explicitly aims to “establish a formal abstract service” expressing results of publications in computer-readable form that captures semantic content; the main repository notes it is “currently in the design phase” and is MIT licensed. citeturn32search0turn32search8  
This direction is closer to “theorems-as-a-service,” but it is fundamentally a curation/formalization pipeline rather than automated extraction from arXiv at scale. citeturn32search0turn32search8

**Formal proof libraries and dependency-graph tooling**  
Formal proof assistants naturally carry fine-grained dependency graphs (by construction) among declared objects. For example, entity["organization","coq-dpdgraph","coq dependency graph plugin"] is a Coq/Rocq plugin that extracts dependencies between Coq objects and produces files for graph visualization; it is LGPL‑2.1 licensed. citeturn32search1turn32search5  
entity["organization","Isabelle","isabelle proof assistant"] includes a “graph browser” for visualizing theory dependency graphs in its system tooling (as documented in the Isabelle System Manual). citeturn32search17  
The formal-math library entity["organization","mathlib","lean formal math library"] is described as a large community-driven library of formalized mathematics (in Lean), representing the “formal DAG” end of the spectrum. citeturn32search12  
These systems are not arXiv extraction projects, but they provide the cleanest data model for statement nodes and dependency edges, and they motivate hybrid approaches (e.g., aligning informal arXiv statements to formal library nodes). citeturn32search13turn32search1turn32search12

**Scholarly metadata/citation graphs (paper-level infrastructure)**  
entity["organization","OpenAlex","open scholarly catalog"] provides an API and snapshot describing scholarly entities (works/authors/sources/etc.) and their connections, and its documentation emphasizes that the full dataset is licensed under CC0. citeturn40search0turn40search4turn40search18  
entity["organization","Semantic Scholar","ai2 scholarly search"] offers a REST API for publication/author/citation metadata (“Academic Graph API”) and hosts downloadable datasets; additionally, entity["organization","S2ORC","semantic scholar open research corpus"] is a large NLP-oriented corpus released under ODC‑By 1.0. citeturn40search1turn40search9turn40search5  
entity["organization","OpenCitations","open citations infrastructure"] states that data held in its datasets are available under CC0. citeturn40search3turn40search7  
entity["organization","zbMATH Open","math reviews and metadata"] exposes a REST API and notes legal/coverage constraints; an EMS Magazine article states API data is provided under CC‑BY‑SA 4.0 (subject to publisher constraints), and the API terms & conditions emphasize the API does not contain complete content but includes bibliographic data, classifications, and sometimes abstracts. citeturn40search6turn40search2turn40search29  
These services are crucial complements: they can anchor paper identities, citations, venues, and sometimes section-level metadata—but they do not natively model *theorem-level* nodes and edges like “Lemma X uses Definition Y.” citeturn40search18turn40search1turn40search6

## Analytical assessment of readiness and gaps

The evidence suggests there *are* public/open projects building statement graphs from arXiv content, but the space is still early and fragmented.

**Closest match to your target (open + statement nodes + directed dependency edges)**  
ArxiTex + MathXiv + ArxiGraph is currently the clearest end-to-end open stack: it (1) extracts theorem-like artifacts and proofs from LaTeX sources, (2) represents them as graph nodes with per-paper stats (node_count/edge_count), (3) includes edge metadata that distinguishes internal/external references and optional inferred dependency types, and (4) provides both dataset export and an interactive UI plus an API backend. citeturn8view0turn15view0turn13view0turn11view2turn10view0  
Its main unresolved “research-hard” challenges are precisely the ones you highlight: robustness across LaTeX styles, meaningful edge typing beyond `\ref`/`\cite`, and evaluation/quality metrics. In the core docs retrieved, explicit quantitative accuracy metrics for edge correctness are **unspecified**. citeturn8view0turn15view0

**Cross-paper theorem dependency graphs remain constrained by data rights and weak supervision**  
TheoremKB’s framing and associated work explicitly aims at a knowledge base of results across papers, and later literature cites it as operating at multi-million-node scale across mathematics arXiv. citeturn36view0turn37view3  
But in publicly accessible artifacts, redistribution of extracted statement/proof text is limited (repo states it “cannot share” the underlying article content), which directly impacts reproducibility and open evaluation. citeturn5search0turn40search10  
Where quantitative evaluation is present, it is often component-level (e.g., theorem/proof segmentation or reference classification) rather than end-to-end validation of a “true” dependency DAG—reflecting the lack of gold-standard theorem-dependency annotations at scale. citeturn5search3turn36view0

**Markup-assisted tools are practical but don’t solve extraction at scale**  
KnowTeX (and similarly positioned annotation-first approaches) can generate clean dependency graphs because edge semantics come directly from explicit commands like `\uses` and `\proves`. citeturn41search0turn41search4  
However, this shifts the core problem from NLP extraction to human/author curation—useful for pedagogy, formalization planning, or authoring new documents, but not a standalone solution for mining the existing arXiv corpus in bulk. citeturn41search0turn41search4

**The strongest “statement graphs” today are curated or formal, not mined**  
The Stacks Project provides a large-scale example of a curated dependency graph where statement identity (“tags”) and dependency structure are first-class; its statistics explicitly describe dependency graph sizes in the thousands for certain tags. citeturn32search3turn32search31  
Formal libraries provide even cleaner dependency graphs, but aligning them with informal arXiv statements remains an active research problem rather than a solved engineering task. citeturn32search13turn32search12turn32search1

**Net conclusion**  
Yes—there are public/open efforts to turn arXiv math papers into statement-level directed graphs, most notably ArxiTex/MathXiv and TheoremKB, but the “full vision” (broad arXiv coverage, rich edge semantics, strong evaluation, and an openly redistributable cross-paper DAG of normalized statements) remains only partially achieved due to (i) extraction difficulty, (ii) statement identity resolution across papers, and (iii) licensing/redistribution constraints for derived textual content. citeturn15view0turn5search0turn40search10turn10view0