# **Structured & Translated Version (English)**

## **1. Clarifying the Research Scope**

A crucial first step in the Introduction is to clearly delineate **Content Detection** from **Actor Detection**, as these are frequently conflated.

- **False Information Detection:**  
    Focuses on determining whether the _content_ (news/posts) is false.  
    _Keywords:_ NLP, Fact-checking.
    
- **Malicious Actor Detection (Your Topic):**  
    Focuses on determining **who** is producing or distributing problematic content.  
    _Keywords:_ User Profiling, Graph Structure Analysis, Behavioral Analysis.
    

> **Recommendation:** Your survey should concentrate on the latter—detecting **users/accounts**, not merely analyzing **text**.

---

## **2. Building a Key Features Matrix**

Based on the professor’s “feature analysis” approach, you can design a cross-paper comparison matrix across the following dimensions. This matrix also supports workload division within your group.

|**Feature Dimension**|**Description & Keywords**|
|---|---|
|**Detection Target**|Whether the model targets **individual accounts** (bots) or **coordinated groups** (botnets, coordinated inauthentic behavior).|
|**Core Methodology**|**Feature-based** (traditional ML), **Graph-based** (GNN, GCN, GAT focusing on relationship graphs), **NLP/LLM-based**, **Multimodal** (text + image).|
|**Input Features**|**Metadata** (creation time, username), **Content** (post text), **Structure** (follower networks), **Temporal** (posting frequency, time series patterns).|
|**Supervision Strategy**|**Supervised**, **Semi-supervised**, or **Unsupervised** (clustering/anomaly detection).|
|**Adversarial Robustness**|Ability to resist evasion attacks (e.g., bots imitating human resting patterns).|

---

## **3. Recommended Structure & Group Division (5 Pages)**

With five members (one page per person), a **method/feature-based structure** is stronger than a chronological one.  
Suggested page assignments:

- **Page 2: Feature- and Behavior-based Detection**  
    Traditional ML (Random Forest, SVM), behavioral sequence modeling (RNN, LSTM), and the extraction of "behavioral fingerprints."
    
- **Page 3: Graph-based Approaches**  
    GCN, GAT, and other GNN architectures; discussion of homophily vs. heterophily in malicious actors' connection patterns.
    
- **Page 4: Content- and Semantics-based Methods**  
    BERT, Transformers, LLM-driven text pattern detection; the dual role of LLMs as _both_ generators and detectors.
    
- **Page 5: Coordinated Group / Campaign Detection**  
    Identifying coordinated actions rather than single accounts (e.g., synchronized retweeting clusters).
    
- **Page 6: Generalization & Adversarial Challenges**  
    Cross-platform transferability, model robustness, and the evolving nature of bot behavior.
    

---

## **4. Critical Analysis Requirements (The “Not False” Perspective)**

To achieve a high score (70%+), integrate the following critical dimensions:

1. **Reliability of Ground Truth**  
    Many datasets (e.g., Twitter bot datasets) rely on heuristic or crowd-annotated labels.  
    _Are those accounts truly malicious, or just highly active?_  
    If labels are unreliable, model evaluation becomes questionable—echoing the professor’s **“Not False”** concept.
    
2. **Arms Race Dynamics**  
    Detection methods often lag behind attacker strategies.  
    Modern bots may use LLMs to generate human-like text—can current SOTA still detect them?
    
3. **Explainability**  
    High-performing GNNs are frequently black boxes.  
    When platforms suspend accounts, **explainability** becomes ethically necessary.
    

---

## **5. How Your Page Should Use the Feature Matrix**

Your page should not summarize assigned papers individually. Instead, use the **feature matrix** to build a logical argument about your assigned method.

### **Example: If You Are Responsible for Page 3 (Graph-based Approaches)**

### **(1) Core Methodology**

Rather than listing each paper:

> Unlike feature-based methods (see Page 2), graph-based approaches assume structural homophily among malicious accounts. **Paper A** pioneered GCN for topology modeling, while **Paper B** introduced GAT to handle heterogeneous or deceptive linking behaviors by weighting neighbors via attention.

### **(2) Input Features**

Use the matrix to create contrast:

> Purely structural models (e.g., Paper A) struggle on isolated nodes. **Paper C** addresses this by integrating text features as node attributes, forming a multimodal graph and mitigating sparsity issues.

### **(3) Performance & Limitations**

Use the matrix to identify shared gaps:

> Although Papers A and B report >95% accuracy under supervised settings, they lack adversarial evaluation. Paper C shows that when bots deliberately sever connections to reduce detectability, GNN performance drops by ~20%. This highlights a major gap: **graph models excel under static conditions but lack adversarial robustness**.

---

## **6. Practical Writing Workflow**

When you receive your assigned papers:

### **Step 1 — Fill the Matrix**

Force yourself to extract each paper’s stance on every feature dimension.  
If a paper does _not_ discuss adversarial robustness: mark **“Not addressed.”**

### **Step 2 — Identify Patterns**

Look down each column.  
If all assigned papers ignore robustness → that becomes your core critique.

### **Step 3 — Write Using Feature Dimensions, Not Paper Order**

Your conclusions should follow this style:

> “Current graph-based research largely overlooks adversarial robustness, representing a substantial and underexplored research gap.”

This is exactly the type of reasoning your professor expects for the “Not False” criterion.
