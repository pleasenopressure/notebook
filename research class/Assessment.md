This is a very typical **“beginner’s search result”**: mixed quality, and the core concepts are confused.

As we emphasized in our previous discussion, your topic is **“Identification of Malicious Actors”**, **not** simple **“Fake News Detection.”**

In the list your teammate found, **more than half of the papers are actually doing text classification (NLP)**.  
If you use these directly in your review, the professor will consider it **off-topic**.

Below is a detailed evaluation and filtering guide, mapped to the page structure I suggested earlier:

---

## 🚨 Red Alert — Suggest Removing (Reject / Low Priority)

These papers mainly focus on _text content_ or _non-technical perspectives_, deviating from the core task: **detecting malicious accounts**.

### **No. 2 (Word and sentence embeddings)**

**Evaluation:** Off-topic.  
The title clearly indicates using word embeddings to detect fake news. That’s pure NLP text classification—deciding whether _a sentence is fake_, not whether _a user is malicious or a bot_.

### **No. 3 (A-SQUARE CNN)**

**Evaluation:** Off-topic.  
CNN is typically used for text or image content. Unless it explicitly applies CNN to _user behavior sequences_, this is almost certainly content detection.

### **No. 4 (ConFake … content-based features)**

**Evaluation:** Off-topic.  
The title literally says _content-based features_.  
Your review should focus on **User / Graph / Behavior features**, not text content.

### **No. 11 (Literacy Strategies)**

**Evaluation:** Too “soft.”  
This belongs to social science/education (improving media literacy), not technical CS/AI research.

---

## ✅ Green Light — High-Quality and Relevant

These papers fit your topic well and can be placed directly into specific pages.

### **Suitable for Page 3: Graph-based / Propagation**

### **No. 12 & No. 15 (Bi-Directional Graph Convolutional Networks)**

> Note: These two entries are the same paper (duplicate).

**Evaluation:** A classic SOTA paper (AAAI).  
Bi-GCN is a representative work using graph structure in rumor detection. It considers not only content but also the **propagation tree**.

**Usage:**  
Place in **Page 3** or **Page 5** to demonstrate that

> “Propagation structure of malicious information differs from normal users.”

---

### **No. 8 (Epidemic diffusion model)**

**Evaluation:** Relevant.  
“Source detecting” and “diffusion models” are typical network-science approaches. Useful for analyzing how propagation paths reveal malicious sources.

---

## Suitable for Page 2: Feature / Behavioral methods

### **No. 13 (Detecting Automation … Human, Bot, or Cyborg)**

**Evaluation:** Foundational work.  
This is likely the classic paper by Chu et al. (2010/2012). Despite age, it is historically important.

**Usage:**  
Place at the beginning of **Page 2**.  
Use it to introduce early rule-based/manual-feature methods (entropy, posting frequency, etc.), then lead into modern deep learning. You _must_ cite this to show historical understanding.

---

## Suitable for Page 4: User + Content Hybrid

### **No. 16 (User Preference-aware Fake News Detection)**

**Evaluation:** Highly relevant (ACM).  
“User Preference-aware” indicates integrating **user profiles** into the model—exactly the kind of user-focused detection your topic requires.

---

## Suitable for Page 6: Intro / Gaps

### **No. 1 (Survey on Explainable AI)**

**Evaluation:** Keep it for reading, not as one of the main 15 papers.  
Since it is a **survey**, don't treat it as a primary technical paper.

**Usage:**  
Read its _limitations/future work_ section and adapt those points into your **Page 6 Research Gaps** (e.g., lack of interpretability).

---

## ⚠️ Yellow Warning — Needs Checking

These items only provide URLs or vague titles. You must open the abstract to judge.

### **No. 5 ("fake news" AND "spreader")**

Contains the keyword “Spreader”—a good sign.  
If it focuses on identifying _spreaders_, keep it. If it’s about _spreading_, also potentially useful.

### **No. 6 (Cyber-Physical Social Services)**

Title sounds buzzword-heavy.  
Check whether it proposes a concrete algorithm or just a conceptual framework. If conceptual only → remove.

### **No. 7, 9, 10, 14 (URL-only)**

I cannot see titles. Use this quick filter:

**Check the first sentence of the abstract:**

- If it’s about **news is fake?** → reject
    
- If it’s about **user is malicious?** → keep
    

**Check publication year:**  
Prefer papers from the last **3–5 years**, unless foundational like No.13.

**Check venue:**  
Springer / IEEE / ACM / AAAI / IJCAI → usually acceptable.

---

## 📝 Action Plan (What You Should Tell Your Team)

To show professional judgment as an AI master's student in Meeting 0, say:

### **1. Deduplicate**

> “No. 12 and No. 15 are duplicates—keep the AAAI version.”

### **2. Remove off-topic papers**

> “No. 2, 3, and 4 are text-classification-only. Our topic is _Malicious Actors_, so it must focus on users/accounts, not content. We should replace these.”

### **3. Add missing directions**

> “We currently have only one strong graph-based paper (No. 12).  
> Graph methods are now SOTA—let’s add 1–2 GAT or heterogeneous graph papers  
> (e.g., BotRGCN, GraphRFI).”

### **4. Assign core papers**

- **Page 2 (Feature/Behavior):** No. 13
    
- **Page 3 (Graph):** No. 12, No. 8
    
- **Page 4 (User + Content):** No. 16
    
