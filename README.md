<div align="center">

# 🏛️ The Land Intelligence Layer — **Bhu-Sanvad**
### Intelligent Land Record Digitization, Conflict Verification & Cadastral Graph Engine

[![SIH 2026](https://img.shields.io/badge/SIH-2026-orange?style=for-the-badge&logo=india&logoColor=white)](https://www.sih.gov.in/)
[![Problem Statement](https://img.shields.io/badge/PS-SIH26018-blue?style=for-the-badge)](https://www.sih.gov.in/)
[![Team](https://img.shields.io/badge/Team-183252%20%7C%20MATRIX123-8A2BE2?style=for-the-badge)](#)
[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-async-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.x-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)](https://pytorch.org/)
[![MongoDB](https://img.shields.io/badge/MongoDB-7.x-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Neo4j](https://img.shields.io/badge/Neo4j-Graph-008CC1?style=for-the-badge&logo=neo4j&logoColor=white)](https://neo4j.com/)
[![PostGIS](https://img.shields.io/badge/PostGIS-Cadastral-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgis.net/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green?style=for-the-badge)](./LICENSE)
[![Architecture](https://img.shields.io/badge/Architecture-Middleware%20Layer%20%7C%20HITL-informational?style=for-the-badge)](#-1-system-architecture--data-flow)

**Problem Statement:** `SIH26018` — Smart Automation · Software Edition
**Sponsoring Agency:** Department of Land Resources (DoLR), Ministry of Rural Development, Govt. of India

> **One-liner:** An evidence-backed AI middleware that turns degraded, cursive historical Indian revenue registers into a *verified, temporal ownership graph* — flagging conflicts before they become litigation, and never letting a machine overrule a Patwari.

</div>

---

## 📑 Table of Contents

1. [Why This Exists](#-why-this-exists)
2. [System Architecture & Data Flow](#-1-system-architecture--data-flow)
3. [Core Subsystem Breakdown](#-2-core-subsystem-breakdown)
   - [2.1 Vision & Indic Document Understanding](#21-vision--indic-document-understanding-pipeline)
   - [2.2 Algorithmic Conflict Verification](#22-algorithmic-conflict-verification--invariant-checking)
   - [2.3 Temporal Land-Story Graph](#23-graph-based-temporal-land-story-reconstruction)
   - [2.4 Evidence-Backed RAG](#24-evidence-backed-rag-layer)
   - [2.5 Cryptographic Trust & Audit](#25-cryptographic-trust--audit-trails)
4. [Repository Structure](#-3-repository-structure)
5. [Local Setup & Installation](#-4-local-setup--installation)
6. [API Reference](#-5-api-reference-core-endpoints)
7. [Benchmarks, Impact & National Alignment](#-6-benchmarks-impact--national-alignment)
8. [Roadmap & Contribution Guidelines](#-7-roadmap--contribution-guidelines)

---

## 🎯 Why This Exists

Land-record modernization under DILRMP has digitized large volumes of scans, but **scanning is not understanding**. Historical *Jamabandi*, *Khasra*, *Khatauni* and *Mutation (Intiqal/Dakhil-Kharij)* registers are:

- handwritten in regional cursive scripts (Devanagari/Kaithi-influenced, Urdu-Persian numerals, Gurmukhi, Gujarati, Odia, Kannada, etc.),
- degraded — ink bleed, foxing, torn margins, fold lines, and overlapping *muhar* (seal) stamps,
- semantically inconsistent — areas that do not sum, mutations that pre-date the acquisition they inherit from, shares that exceed 100%.

Naive OCR yields character strings; **it does not yield trustworthy records**. Unvalidated records are the seed of civil litigation. Bhu-Sanvad ("Land Dialogue") is the missing *intelligence layer* between the scan and the Record of Rights.

| Failure Mode of Naive OCR | Bhu-Sanvad Countermeasure |
|---|---|
| Stamp/seal overlaps text → garbage tokens | Morphological seal isolation + inpainting before HTR |
| Uneven ink & yellowed paper → broken strokes | Sauvola adaptive binarization + deskew/denoise |
| Free-text output, no structure | LayoutLMv3 form-aware entity extraction |
| Silent numerical inconsistencies | Formal invariant checker with tolerance ε |
| Black-box "AI says so" | Citation-backed RAG → pixel bounding boxes on the original folio |
| Tamperable digital copies | SHA-256 hash chain + append-only audit ledger |
| Automation replacing officials | Mandatory Human-In-The-Loop sign-off by Patwari/Tehsildar |

---

## 🧭 1. System Architecture & Data Flow

### 1.1 End-to-End Pipeline (Mermaid)

```mermaid
flowchart TD
    A[📄 Raw Scans / PDFs<br/>Jamabandi · Khasra · Khatauni · Mutation Registers] --> B

    subgraph ING[Stage 1 — Ingestion & Preprocessing]
        B[FastAPI /ingest<br/>async batching] --> B1[SHA-256 of raw bytes<br/>→ genesis hash]
        B1 --> B2[Celery + Redis queue]
        B2 --> B3[Denoise · Deskew · Sauvola Binarize<br/>Seal / Stamp Masking]
    end

    B3 --> C

    subgraph HTR[Stage 2 — Indic HTR & Layout Understanding]
        C[LayoutLMv3<br/>region / table / field detection] --> C1[Line & word segmentation]
        C1 --> C2[IndicTrOCR / CRNN+CTC<br/>script-adaptive recognition]
        C2 --> C3[Post-correction<br/>lexicon + BHASHINI-assisted normalization]
    end

    C3 --> D

    subgraph EXT[Stage 3 — Structured Extraction]
        D[Entity Extraction<br/>Owner · Khasra · Khata · Area · Share · Mutation# · Date]
        D --> D1[(MongoDB<br/>doc metadata · bboxes · audit)]
    end

    D --> E

    subgraph GRAPH[Stage 4 — Graph Builder & Conflict Checker]
        E[Graph Builder<br/>Owner→Plot→Deed lineage] --> E1[(Neo4j / NetworkX<br/>Temporal Ownership DAG)]
        E --> E2[(PostGIS<br/>Bhu-Naksha polygons)]
        E1 --> E3[Invariant Conflict Checker<br/>Area · Time · Share · Continuity]
        E2 --> E3
    end

    E3 --> F

    subgraph RAG[Stage 5 — Retrieval & Reasoning]
        F[Chunker → Embeddings → FAISS index] --> F1[Retriever<br/>top-k evidence with bbox]
        F1 --> F2[Local SLM / open-weights LLM<br/>citation-constrained explanation]
    end

    F2 --> G

    subgraph TRUST[Stage 6 — Cryptographic Ledger]
        G[Hash Chain<br/>H_i = SHA256 prev ‖ doc ‖ meta]
        G --> G1[(Append-only Audit Ledger)]
    end

    G --> H

    subgraph HITL[Stage 7 — Human-In-The-Loop]
        H[🖥️ Patwari / Tehsildar Dashboard<br/>Leaflet · D3 timeline · bbox overlay]
        H -->|Approve / Reject / Amend| I[DILRMP / RoR Sync Adapter]
    end

    H -.feedback labels.-> C2
    H -.audit event.-> G1
```

### 1.2 ASCII Reference Diagram

```text
┌──────────────┐   ┌────────────────────┐   ┌─────────────────────────┐   ┌──────────────────────┐
│ Raw Scans /  │──▶│ Ingestion &        │──▶│ Indic HTR & Layout      │──▶│ Structured Entity    │
│ PDFs (TIFF,  │   │ Preprocessing      │   │ LayoutLMv3 + IndicTrOCR │   │ Extraction           │
│ JPEG, PDF)   │   │ denoise/deskew/    │   │ / CRNN+CTC              │   │ owner, khasra, area, │
└──────────────┘   │ Sauvola + seal mask│   └─────────────────────────┘   │ share, mutation, date│
                   └────────────────────┘                                 └──────────┬───────────┘
                                                                                     │
        ┌────────────────────────────────────────────────────────────────────────────┘
        ▼
┌───────────────────────────┐   ┌──────────────────────────┐   ┌──────────────────────────────┐
│ Graph Builder             │──▶│ Invariant Conflict       │──▶│ FAISS Indexing & RAG         │
│ Neo4j/NetworkX + PostGIS  │   │ Checker (area, temporal, │   │ evidence → bbox → SLM        │
│ ownership DAG             │   │ share, continuity)       │   │ citation-backed explanation  │
└───────────────────────────┘   └──────────────────────────┘   └──────────────┬───────────────┘
                                                                              │
                        ┌─────────────────────────────────────────────────────┘
                        ▼
              ┌───────────────────────┐   ┌───────────────────────────┐   ┌───────────────────┐
              │ Cryptographic Ledger  │──▶│ HITL Patwari / Tehsildar  │──▶│ DILRMP / RoR Sync │
              │ SHA-256 hash chain    │   │ Decision Dashboard        │   │ (on human approval)│
              └───────────────────────┘   └───────────────────────────┘   └───────────────────┘
```

### 1.3 Design Principles

- **Middleware, not replacement** — reads from and writes back to DILRMP / Bhu-Naksha / state RoR systems through adapters; no rip-and-replace.
- **Decision support, not decision making** — every state-changing action requires an authenticated human approval, logged on the ledger.
- **Evidence or silence** — the LLM may only state what it can cite; uncited claims are dropped by a post-generation validator.
- **Deterministic core, probabilistic edge** — conflict detection is symbolic/numerical (reproducible); AI is used for perception and explanation only.

---

## 🧠 2. Core Subsystem Breakdown

### 2.1 Vision & Indic Document Understanding Pipeline

**Stage order:** `load → deskew → denoise → illumination-normalize → seal-mask → Sauvola binarize → layout → line-segment → HTR → post-correct`

#### a) Adaptive Binarization — Sauvola

Global (Otsu) thresholding fails on stained, unevenly lit folios. Sauvola computes a *local* threshold from the windowed mean $m(x,y)$ and standard deviation $s(x,y)$:

$$
T(x,y) = m(x,y)\left[1 + k\left(\frac{s(x,y)}{R} - 1\right)\right]
$$

with $k \in [0.2, 0.5]$ and dynamic range $R = 128$ for 8-bit images. Window size is chosen adaptively from estimated stroke width (median of the distance transform on a coarse Otsu pass).

```python
# models/pipeline/binarize.py
import cv2
import numpy as np

def sauvola(gray: np.ndarray, window: int = 51, k: float = 0.25, R: float = 128.0) -> np.ndarray:
    """Vectorised Sauvola thresholding using box filters (O(1) per pixel)."""
    g = gray.astype(np.float64)
    mean = cv2.boxFilter(g, -1, (window, window), normalize=True, borderType=cv2.BORDER_REFLECT)
    sq   = cv2.boxFilter(g * g, -1, (window, window), normalize=True, borderType=cv2.BORDER_REFLECT)
    std  = np.sqrt(np.maximum(sq - mean * mean, 0.0))
    thr  = mean * (1.0 + k * (std / R - 1.0))
    return ((g > thr).astype(np.uint8)) * 255

def estimate_window(gray: np.ndarray) -> int:
    _, coarse = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)
    dist = cv2.distanceTransform(coarse, cv2.DIST_L2, 3)
    stroke = max(1.0, 2.0 * float(np.median(dist[dist > 0])) if (dist > 0).any() else 3.0)
    w = int(round(stroke * 9))
    return w if w % 2 == 1 else w + 1
```

#### b) Morphological Seal / Stamp Isolation

Revenue registers carry circular/rectangular *muhar* stamps in red, blue or violet ink that overlap the script. We isolate them in HSV chroma space, refine by morphology and shape priors (circularity), then **inpaint** so the HTR never sees them — while the stamp mask itself is stored as an evidence artefact (stamps carry legal weight and are preserved as separate regions).

```python
# models/pipeline/seal_mask.py
import cv2
import numpy as np

STAMP_RANGES = [  # (lower HSV, upper HSV) — red wraps around hue 0/180
    ((0,   70, 50), (10, 255, 255)),
    ((170, 70, 50), (180, 255, 255)),
    ((100, 60, 40), (135, 255, 255)),  # blue / violet ink
]

def isolate_seals(bgr: np.ndarray, min_area: int = 1500, min_circularity: float = 0.35):
    hsv = cv2.cvtColor(bgr, cv2.COLOR_BGR2HSV)
    mask = np.zeros(hsv.shape[:2], np.uint8)
    for lo, hi in STAMP_RANGES:
        mask |= cv2.inRange(hsv, np.array(lo), np.array(hi))

    k_close = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (15, 15))
    k_open  = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (5, 5))
    mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, k_close)  # bridge ring gaps
    mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN,  k_open)   # drop speckle

    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    seal_mask, seals = np.zeros_like(mask), []
    for c in contours:
        area, peri = cv2.contourArea(c), cv2.arcLength(c, True)
        if area < min_area or peri == 0:
            continue
        circularity = 4.0 * np.pi * area / (peri * peri)
        if circularity >= min_circularity:
            cv2.drawContours(seal_mask, [c], -1, 255, cv2.FILLED)
            seals.append({"bbox": cv2.boundingRect(c), "circularity": round(circularity, 3)})

    seal_mask = cv2.dilate(seal_mask, np.ones((7, 7), np.uint8))
    clean = cv2.inpaint(bgr, seal_mask, 5, cv2.INPAINT_TELEA)
    return clean, seal_mask, seals
```

#### c) Layout Parsing & HTR

| Step | Model | Output |
|---|---|---|
| Layout / table / field detection | **LayoutLMv3** (fine-tuned on annotated revenue-register templates) | Typed regions: `header`, `khasra_table`, `owner_cell`, `remarks`, `seal`, `signature` |
| Line & word segmentation | Projection profiles + connected-component clustering | Ordered line crops with reading-order index |
| Script-adaptive recognition | **IndicTrOCR** (transformer) primary; **CRNN + CTC** lightweight fallback for edge/CPU | Text + per-character confidence |
| Post-correction | Domain lexicon (village names, *gotra*/surname lists, revenue terms) + edit-distance re-ranking | Normalized tokens, numerals mapped to ASCII |
| Numeral normalization | Devanagari/Urdu/Gurmukhi/etc. → ASCII digits, fraction parsing (`३/८` → `3/8`) | Machine-checkable numbers |

CRTC training objective for the CRNN branch:

$$
\mathcal{L}_{CTC} = -\ln \sum_{\pi \in \mathcal{B}^{-1}(\mathbf{y})} \prod_{t=1}^{T} p_t(\pi_t \mid \mathbf{x})
$$

where $\mathcal{B}$ collapses repeats and blanks. Low-confidence lines (mean char-prob $< \tau_{htr}$, default `0.82`) are **routed to the human queue** rather than silently accepted.

---

### 2.2 Algorithmic Conflict Verification & Invariant Checking

Extracted entities are validated against a formal invariant set. Each invariant is a pure function `records → List[Violation]` so results are reproducible and unit-testable.

#### I1 — Area Conservation

For a parent *Khata/Khasra* of area $A_{khata}$ split into $n$ sub-plots:

$$
\left|\sum_{i=1}^{n} \text{SubPlotArea}_i - \text{TotalKhataArea}\right| \le \epsilon
$$

where $\epsilon = \max(\epsilon_{abs},\ \epsilon_{rel} \cdot A_{khata})$ — defaults: $\epsilon_{abs}=0.0005$ ha, $\epsilon_{rel}=0.5\%$ (absorbs unit-conversion rounding of bigha/biswa/kanal/marla → hectare).

#### I2 — Mutation Temporal Acyclicity & Order

For every mutation $m$ that derives from parent acquisition $p$:

$$
T_{mutation}(m) > T_{parent\_acquisition}(p) \quad \wedge \quad G_{lineage}\ \text{is a DAG}
$$

Cycle detection uses Kahn's topological sort; any residual node set is reported as a *circular ownership* violation.

#### I3 — Share Conservation (Inheritance)

For a plot with co-owners at time $t$:

$$
\sum_{j \in \text{Owners}(t)} s_j = 1 \pm \delta, \qquad 0 < s_j \le 1
$$

#### I4 — Transfer Continuity (Chain of Title)

A seller in transaction $k$ must be a holder of record after transaction $k-1$:

$$
\text{Seller}_k \in \text{Owners}\big(\text{Plot},\ t_{k-1}^{+}\big)
$$

#### I5 — Cadastral–Textual Consistency

Recorded area versus PostGIS polygon area (Bhu-Naksha geometry, projected CRS):

$$
\frac{\left|A_{record} - A_{geom}\right|}{A_{geom}} \le \rho \quad (\rho = 0.05\ \text{default})
$$

| ID | Invariant | Severity | Typical Real-World Cause |
|---|---|---|---|
| I1 | Area conservation | 🔴 High | Fraudulent over-partition, transcription error |
| I2 | Temporal order / acyclicity | 🔴 High | Backdated mutation, forged lineage |
| I3 | Share conservation | 🟠 Medium | Missed heir, double-counted share |
| I4 | Chain-of-title continuity | 🔴 High | Sale by non-owner, missing intermediate mutation |
| I5 | Record vs. geometry | 🟡 Low–Med | Encroachment, survey drift |

```python
# graph/invariants.py
from __future__ import annotations
from dataclasses import dataclass
from datetime import date
from decimal import Decimal
from typing import Iterable
import networkx as nx

@dataclass(frozen=True)
class Violation:
    code: str
    severity: str
    khasra_id: str
    message: str
    evidence_node_ids: tuple[str, ...]

def check_area_conservation(khata_area: Decimal, sub_areas: Iterable[Decimal], khasra_id: str,
                            eps_abs=Decimal("0.0005"), eps_rel=Decimal("0.005")) -> list[Violation]:
    total = sum(sub_areas, Decimal(0))
    eps = max(eps_abs, eps_rel * khata_area)
    diff = abs(total - khata_area)
    if diff > eps:
        return [Violation("I1_AREA", "HIGH", khasra_id,
                          f"Σ sub-plots = {total} ha vs khata {khata_area} ha (Δ={diff} > ε={eps})", ())]
    return []

def check_temporal_dag(G: nx.DiGraph, khasra_id: str) -> list[Violation]:
    out: list[Violation] = []
    if not nx.is_directed_acyclic_graph(G):
        cycle = nx.find_cycle(G)
        out.append(Violation("I2_CYCLE", "HIGH", khasra_id,
                             "Circular ownership lineage detected", tuple(str(u) for u, _ in cycle)))
    for parent, child, data in G.edges(data=True):
        t_parent: date = G.nodes[parent].get("acquired_on")
        t_child:  date = G.nodes[child].get("mutation_date")
        if t_parent and t_child and not (t_child > t_parent):
            out.append(Violation("I2_ORDER", "HIGH", khasra_id,
                       f"Mutation {child} ({t_child}) not after parent acquisition {parent} ({t_parent})",
                       (str(parent), str(child))))
    return out

def check_share_sum(shares: dict[str, Decimal], khasra_id: str, delta=Decimal("0.001")) -> list[Violation]:
    s = sum(shares.values(), Decimal(0))
    if abs(s - 1) > delta:
        return [Violation("I3_SHARE", "MEDIUM", khasra_id, f"Owner shares sum to {s}, expected 1", ())]
    return []
```

---

### 2.3 Graph-Based Temporal Land Story Reconstruction

Every extracted entity becomes a typed node; every legal event becomes a *time-stamped edge*. The resulting **Temporal Ownership Knowledge Graph** is a directed acyclic graph (DAG) ordered by legal effective date — a land parcel's "story" is a path query.

**Node types:** `Person` · `Khata` · `KhasraPlot` · `Deed/Mutation` · `Court/Order` · `Folio` (source scan page)
**Edge types:** `OWNS{from,to,share}` · `TRANSFERRED_BY` · `INHERITED_FROM` · `SPLIT_INTO` · `MERGED_INTO` · `EVIDENCED_BY{bbox}`

```mermaid
graph LR
    P1((Ram Lal<br/>owner 1962)) -- "OWNS 1.0<br/>1962–1987" --> K1[Khasra 214/1<br/>2.400 ha]
    P1 -- "died 1987" --> M1{{Mutation #3391<br/>Inheritance 1988}}
    M1 -- "INHERITED 0.5" --> P2((Shyam Lal))
    M1 -- "INHERITED 0.5" --> P3((Geeta Devi))
    K1 -- "SPLIT_INTO" --> K2[214/1/A<br/>1.200 ha]
    K1 -- "SPLIT_INTO" --> K3[214/1/B<br/>1.200 ha]
    P2 -- "OWNS 1.0" --> K2
    P3 -- "SALE deed #8812 · 1996" --> P4((Kishan Rao))
    P4 -- "OWNS 1.0" --> K3
    M1 -. "EVIDENCED_BY bbox" .-> F1[[Folio 47 · x:212 y:640 w:388 h:52]]
```

**Neo4j schema & a "land story" query:**

```cypher
// graph/cypher/constraints.cypher
CREATE CONSTRAINT person_id IF NOT EXISTS FOR (p:Person)     REQUIRE p.person_id IS UNIQUE;
CREATE CONSTRAINT plot_id   IF NOT EXISTS FOR (k:KhasraPlot) REQUIRE k.khasra_id IS UNIQUE;
CREATE CONSTRAINT deed_id   IF NOT EXISTS FOR (d:Deed)       REQUIRE d.deed_id   IS UNIQUE;

// Full chain of title for one plot, chronologically ordered
MATCH path = (p:Person)-[o:OWNS]->(k:KhasraPlot {khasra_id: $khasra_id})
OPTIONAL MATCH (d:Deed)-[:TRANSFERRED_BY|INHERITED_FROM]->(p)
RETURN p.name AS owner, o.share AS share, o.valid_from AS from, o.valid_to AS to,
       d.deed_id AS via, d.deed_type AS type, d.sha256 AS deed_hash
ORDER BY o.valid_from ASC;
```

**Reconstruction algorithm (simplified):**

1. **Entity resolution** — fuzzy match owner names across folios (Jaro–Winkler on transliterated names + father's-name + village co-occurrence) to merge spelling variants into one `Person`.
2. **Event ordering** — sort mutations by `effective_date`; resolve missing dates via register serial-number monotonicity.
3. **Edge synthesis** — inheritance events create fractional `OWNS` edges; sales transfer full share; partitions create `SPLIT_INTO` children.
4. **Validation pass** — run invariants I1–I5 on each connected component.
5. **Publish** — persist to Neo4j; export `GraphJSON` for the D3 timeline in the dashboard.

*NetworkX* is used for in-memory batch validation and for deployments where a Neo4j server is not available (offline tehsil kiosks).

---

### 2.4 Evidence-Backed RAG Layer

**Goal:** when a conflict is flagged, the Patwari sees a plain-language explanation whose every sentence is *anchored to pixels in the original scan*.

```text
Conflict (I1_AREA @ Khasra 214/1)
   │
   ├─▶ Query builder: "khasra 214/1 sub-plot area partition 1996 mutation"
   ├─▶ FAISS top-k retrieval over chunked deed/register text
   │      chunk = { text, folio_id, page, bbox[x,y,w,h], line_conf, sha256 }
   ├─▶ Cross-encoder rerank (evidence relevance)
   ├─▶ Local SLM prompt (evidence-only, JSON output, citation ids mandatory)
   └─▶ Citation validator: every claim must reference a retrieved chunk_id, else dropped
            │
            ▼
   Explanation + [chunk_id → folio bbox] → highlighted on the scanned image in the dashboard
```

```python
# rag/retriever.py
import faiss, json, numpy as np
from sentence_transformers import SentenceTransformer

class EvidenceRetriever:
    def __init__(self, index_path: str, meta_path: str,
                 model_name: str = "intfloat/multilingual-e5-base"):
        self.index = faiss.read_index(index_path)              # IndexFlatIP over L2-normalised vectors
        self.meta  = [json.loads(l) for l in open(meta_path, encoding="utf-8")]
        self.enc   = SentenceTransformer(model_name)

    def search(self, query: str, k: int = 6, min_score: float = 0.35):
        q = self.enc.encode([f"query: {query}"], normalize_embeddings=True).astype("float32")
        scores, ids = self.index.search(q, k)
        return [
            {**self.meta[i], "score": float(s)}
            for s, i in zip(scores[0], ids[0]) if i != -1 and s >= min_score
        ]   # each hit carries folio_id, page, bbox, sha256
```

**Grounded generation contract** (enforced by the validator, not just the prompt):

```json
{
  "summary": "Sub-plots 214/1/A and 214/1/B total 2.900 ha but the parent khata records 2.400 ha.",
  "claims": [
    { "text": "Khata area is 2.400 ha.",            "evidence": ["chk_0193"] },
    { "text": "Partition entry lists 1.200 + 1.700 ha.", "evidence": ["chk_0207", "chk_0208"] }
  ],
  "confidence": 0.91,
  "requires_human_review": true
}
```

Claims with empty or non-retrieved `evidence` IDs are **discarded** before display. The model runs locally (open-weights SLM via `llama.cpp`/`vLLM`), so no land data leaves the deployment boundary.

---

### 2.5 Cryptographic Trust & Audit Trails

- **Document integrity:** SHA-256 (FIPS 180-4) of the raw uploaded bytes at ingestion → immutable *document fingerprint*.
- **Chain linking:** every lifecycle event (ingest, HTR complete, conflict flagged, human decision, sync) appends a block:

$$
H_i = \text{SHA-256}\big(H_{i-1} \,\Vert\, \text{SHA-256}(\text{payload}_i) \,\Vert\, \text{ts}_i \,\Vert\, \text{actor}_i\big), \qquad H_0 = \text{SHA-256}(\text{raw\_bytes})
$$

- **Tamper evidence:** altering any past payload changes its hash and invalidates every subsequent $H_j,\ j \ge i$.
- **Access control:** JWT + RBAC roles — `PATWARI`, `TEHSILDAR`, `AUDITOR`, `ADMIN`, `SERVICE`. Only `PATWARI`/`TEHSILDAR` may sign decisions; `AUDITOR` is read-only.
- **Abuse protection:** per-token and per-IP rate limiting (sliding window, Redis-backed).

```python
# api/security/ledger.py
import hashlib, json, time

def sha256_hex(b: bytes) -> str:
    return hashlib.sha256(b).hexdigest()

def append_block(prev_hash: str, payload: dict, actor: str) -> dict:
    ts = int(time.time())
    payload_hash = sha256_hex(json.dumps(payload, sort_keys=True, ensure_ascii=False).encode())
    block_hash = sha256_hex(f"{prev_hash}|{payload_hash}|{ts}|{actor}".encode())
    return {"prev": prev_hash, "payload_hash": payload_hash, "ts": ts,
            "actor": actor, "hash": block_hash}

def verify_chain(genesis: str, blocks: list[dict]) -> tuple[bool, int | None]:
    prev = genesis
    for i, b in enumerate(blocks):
        expect = sha256_hex(f"{prev}|{b['payload_hash']}|{b['ts']}|{b['actor']}".encode())
        if b["prev"] != prev or b["hash"] != expect:
            return False, i          # index of first corrupted block
        prev = b["hash"]
    return True, None
```

**MongoDB document schema (excerpt):**

```json
{
  "_id": "doc_01J9ZK4N7Q",
  "source": { "filename": "jamabandi_1962_p47.tiff", "district": "Tumakuru", "village": "Kyatsandra", "script": "Kannada" },
  "sha256": "9f2c1e5b7a4d3c...e81a",
  "pages": [
    { "page": 47, "regions": [
        { "type": "khasra_table", "bbox": [212, 640, 388, 52], "text": "214/1  2.400  रामलाल", "conf": 0.93 }
    ]}
  ],
  "status": "PENDING_REVIEW",
  "ledger_head": "b71d90...c04f",
  "created_at": "2026-09-14T08:41:12Z"
}
```

**PostGIS table (cadastral geometry):**

```sql
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE cadastral_parcel (
    ulpin        CHAR(14) UNIQUE,                 -- Bhu-Aadhaar
    khasra_id    TEXT PRIMARY KEY,
    village_code TEXT NOT NULL,
    area_ha      NUMERIC(12,4) NOT NULL,
    geom         geometry(MultiPolygon, 4326) NOT NULL
);
CREATE INDEX idx_parcel_geom ON cadastral_parcel USING GIST (geom);

-- I5: recorded vs. geometric area (projected to metric CRS)
SELECT khasra_id, area_ha,
       ST_Area(ST_Transform(geom, 32643)) / 10000.0 AS geom_ha
FROM cadastral_parcel
WHERE ABS(area_ha - ST_Area(ST_Transform(geom, 32643)) / 10000.0)
      / NULLIF(ST_Area(ST_Transform(geom, 32643)) / 10000.0, 0) > 0.05;
```

---

## 🗂️ 3. Repository Structure

```text
bhu-sanvad/
├── api/                          # FastAPI service
│   ├── main.py                   # app factory, lifespan, routers
│   ├── config.py                 # pydantic-settings (.env)
│   ├── routers/
│   │   ├── ingest.py             # POST /api/v1/ingest
│   │   ├── records.py            # GET  /api/v1/records/{khasra_id}/conflicts
│   │   ├── rag.py                # POST /api/v1/rag/explain
│   │   ├── verify.py             # POST /api/v1/verify/integrity
│   │   └── decisions.py          # POST /api/v1/decisions (HITL sign-off)
│   ├── security/
│   │   ├── auth.py               # JWT issue/verify
│   │   ├── rbac.py               # role guards
│   │   ├── ratelimit.py          # Redis sliding window
│   │   └── ledger.py             # SHA-256 hash chain
│   ├── schemas/                  # Pydantic request/response models
│   └── workers/
│       ├── celery_app.py
│       └── tasks.py              # preprocess → htr → extract → graph → index
├── models/
│   ├── pipeline/
│   │   ├── preprocess.py         # deskew, denoise, illumination normalisation
│   │   ├── binarize.py           # Sauvola
│   │   ├── seal_mask.py          # morphological stamp isolation
│   │   ├── layout.py             # LayoutLMv3 inference
│   │   ├── htr.py                # IndicTrOCR / CRNN+CTC
│   │   ├── postcorrect.py        # lexicon + edit-distance
│   │   └── extract.py            # entity extraction & numeral normalisation
│   ├── training/                 # fine-tuning scripts, CTC configs, augmentations
│   └── weights/                  # (git-ignored) model checkpoints
├── graph/
│   ├── builder.py                # entity resolution + edge synthesis
│   ├── invariants.py             # I1–I5 checkers
│   ├── neo4j_store.py
│   ├── networkx_store.py
│   └── cypher/                   # constraints.cypher, story queries
├── rag/
│   ├── chunker.py                # line/region-aware chunking with bbox carry-over
│   ├── indexer.py                # FAISS build/update
│   ├── retriever.py
│   ├── generator.py              # local SLM + citation validator
│   └── prompts/
├── geo/
│   ├── bhunaksha_adapter.py      # parcel geometry ingestion
│   └── migrations/               # PostGIS DDL
├── integrations/
│   ├── dilrmp_sync.py            # RoR/DILRMP adapter (post human approval)
│   └── bhashini_client.py        # language services
├── frontend/
│   ├── index.html
│   ├── css/
│   ├── js/
│   │   ├── map.js                # Leaflet / OpenLayers parcel view
│   │   ├── timeline.js           # D3 ownership timeline
│   │   ├── folio_viewer.js       # scan viewer + bbox overlay
│   │   └── decisions.js          # approve / reject / amend
│   └── assets/
├── tests/
│   ├── unit/                     # invariants, ledger, binarization
│   ├── integration/              # API + worker + DB
│   ├── e2e/                      # sample folio → dashboard
│   └── fixtures/                 # synthetic degraded folios, golden JSON
├── docker/
│   ├── Dockerfile.api
│   ├── Dockerfile.worker
│   ├── docker-compose.yml        # api, worker, redis, mongo, neo4j, postgis
│   └── nginx.conf
├── scripts/
│   ├── seed_sample_data.py
│   └── build_faiss_index.py
├── docs/                         # architecture notes, ADRs, data-privacy notes
├── .env.example
├── requirements.txt
├── pyproject.toml
├── Makefile
├── LICENSE
└── README.md
```

---

## ⚙️ 4. Local Setup & Installation

### 4.1 Prerequisites

| Requirement | Version | Notes |
|---|---|---|
| Python | 3.11+ | `python3.11 -m venv` |
| CUDA / PyTorch | CUDA 12.x, torch 2.x | Optional; CPU fallback uses the CRNN+CTC branch |
| Docker & Compose | 24+ | For Mongo, Redis, Neo4j, PostGIS |
| MongoDB | 7.x | Document metadata & audit logs |
| PostgreSQL + PostGIS | 15 / 3.4 | Cadastral polygons |
| Redis | 7.x | Celery broker + rate limiter |
| Tesseract / poppler-utils | latest | PDF rasterisation (`pdftoppm`) |

### 4.2 Installation

```bash
# 1. Clone
git clone https://github.com/matrix123-sih/bhu-sanvad.git
cd bhu-sanvad

# 2. Virtual environment
python3.11 -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate

# 3. Dependencies (choose ONE torch line matching your hardware)
pip install --upgrade pip
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121   # GPU
# pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu   # CPU only
pip install -r requirements.txt

# 4. Environment configuration
cp .env.example .env
```

**`.env.example`**

```dotenv
APP_ENV=development
API_HOST=0.0.0.0
API_PORT=8000
JWT_SECRET=change-me-with-32+random-bytes
JWT_EXPIRE_MINUTES=60
RATE_LIMIT_PER_MIN=120

MONGO_URI=mongodb://localhost:27017
MONGO_DB=bhu_sanvad
REDIS_URL=redis://localhost:6379/0
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=neo4j_dev_password
POSTGIS_DSN=postgresql://bhu:bhu_pw@localhost:5432/cadastre

HTR_MODEL=ai4bharat/IndicTrOCR
LAYOUT_MODEL=microsoft/layoutlmv3-base
HTR_CONF_THRESHOLD=0.82
EMBED_MODEL=intfloat/multilingual-e5-base
LLM_BACKEND=local
LLM_MODEL_PATH=./models/weights/slm.gguf
FAISS_INDEX_PATH=./data/faiss/land.index
AREA_EPS_ABS_HA=0.0005
AREA_EPS_REL=0.005
```

```bash
# 5. Start infrastructure (Mongo, Redis, Neo4j, PostGIS)
docker compose -f docker/docker-compose.yml up -d mongo redis neo4j postgis

# 6. Database migrations / constraints
psql "$POSTGIS_DSN" -f geo/migrations/001_cadastral_parcel.sql
cypher-shell -u neo4j -p "$NEO4J_PASSWORD" -f graph/cypher/constraints.cypher
python -m api.db.init_mongo           # creates collections + indexes (sha256 unique, TTL for jobs)

# 7. (Optional) seed synthetic degraded folios and build the FAISS index
python scripts/seed_sample_data.py
python scripts/build_faiss_index.py

# 8. Run the API
uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload

# 9. In a second terminal: Celery worker
celery -A api.workers.celery_app worker --loglevel=INFO --concurrency=2 -Q ingest,htr,graph

# 10. Frontend (static)
cd frontend && python -m http.server 5173
```

Open **http://localhost:8000/docs** for Swagger UI and **http://localhost:5173** for the Patwari dashboard.

```bash
# Run tests
pytest -q tests/unit tests/integration
```

---

## 🔌 5. API Reference (Core Endpoints)

Base URL: `/api/v1` · Auth: `Authorization: Bearer <JWT>` · Content type `application/json` unless noted.

| Method | Endpoint | Role | Request Payload | Success | Errors |
|---|---|---|---|---|---|
| `POST` | `/api/v1/ingest` | `PATWARI`, `SERVICE` | `multipart/form-data`: `files[]` (TIFF/JPEG/PDF), `district`, `village_code`, `script_hint?` | `202 Accepted` + `job_id` | `400` bad file, `401`, `403`, `413` too large, `415` unsupported type, `429` rate limited |
| `GET` | `/api/v1/jobs/{job_id}` | any auth | – | `200` job state & progress | `404` |
| `GET` | `/api/v1/records/{khasra_id}/conflicts` | `PATWARI`, `TEHSILDAR`, `AUDITOR` | Query: `severity?`, `include_evidence=true` | `200` conflict list | `401`, `403`, `404` unknown khasra |
| `POST` | `/api/v1/rag/explain` | `PATWARI`, `TEHSILDAR` | `{ "khasra_id", "conflict_id", "language" }` | `200` grounded explanation | `404`, `422` schema, `429`, `503` LLM unavailable |
| `POST` | `/api/v1/verify/integrity` | any auth | `{ "document_id", "sha256"?}` or file upload | `200` verification result | `404`, `409` hash mismatch, `422` |
| `POST` | `/api/v1/decisions` | `PATWARI`, `TEHSILDAR` | `{ "conflict_id", "action": "APPROVE\|REJECT\|AMEND", "note" }` | `201 Created` + ledger block | `403`, `404`, `409` already decided |

### 5.1 Examples

**Ingest**

```bash
curl -X POST http://localhost:8000/api/v1/ingest \
  -H "Authorization: Bearer $TOKEN" \
  -F "files=@samples/jamabandi_1962_p47.tiff" \
  -F "district=Tumakuru" -F "village_code=KA-TMK-0142" -F "script_hint=kannada"
```
```json
{
  "job_id": "job_7f3a91c2",
  "status": "QUEUED",
  "documents": [
    { "document_id": "doc_01J9ZK4N7Q", "sha256": "9f2c1e5b7a4d3c88b2f06a1d5e7c4b90a3d21f6e8c5b7a49d0e3f1a2b4c6e81a" }
  ],
  "ledger_head": "b71d90a4e2c35f1d8a6b09e7c4d2f3a15b8e6c07d9a4f1b2e3c50a8d6f7e2c04"
}
```

**Conflicts**

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "http://localhost:8000/api/v1/records/214-1/conflicts?include_evidence=true"
```
```json
{
  "khasra_id": "214-1",
  "ulpin": "KA0142214100017",
  "conflicts": [
    {
      "conflict_id": "cf_00412",
      "code": "I1_AREA",
      "severity": "HIGH",
      "message": "Σ sub-plots = 2.900 ha vs khata 2.400 ha (Δ=0.500 > ε=0.012)",
      "evidence": [
        { "folio_id": "doc_01J9ZK4N7Q", "page": 47, "bbox": [212, 640, 388, 52], "ocr_text": "214/1  2.400", "conf": 0.93 },
        { "folio_id": "doc_01J9ZK4N7Q", "page": 48, "bbox": [198, 312, 402, 48], "ocr_text": "214/1/B  1.700", "conf": 0.88 }
      ],
      "status": "OPEN"
    }
  ]
}
```

**RAG Explain**

```bash
curl -X POST http://localhost:8000/api/v1/rag/explain \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"khasra_id":"214-1","conflict_id":"cf_00412","language":"en"}'
```
```json
{
  "conflict_id": "cf_00412",
  "summary": "Sub-plots 214/1/A and 214/1/B total 2.900 ha but the parent khata records 2.400 ha.",
  "claims": [
    { "text": "Parent khata area is 2.400 ha.", "evidence": [{ "chunk_id": "chk_0193", "folio_id": "doc_01J9ZK4N7Q", "page": 47, "bbox": [212, 640, 388, 52] }] },
    { "text": "Sub-plot 214/1/B is recorded as 1.700 ha in the 1996 partition entry.", "evidence": [{ "chunk_id": "chk_0208", "folio_id": "doc_01J9ZK4N7Q", "page": 48, "bbox": [198, 312, 402, 48] }] }
  ],
  "suggested_actions": ["Verify 214/1/B area against field measurement", "Check whether 0.5 ha is a mis-read digit (1.200 → 1.700)"],
  "requires_human_review": true,
  "model": "local-slm", "confidence": 0.91
}
```

**Integrity Verification**

```bash
curl -X POST http://localhost:8000/api/v1/verify/integrity \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"document_id":"doc_01J9ZK4N7Q","sha256":"9f2c1e5b7a4d3c88b2f06a1d5e7c4b90a3d21f6e8c5b7a49d0e3f1a2b4c6e81a"}'
```
```json
{
  "document_id": "doc_01J9ZK4N7Q",
  "hash_match": true,
  "chain_valid": true,
  "blocks_verified": 9,
  "first_corrupted_block": null,
  "ledger_head": "b71d90a4e2c35f1d8a6b09e7c4d2f3a15b8e6c07d9a4f1b2e3c50a8d6f7e2c04",
  "standard": "FIPS 180-4 (SHA-256)"
}
```

### 5.2 Status Code Summary

| Code | Meaning |
|---|---|
| `200` | OK |
| `201` | Decision recorded and ledger block appended |
| `202` | Job accepted for async processing |
| `400 / 415 / 422` | Malformed request / unsupported media / validation error |
| `401 / 403` | Unauthenticated / role not permitted |
| `404` | Resource not found |
| `409` | Hash mismatch or conflicting state |
| `413` | Payload too large |
| `429` | Rate limit exceeded (`Retry-After` header set) |
| `503` | Downstream model or DB unavailable |

---

## 📊 6. Benchmarks, Impact & National Alignment

### 6.1 Comparison

> ⚠️ **Reading this table:** the *Manual* column reflects typical field practice ranges; the *Land Intelligence Layer* column lists **design targets** for the pilot evaluation protocol below (held-out, expert-annotated folios stratified by script, degradation level and seal overlap). Report your own measured values from `tests/eval/` before quoting them externally.

| Dimension | Manual Patwari Workflow | Generic Cloud OCR | **Land Intelligence Layer (Bhu-Sanvad)** |
|---|---|---|---|
| Handwritten Indic cursive accuracy (CER) | n/a (human reading) | High CER on degraded cursive; not tuned for revenue vocabulary | **Target CER ≤ 8% on degraded folios** with domain lexicon post-correction |
| Stamp / seal overlap handling | Human judgment | Text corrupted by overlap | **Dedicated seal isolation + inpainting** |
| Time per folio | Tens of minutes (read, cross-check, key in) | Seconds, but output unusable without rework | **Target < 30 s machine time + minutes of focused review** |
| Structure (fields, tables) | Manual | Flat text / generic layout | **LayoutLMv3 typed fields** |
| Cross-record consistency checks | Ad hoc, memory-based | None | **Formal invariants I1–I5, every record** |
| Explainability | Verbal / informal | None | **Citations → pixel bounding boxes** |
| Tamper evidence | Paper trail only | None | **SHA-256 hash chain, verifiable on demand** |
| Data residency | On paper | Data leaves to third-party cloud | **On-prem / NIC-cloud deployable; local SLM** |
| Compliance posture | Established | Not designed for land-record legal use | **HITL by design; RBAC; audit ledger** |
| Scalability | Linear with headcount | High | **Horizontal via Celery workers** |

**Evaluation metrics:** Character Error Rate (CER) & Word Error Rate (WER) for HTR · field-level F1 for entity extraction · conflict-detection precision/recall against a seeded-error test set · Recall@k and citation-faithfulness rate for RAG · median reviewer time per folio.

### 6.2 Impact

- **Litigation prevention** — inconsistencies are surfaced at digitization time, before a dispute exists.
- **Officer productivity** — Patwaris review *flagged exceptions* instead of re-reading every folio.
- **Trust** — every automated statement is traceable to a pixel region and a hash-verified source scan.
- **Inclusivity** — multilingual, multi-script by design, not English-first.

### 6.3 Alignment with National Initiatives

| Initiative | How Bhu-Sanvad Aligns |
|---|---|
| **DILRMP** (Digital India Land Records Modernization Programme) | Acts as a validation and structuring layer over scanned records; outputs sync back through the RoR adapter only after human approval. |
| **SVAMITVA** | Ownership graph and PostGIS parcel model can ingest drone-survey property boundaries to cross-check rural *abadi* records (Invariant I5). |
| **BHASHINI** | Language services for script/language normalization, translation of explanations to the officer's preferred language, and vernacular UI strings. |
| **Bhu-Aadhaar / ULPIN** | Each parcel is keyed to a 14-character unique land parcel identifier (`cadastral_parcel.ulpin`), enabling unambiguous cross-system joins. |
| **Bhu-Naksha** | Cadastral polygon ingestion and geometry-vs-record consistency checks. |

---

## 🗺️ 7. Roadmap & Contribution Guidelines

### 7.1 Roadmap

| Phase | Milestone |
|---|---|
| **Now (SIH 2026 prototype)** | End-to-end pipeline, I1–I5 invariants, FAISS RAG with bbox citations, hash-chain ledger, HITL dashboard |
| **Next** | 📱 **Edge scanning app** — on-device capture guidance (blur/skew/glare feedback), offline CRNN inference, encrypted sync when connectivity returns |
| **Next** | ☁️ **Integration with State Data Centres / NIC cloud** — containerized deployment, Keycloak/SSO with state identity, MeghRaj-compatible hosting |
| **Later** | 🗺️ **Automated cadastral vector boundary realignment** — snap textual survey descriptions and drone orthomosaics to Bhu-Naksha polygons; propose corrected geometries for human approval |
| **Later** | Active-learning loop: officer corrections continuously fine-tune HTR & layout models |
| **Later** | Federated evaluation across states without centralizing sensitive records |

### 7.2 Git Workflow

We use a **trunk-based flow with short-lived feature branches** and protected `main`.

```text
main         ← protected, release-ready, CI must pass
 └─ develop  ← integration branch
     ├─ feat/<area>-<short-desc>       e.g. feat/htr-sauvola-window-adapt
     ├─ fix/<area>-<short-desc>        e.g. fix/graph-cycle-report
     ├─ docs/<short-desc>              e.g. docs/api-examples
     ├─ chore/<short-desc>             e.g. chore/bump-fastapi
     └─ hotfix/<short-desc>            (branches from main)
```

**Commit convention:** [Conventional Commits](https://www.conventionalcommits.org/) — `feat(graph): add share-conservation invariant I3`.

```bash
git checkout develop && git pull
git checkout -b feat/rag-citation-validator
# ...work...
pytest -q && ruff check . && black --check . && mypy api graph rag
git commit -m "feat(rag): drop claims with unretrieved evidence ids"
git push -u origin feat/rag-citation-validator
```

### 7.3 Pull Request Checklist

- [ ] Branch name follows the convention above and targets `develop`
- [ ] Unit tests added/updated; `pytest -q` passes locally
- [ ] `ruff`, `black`, and `mypy` are clean
- [ ] No secrets, real citizen data, or unmasked Aadhaar/PII in code, fixtures, or logs
- [ ] New invariants include a golden-case test (pass **and** violation)
- [ ] API changes are reflected in the OpenAPI schema and this README
- [ ] Any change touching the ledger preserves backward-verifiable hashes (add a migration test)
- [ ] HITL guarantee preserved — no code path mutates official records without an authenticated human decision
- [ ] PR description explains *what*, *why*, and how it was validated; screenshots for UI changes

### 7.4 Data & Ethics Notes

- Use **synthetic or officially released, anonymized** folios for development and demos.
- Model outputs are **advisory**; the Patwari/Tehsildar remains the legal decision-maker.
- Report security issues privately via `SECURITY.md`, not public issues.

---

<div align="center">

**Team MATRIX123 · Team ID 183252 · Smart India Hackathon 2026 · PS SIH26018**

*Turning scanned paper into verified truth — one folio at a time.* 🇮🇳

Licensed under the [Apache License 2.0](./LICENSE)

</div>
