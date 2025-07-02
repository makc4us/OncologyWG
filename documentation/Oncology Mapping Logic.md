# Mapping Logic for Oncology Vocabularies

This document outlines the mapping logic for oncology vocabularies. The goal of this mapping is to ensure consistent and accurate representation of oncology concepts, supporting comprehensive data analysis and interoperability.

## Key Mapping Principles

### Oncologic Conditions
* All concepts representing **oncologic conditions** (e.g., "Breast Cancer," "Lung Neoplasm") must be mapped to the **Condition** domain.
* If a concept has lexical equivalents across multiple domains (e.g., SNOMED Observation, ICDO3 Condition), the **Condition** domain takes precedence, regardless of vocabulary priority.

### Tumor Staging and Grading
* All concepts related to **tumor staging and grading** (e.g., "cT4d," "IB") must be mapped to the **Measurement** domain within the **Cancer Modifier** vocabulary.
* If the Cancer Modifier vocabulary is unavailable, **SNOMED** is the secondary target vocabulary.

### Metastasis and Tumor Dimensions
* Concepts representing **metastasis, lymph node involvement, and tumor/metastasis dimensions** must be mapped to the **Measurement** domain within the **Cancer Modifier** vocabulary.

### Genomic Abnormalities
* All concepts related to **genetic abnormalities** must be mapped to the **OMOP Genomic Measurement** domain, using the relationship `relationship_id = 'Has variant'`.

### Vaccines
* All vaccine concepts must be categorized using the **CVX** vocabulary.

### Anesthesia
* Granular anesthesia concepts (e.g., "Anesthesia for puncture") must be mapped to the general concept of "**Administration of anesthesia**".

### AJCC/UICC Categorization
* All in situ cancer concepts (e.g., "Carcinoma in situ, NOS, of breast, NOS") must be additionally categorized using the **AJCC/UICC** vocabulary (specifically, the concept representing **AJCC/UICC Tis Category**).

### Grade and Resultative Measurements
* Measurements that have **grade components, or resultative values (Lab tests), have to be postcoordinated**.

---

## SNOMED Mapping Logic

This section defines the mapping logic for selected SNOMED CT Standard concepts. The aim is to align clinically relevant oncology terms with the OMOP Common Data Model to preserve semantic integrity, support composability, and enable robust real-world evidence analysis.

Mappings are grouped by transformation patterns according to SNOMED source domain and the required OMOP representation.

### 1. Post-coordination of Measurement Concepts: Measurement → Measurement + Meas Value

**Mapping Logic**
Some SNOMED Measurement concepts encode both the test and its qualitative result. These are split into measurement + result using post-coordination.

**Example:**

| source\_concept\_id | source\_concept\_name                 | source\_domain\_id | relationship\_id | target\_concept\_id | target\_concept\_name        | target\_domain\_id |
| :------------------ | :----------------------------------- | :----------------- | :--------------- | :------------------ | :--------------------------- | :----------------- |
| 40482494            | High carcinoembryonic antigen level  | Measurement        | Maps to          | 4197913             | CA 125 measurement           | Measurement        |
| 40482494            | High carcinoembryonic antigen level  | Measurement        | Maps to value    | 4084765             | Above reference range        | Meas Value         |
| 4134634             | No metastases                        | Measurement        | Maps to          | 36769180            | Metastasis                   | Measurement        |
| 4134634             | No metastases                        | Measurement        | Maps to value    | 9189                | Negative                     | Meas Value         |
| 4245252             | Prostate specific antigen above reference range | Measurement        | Maps to          | 4272032             | Prostate specific antigen measurement | Measurement        |
| 4245252             | Prostate specific antigen above reference range | Measurement        | Maps to value    | 4084765             | Above reference range        | Meas Value         |
| 4331508             | Cancer antigen 125 above reference range | Measurement        | Maps to          | 4197913             | CA 125 measurement           | Measurement        |
| 4331508             | Cancer antigen 125 above reference range | Measurement        | Maps to value    | 4084765             | Above reference range        | Meas Value         |
| 4245698             | Tumor metastasis to non-regional lymph nodes absent | Condition          | Maps to          | 36769243            | Distant spread to lymph node | Measurement        |
| 4245698             | Tumor metastasis to non-regional lymph nodes absent | Condition          | Maps to value    | 9189                | Negative                     | Meas Value         |
| 4263144             | Tumor size, largest metastasis       | Measurement        | Maps to          | 36769180            | Metastasis                   | Measurement        |
| 4263144             | Tumor size, largest metastasis       | Measurement        | Maps to          | 36769446            | Dimension of Largest Metastasis | Measurement        |

