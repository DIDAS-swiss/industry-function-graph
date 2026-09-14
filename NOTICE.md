# Origin and attribution

The original work and implementation in this repository are © 2026 Accelerate GmbH and were contributed to the DIDAS ecosystem. Third-party classifications, standards, specifications and referenced materials remain subject to their respective rights and terms.

Canonical upstream: https://github.com/Accelerate-GmbH/industry-function-graph
DIDAS publication: https://github.com/DIDAS-swiss/industry-function-graph

# Third-Party Content Notices

This repository's own content is licensed [CC BY 4.0](LICENSE-CONTENT) and its
build scripts [MIT](LICENSE). Nothing below is covered by either grant.

This repository renders parts of several statistical classifications as SKOS
concepts so that use cases can be filed against them. Only the codes and titles
needed for the mapped use cases are carried, with attribution; **none of these
classifications is reproduced in full, and none of them is redistributed as a
dataset.** Each remains under the terms of its publisher.

| Source | Terms | Used in |
|---|---|---|
| [ISIC Rev. 5](https://unstats.un.org/unsd/classifications/Econ/isic) (United Nations Statistics Division) | UN publication; codes and titles taken from the official structure file and referenced with attribution, not reproduced in full — only the 22 sections plus the divisions and classes the use cases reach | `data/sectors.csv` — the sector layer (`skos:notation`) |
| [NACE Rev. 2.1](https://ec.europa.eu/eurostat/web/nace) (Eurostat) | European Commission / Eurostat; codes referenced with attribution | `data/sectors.csv` — `ifm:naceRev21Code`, at section and division level only |
| [NOGA 2025](https://www.kubb-tool.bfs.admin.ch/en) (Swiss Federal Statistical Office) | Referenced for a structural cross-check only; no NOGA code is stored in this repository | The section/division alignment check recorded in README.md |
| [Classification of Business Functions](https://unece.org/trade/statistics) (UNECE / Eurostat) | Referenced with attribution; labels here are marked `codeStatus "provisional"` until checked against the official publication | `data/cbf.csv` — mapping target for the function layer |
| [APQC Process Classification Framework](https://www.apqc.org/process-frameworks) (APQC) | Published by APQC under its own terms; **not redistributed here** — only the category numbers and names an alignment references are carried, marked `codeStatus "provisional"` | `data/apqc-pcf.csv` — mapping target for the function layer |

Every concept carries an `ifm:codeStatus` saying how far it has been checked
against the official publication. See [README.md](README.md) § "Provenance and
honesty".

## Use cases referenced

Six of the seeded use cases in `data/use-cases.csv` link to trust flow diagrams
in the [DIDAS Trust Flow Diagram Repository](https://github.com/DIDAS-swiss/Trust-Flow-Diagram-Repository),
which publishes them under [CC BY 4.0](https://github.com/DIDAS-swiss/Trust-Flow-Diagram-Repository/blob/main/LICENSE-CONTENT).
The diagrams themselves are not copied here — `documented_by` is a link, and the
descriptions in this repository are its own summaries.

Those diagrams are not official flows of the swiyu team, the Swiss
Confederation, or any other authority.