### 2. Domain Reassignment: Observation → Condition

**Mapping Logic**
Many concepts originally categorized under Observation in SNOMED are clinically interpreted as conditions. This mapping pattern corrects the semantic domain for use in clinical staging or diagnosis.

**Example:**

| source\_concept\_id | source\_concept\_name | source\_domain\_id | relationship\_id | target\_concept\_id | target\_concept\_name     | target\_domain\_id |
| :------------------ | :-------------------- | :----------------- | :--------------- | :------------------ | :----------------------- | :----------------- |
| 37312397            | Benign insulinoma     | Observation        | Maps to          | 4111803             | Endocrine pancreatic adenoma | Condition          |

### 3. Mapping SNOMED Staging Concepts to Cancer Modifier

Certain SNOMED Meas Value concepts representing general stage groupings (e.g., Stage 0, Stage 1, etc.) require mapping into the **Cancer Modifier** vocabulary to unify staging logic under AJCC-compliant architecture. This is necessary to ensure consistent staging representation across data sources.

This correction ensures semantic integrity and improves analytical validity in cohort stratification and staging-based modeling.

**Example:**

| source\_concept\_id | source\_concept\_name | source\_domain\_id | source\_vocabulary\_id | relationship\_id | target\_concept\_id | target\_concept\_name | target\_domain\_id | target\_vocabulary\_id |
| :------------------ | :-------------------- | :----------------- | :--------------------- | :--------------- | :------------------ | :-------------------- | :----------------- | :--------------------- |
| 4127500             | Stage 0               | Meas Value         | SNOMED                 | Maps to          | 1634946             | Stage 0               | Measurement        | Cancer Modifier        |

### 4. Improvements in Location Concepts

Some location-related SNOMED concepts used in oncology ETLs are overly broad or non-specific. Mapping them to more precise admission-related concepts improves the clinical granularity and semantic hierarchy, which benefits location-based analytics (e.g., identifying patients admitted to specialized cancer departments).

**Example:**

| source\_concept\_id | source\_concept\_code | source\_concept\_name         | source\_domain\_id | relationship\_id | target\_concept\_id | target\_concept\_code | target\_concept\_name               | target\_domain\_id |
| :------------------ | :-------------------- | :--------------------------- | :----------------- | :--------------- | :------------------ | :-------------------- | :--------------------------------- | :----------------- |
| 4147096             | 309948006             | Pediatric oncology department | Observation        | Maps to          | 4138940             | 305390000             | Admission to pediatric oncology department | Observation        |
| 4147086             | 309903007             | Radiotherapy department      | Observation        | Maps to          | 4202016             | 308252005             | Admission to radiotherapy department | Observation        |

### 5. Standardizing and Unifying Metastasis and Lymph Node Concepts

SNOMED contains multiple redundant and precoordinated concepts describing metastatic spread and lymph node involvement. These lead to ambiguity, inconsistent staging, and incorrect patient cohorting for oncology research.

**Desired Logic**
SNOMED metastasis-related and lymph node concepts must be uniformly mapped to standardized **Cancer Modifier** concepts, clearly specifying metastatic presence, location, or lymph node involvement. This standardization is essential for accurate patient cohorting and reliable staging analytics.

**Example:**

| SOURCE | | | | | | | TARGET | | | | | |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **concept\_id** | **concept\_code** | **concept\_name** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** | **relationship\_id** | **concept\_id** | **concept\_code** | **concept\_name** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** |
| 4163446 | 399409002 | Distant metastasis present | Condition | SNOMED | Clinical Finding | Maps to | 36769180 | OMOP4998856 | Metastasis | Measurement | Cancer Modifier | Metastasis |
| 4245697 | 396787000 | Tumor metastasis to non-regional lymph nodes present | Condition | SNOMED | Clinical Finding | Maps to | 36769243 | OMOP4998920 | Distant spread to lymph node | Measurement | Cancer Modifier | Nodes |
| 4265295 | 397440000 | Anatomic location of excised lymph node containing metastatic neoplasm | Observation | SNOMED | Observable Entity | Maps to | 36768587 | OMOP4998263 | Spread to lymph node | Measurement | Cancer Modifier | Nodes |
| 4336816 | 8707003 | Metastatic from | Observation | SNOMED | Attribute | Maps to | 36769180 | OMOP4998856 | Metastasis | Measurement | Cancer Modifier | Metastasis |
| 4129938 | 261731003 | CNS metastases | Measurement | SNOMED | Staging / Scales | Maps to | 35226096 | OMOP5031564 | Metastasis to central nervous system | Measurement | Cancer Modifier | Metastasis |
| 4299435 | 385421009 | Site of distant metastasis | Observation | SNOMED | Observable Entity | Maps to | 36769180 | OMOP4998856 | Metastasis | Measurement | Cancer Modifier | Metastasis |
| 4161021 | 399462009 | Secondary tumor site | Observation | SNOMED | Observable Entity | Maps to | 36769180 | OMOP4998856 | Metastasis | Measurement | Cancer Modifier | Metastasis |
| 4168514 | 417957003 | Uveal metastasis | Measurement | SNOMED | Staging / Scales | Maps to | 35225584 | OMOP5031993 | Metastasis to uveal tract | Measurement | Cancer Modifier | Metastasis |
| 4154265 | 371512006 | Presence of direct invasion by primary malignant neoplasm to lymphatic vessel and/or small blood vessel | Observation | SNOMED | Observable Entity | Maps to | 36768891 | OMOP4998568 | Lymphovascular Invasion (LVI) | Measurement | Cancer Modifier | Extension/Invasion |

### 6. Post-Coordination mappings with Morph Abnormality `source_concept_class`: Morph Abnormality → Condition + Cancer Modifier

Concepts combining tumor histology/topography with metastatic status should be split for clarity and modularity. These mappings involve post-coordination, where pre-coordinated SNOMED concepts are split into separate **Condition** and **Cancer Modifier** concepts.

**Mapping Logic**
If a condition concept precisely matching the source meaning (e.g., specifically metastatic adenocarcinoma rather than generic adenocarcinoma) is available, it should be selected as the target condition concept.

**Example 1 - Precise matching concept available:**

| source\_concept\_id | source\_concept\_name | source\_domain\_id | source\_concept\_class | source\_vocabulary\_id | relationship\_id | target\_concept\_id | target\_concept\_name | target\_domain\_id | target\_concept\_class | target\_vocabulary\_id |
| :------------------ | :-------------------- | :----------------- | :-------------------- | :--------------------- | :--------------- | :------------------ | :-------------------- | :----------------- | :-------------------- | :--------------------- |
| 4263459             | Adenocarcinoma, metastatic | Observation        | Morph Abnormality     | SNOMED                 | Maps to          | 604497              | Metastatic adenocarcinoma | Condition          | Disorder              | SNOMED                 |
| 4263459             | Adenocarcinoma, metastatic | Observation        | Morph Abnormality     | SNOMED                 | Maps to          | 36769180            | Metastasis            | Measurement        | Metastasis            | Cancer Modifier        |

If a specific concept describing the exact metastatic nature of the neoplasm is unavailable, the nearest clinically relevant and general concept (Malignant epithelial neoplasm) is chosen. The modifier Metastasis is separately mapped to maintain clinical precision and analytical granularity.

**Example 2 - Precise matching concept not available (nearest meaning used):**

| source\_concept\_id | source\_concept\_name | source\_domain\_id | source\_concept\_class | source\_vocabulary\_id | relationship\_id | target\_concept\_id | target\_concept\_name | target\_domain\_id | target\_concept\_class | target\_vocabulary\_id |
| :------------------ | :-------------------- | :----------------- | :-------------------- | :--------------------- | :--------------- | :------------------ | :-------------------- | :----------------- | :-------------------- | :--------------------- |
| 4194691             | Carcinoma, metastatic | Observation        | Morph Abnormality     | SNOMED                 | Maps to          | 36716620            | Malignant epithelial neoplasm | Condition          | Disorder              | SNOMED                 |
| 4194691             | Carcinoma, metastatic | Observation        | Morph Abnormality     | SNOMED                 | Maps to          | 36769180            | Metastasis            | Measurement        | Metastasis            | Cancer Modifier        |

**Other mappings with Post-coordination (Morph Abnormality → Condition + Cancer Modifier)**
Although currently mappings with Morph Abnormality are not considered for implementation, these mappings should be reviewed.

| SOURCE | | | | | | TARGET | | | | |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **concept\_id** | **concept\_name** | **domain\_id** | **concept\_class** | **vocabulary\_id** | **relationship\_id** | **concept\_id** | **concept\_name** | **domain\_id** | **concept\_class** | **vocabulary\_id** |
| 4183217 | Choriocarcinoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 4092517 | Choriocarcinoma | Condition | Disorder | SNOMED |
| 4183217 | Choriocarcinoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 4162254 | Kaposi’s sarcoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 434584 | Kaposi’s sarcoma (clinical) | Condition | Disorder | SNOMED |
| 4162254 | Kaposi’s sarcoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 4004515 | Malignant lymphoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 432571 | Malignant lymphoma | Condition | Disorder | SNOMED |
| 4004515 | Malignant lymphoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 4157460 | Malignant melanoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 40481908 | Metastatic malignant melanoma | Condition | Disorder | SNOMED |
| 4157460 | Malignant melanoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 46270968 | Metastatic hepatocellular carcinoma | Observation | Morph Abnormality | SNOMED | Maps to | 37162050 | Metastatic hepatocellular carcinoma | Condition | Disorder | SNOMED |
| 46270968 | Metastatic hepatocellular carcinoma | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 46273704 | Metastatic Mullerian mixed tumor | Observation | Morph Abnormality | SNOMED | Maps to | 42513618 | Neoplasm defined only by histology: Carcinosarcoma, NOS | Condition | ICDO Condition | ICDO3 |
| 46273704 | Metastatic Mullerian mixed tumor | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 4181016 | Metastatic signet ring cell carcinoma | Observation | Morph Abnormality | SNOMED | Maps to | 42513333 | Neoplasm defined only by histology: Signet ring cell carcinoma | Condition | ICDO Condition | ICDO3 |
| 4181016 | Metastatic signet ring cell carcinoma | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 46271141 | Metastatic small cell carcinoma | Observation | Morph Abnormality | SNOMED | Maps to | 37165127 | Metastatic small cell neuroendocrine carcinoma | Condition | Disorder | SNOMED |
| 46271141 | Metastatic small cell carcinoma | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 4032806 | Neoplasm, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 443392 | Malignant neoplastic disease | Condition | Disorder | SNOMED |
| 4032806 | Neoplasm, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 4163000 | Sarcoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 40480137 | Metastatic sarcoma | Condition | Disorder | SNOMED |
| 4163000 | Sarcoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 4271714 | Squamous cell carcinoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 4300118 | Squamous cell carcinoma | Condition | Disorder | SNOMED |
| 4271714 | Squamous cell carcinoma, metastatic | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |
| 46271138 | Metastatic leiomyosarcoma | Observation | Morph Abnormality | SNOMED | Maps to | 40482829 | Leiomyosarcoma | Condition | Disorder | SNOMED |
| 46271138 | Metastatic leiomyosarcoma | Observation | Morph Abnormality | SNOMED | Maps to | 36769180 | Metastasis | Measurement | Metastasis | Cancer Modifier |

---

## NAACCR Mapping Logic

### Handling of Duplicate or Contextual TNM Values

When mapping NAACCR Value concepts to **Cancer Modifier**, identical staging strings (e.g., cM0, pTis, cN0) can appear under different NAACCR Variables, such as TNM Clin M or TNM Path M. Despite field differences, these values often represent the same clinical or pathological assertion.

Mapping must prioritize clinical semantics over field origin, assigning a single OMOP standard concept per staging meaning. This ensures AJCC-compliant logic and avoids creating invalid or redundant Cancer Modifier entries.

**Example 1 – cM0(i+) used in both Clinical and Pathological M fields**

| SOURCE | | | | | | | TARGET | | | | | |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **concept\_id** | **concept\_code** | **concept\_name** | **NAACCR Variable** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** | **relationship\_id** | **concept\_id** | **concept\_code** | **concept\_name** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** |
| 35919673 | 960@c0 | cM0 | TNM Clin M | Meas Value | NAACCR | NAACCR Value | Maps to | 1635291 | c-AJCC/UICC-M0 | AJCC/UICC clinical M0 Category | Measurement | Cancer Modifier | Staging/Grading |
| 35919383 | 900@c0 | cM0 | TNM Path M | Meas Value | NAACCR | NAACCR Value | Maps to | 1635291 | c-AJCC/UICC-M0 | AJCC/UICC clinical M0 Category | Measurement | Cancer Modifier | Staging/Grading |

AJCC does not define a ‘pM0’ category. When distant metastases are excluded clinically, but no histological evidence exists, registrars may record ‘cM0’ even under the Path M field. Both must map to the clinical ‘cM0’ concept to avoid generating invalid pathological semantics.

**Example 2 – pTis in Clinical and Pathological T fields**

| SOURCE | | | | | | | TARGET | | | | | |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **concept\_id** | **concept\_code** | **concept\_name** | **NAACCR Variable** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** | **relationship\_id** | **concept\_id** | **concept\_code** | **concept\_name** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** |
| 35918423 | 880@p000 | pTis | TNM Path T | Meas Value | NAACCR | NAACCR Value | Maps to | 1634581 | p-AJCC/UICC-Tis | AJCC/UICC pathological Tis Category | Measurement | Cancer Modifier | Staging/Grading |
| 35918651 | 840@c000 | pTis | TNM Clin T | Meas Value | NAACCR | NAACCR Value | Maps to | 1634581 | p-AJCC/UICC-Tis | AJCC/UICC pathological Tis Category | Measurement | Cancer Modifier | Staging/Grading |

Tis (in situ) stage can be defined in a clinical context only with pathological confirmation and cannot exist as a clinical category. Even if recorded in the clinical T field, they must map to a pathological Tis concept to preserve semantic correctness and AJCC compliance.

**Example 3 – cN0m+ in both Clinical and Pathological N fields**

| SOURCE | | | | | | | TARGET | | | | | |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **concept\_id** | **concept\_code** | **concept\_name** | **NAACCR Variable** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** | **relationship\_id** | **concept\_id** | **concept\_code** | **concept\_name** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** |
| 35919012 | 950@c1 | cM1 | TNM Clin M | Meas Value | NAACCR | NAACCR Value | Maps to | 1635085 | c-AJCC/UICC-M1 | AJCC/UICC clinical M1 Category | Measurement | Cancer Modifier | Staging/Grading |
| 35919300 | 900@c1 | cM1 | TNM Path M | Meas Value | NAACCR | NAACCR Value | Maps to | 1635085 | c-AJCC/UICC-M1 | AJCC/UICC clinical M1 Category | Measurement | Cancer Modifier | Staging/Grading |
| 35919370 | 900@p1 | pM1 | TNM Path M | Meas Value | NAACCR | NAACCR Value | Maps to | 1635505 | p-AJCC/UICC-M1 | AJCC/UICC pathological M1 Category | Measurement | Cancer Modifier | Staging/Grading |
| 35919660 | 960@p1 | pM1 | TNM Clin M | Meas Value | NAACCR | NAACCR Value | Maps to | 1635505 | p-AJCC/UICC-M1 | AJCC/UICC pathological M1 Category | Measurement | Cancer Modifier | Staging/Grading |

Unlike M0, ‘pM1’ is a valid AJCC category for histologically confirmed metastases. Regardless of source field, ‘pM1’ should map to the pathological concept, and ‘cM1’ to the clinical one. Field origin must not override true clinical meaning.

**Example 4 – ‘cN0’ Used in TNM Clin N and TNM Path N**

| SOURCE | | | | | | | TARGET | | | | | |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **concept\_id** | **concept\_code** | **concept\_name** | **NAACCR Variable** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** | **relationship\_id** | **concept\_id** | **concept\_code** | **concept\_name** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** |
| 35919626 | 950@c0 | cN0 | TNM Clin N | Meas Value | NAACCR | NAACCR Value | Maps to | 1634145 | c-AJCC/UICC-N0 | AJCC/UICC clinical N0 Category | Measurement | Cancer Modifier | Staging/Grading |
| 35919481 | 890@c0 | cN0 | TNM Path N | Meas Value | NAACCR | NAACCR Value | Maps to | 1634145 | c-AJCC/UICC-N0 | AJCC/UICC clinical N0 Category | Measurement | Cancer Modifier | Staging/Grading |

Although this value appears under both TNM Clin N and TNM Path N, it semantically represents clinical staging. If no histological confirmation of lymph node status is available (e.g., no lymph node dissection), then even entries under TNM Path N using cN0 must be interpreted as clinical in nature. Mapping both entries to the AJCC/UICC clinical N0 Category avoids semantic errors like falsely assuming pathological confirmation of node status, which is critical for accurate cohort stratification and survival analysis.

**Example 5 – Mapping Logic for cN0m± / pN0m± (Molecular assessment of regional lymph nodes) NAACCR Values**

**Mapping Logic:**
NAACCR Value concepts with additional molecular (mol+/mol-) or histological (i+/i-) specifications describe the detection of isolated tumor cells (ITCs) in regional lymph nodes via molecular or histological methods, respectively. They are fundamentally distinct from distant metastases. Therefore, mapping these concepts to the general "Metastasis" cancer modifier concept is semantically incorrect.

| SOURCE | | | | | | | TARGET | | | |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **concept\_id** | **concept\_code** | **concept\_name** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** | **relationship\_id** | **concept\_id** | **concept\_code** | **concept\_name** |
| 35910008 | 950@c0M- | cN0m- | Meas Value | NAACCR | NAACCR Value | Maps to | 1634145 | c-AJCC/UICC -N0 | AJCC/UICC clinical N0 Category |
| 35919731 | 950@c0M+ | cN0m+ | Meas Value | NAACCR | NAACCR Value | Maps to | 1634145 | c-AJCC/UICC -N0 | AJCC/UICC clinical N0 Category |
| 35919273 | 890@p0M- | pN0m- | Meas Value | NAACCR | NAACCR Value | Maps to | 1635597 | p-AJCC/UICC -N0 | AJCC/UICC pathologic N0 Category |
| 35919608 | 890@p0M+ | pN0m+ | Meas Value | NAACCR | NAACCR Value | Maps to | 1633485 | p-AJCC/UICC -N0(mol+) | AJCC/UICC pathologic N0(mol+) Category |

**Rationale:**
* **m+/m- in NAACCR = mol+/mol- in AJCC.** They denote the presence or absence of isolated tumor cells detected only by molecular techniques (e.g., RT-PCR) in regional lymph nodes. This has no relation to distant metastasis.
* **Clinical stage.** AJCC 8 (2016) abolished the use of (mol±) suffixes in clinical staging; NAACCR subsequently dropped these codes. Therefore, both `cN0m+` and `cN0m-` are mapped to the general clinical N0 concept (1634145) to preserve valid analytics, even though `cN0m+` is still erroneously "Standard/Valid" in Athena.
* **Pathologic stage.** AJCC retains (mol+) for pathology. `pN0m+` is mapped to the specific `pN0(mol+)` concept (1633485) to keep the molecular detail. There is no `pN0(mol-)` concept in Cancer Modifier; `pN0m-` therefore maps to the broader `pN0` (1635597). Creating a `pN0(mol-)` concept would achieve one-to-one coverage but is not critical for cohort logic: both `pN0` and `pN0(mol-)` represent “no metastatic involvement” of nodes - the only difference is documentation of additional testing.

**Example 6 – Mapping Logic for cN0i± / pN0(i±) (Immunohistochemical assessment of regional lymph nodes) NAACCR Values**

| SOURCE | | | | | | | TARGET | | | |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **concept\_id** | **concept\_code** | **concept\_name** | **domain\_id** | **vocabulary\_id** | **concept\_class\_id** | **relationship\_id** | **concept\_id** | **concept\_code** | **concept\_name** |
| 35919501 | 950@c0I- | cN0i- | Meas Value | NAACCR | NAACCR Value | Maps to | 1634145 | c-AJCC/UICC -N0 | AJCC/UICC clinical N0 Category |
| 35919561 | 950@c0I+ | cN0i+ | Meas Value | NAACCR | NAACCR Value | Maps to | 1634145 | c-AJCC/UICC -N0 | AJCC/UICC clinical N0 Category |
| 35919313 | 890@p0I- | pN0(i-) | Meas Value | NAACCR | NAACCR Value | Maps to | 1635597 | p-AJCC/UICC -N0 | AJCC/UICC pathologic N0 Category |
| 35919328 | 890@p0I+ | pN0(i+) | Meas Value | NAACCR | NAACCR Value | Maps to | 1633503 | p-AJCC/UICC -N0(i+) | AJCC/UICC pathologic N0(i+) Category |

**Rationale:**
* **i+/i-** capture isolated tumor cells detected morphologically or by IHC. Logic parallels the molecular case: AJCC bans (i±) in clinical N after 2016, so both clinical codes map to generic cN0. For pathology (i+) is retained and mapped specifically; (i-) maps to generic pN0 due to missing Cancer Modifier concept.

**Comment on the “Metastasis” suggestion:**
Mapping `…m+` to OMOP 4998856 “Metastasis” is incorrect:
* `N0m+/N0mol+` refers to regional nodes, not distant disease.
* AJCC explicitly classifies these cases as N0 (regional, not M category).
* Inflating them to a distant-metastasis modifier would misclassify stage, distort cohort definitions, and bias survival analyses.

**Final mapping rules**

* **Clinical codes:**
    * All legacy `cN0m+/-` and `cN0i+/-` map to **c-AJCC/UICC N0 (1634145)**.

* **Pathologic codes:**
    * `pN0m+` → **p-AJCC/UICC N0(mol+) (1633485)**
    * `pN0i+` → **p-AJCC/UICC N0(i+) (1633503)**
    * `pN0m-`, `pN0(i-)` → **p-AJCC/UICC N0 (1635597)**.

* No secondary mapping to generic Cancer Modifier “Metastasis” for any `m+` or `i+` value.

**Future enhancement:** Consider adding `pN0(mol-)` and `pN0(i-)` to Cancer Modifier for complete one-to-one coverage, though the absence of these concepts has no material impact on stage grouping or patient selection in Atlas.
