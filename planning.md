# Provenance Guard Planning Document

## Detection Signals

Provenance Guard uses two independent detection signals.

## Signal 1: Language Predictability Score

Purpose:

Measure how predictable the writing is based on word choice and sentence structure.

AI-generated text often has lower randomness and more consistent phrasing.

Output:

A score between:

```
0 - 1
```

Where:

* 0 = human-like unpredictability
* 1 = highly predictable AI-like writing

---

## Signal 2: Stylometric Pattern Score

Purpose:

Measure writing style characteristics.

Features analyzed:

* sentence length consistency
* punctuation usage
* vocabulary diversity
* repeated phrases

Output:

A score between:

```
0 - 1
```

Where:

* 0 = human-like writing patterns
* 1 = AI-like writing patterns

---

## Combining Signals

The final confidence score is calculated:

```
confidence =
(signal_1 * 0.5)
+
(signal_2 * 0.5)
```

The score represents confidence that the content was AI-generated.

Classification thresholds:

```
0.85 - 1.0  -> likely AI

0.40 - 0.85 -> uncertain

0.0 - 0.40  -> likely human
```

---

# Uncertainty Representation

A confidence score represents how strongly the system believes the classification.

Example:

```
0.95
```

means both signals strongly agree the content is AI-like.

Example:

```
0.60
```

means the signals show mixed evidence and the system is uncertain.

The system intentionally separates uncertain cases rather than forcing every result into AI or human categories.

---

# Transparency Label Design

## High-confidence AI

Displayed:

"This content was likely created with AI assistance. Our analysis found strong indicators of AI-generated writing."

---

## High-confidence Human

Displayed:

"This content was likely written by a human. Our analysis found strong indicators of human writing patterns."

---

## Uncertain

Displayed:

"We could not confidently determine whether this content was AI-generated or human-written. Additional review may be needed."

---

# Appeals Workflow

Who can submit an appeal:

Creators whose content has received an attribution classification.

Information required:

* Content ID
* Creator explanation/reason for dispute

When an appeal is submitted:

1. Store creator reasoning.
2. Link appeal to original attribution decision.
3. Change content status:

```
under_review
```

4. Add appeal event to audit log.

Human reviewers would see:

* Original content
* Original classification
* Confidence score
* Detection signals
* Creator explanation

---

# Anticipated Edge Cases

## Case 1: Short poetry with simple structure

A poem using repetition and simple vocabulary may appear AI-generated because the writing patterns are predictable.

Example:

A poem with repeated phrases and similar sentence lengths.

---

## Case 2: Human writers editing with AI tools

A creator may write most of a piece themselves but use AI for grammar improvements. The system may incorrectly classify it as fully AI-generated.

---

# Architecture

```
                 +----------------+
                 |  User Platform |
                 +-------+--------+
                         |
                         |
                  POST /submit
                         |
                         v
              +--------------------+
              | Provenance Guard API |
              +--------------------+
                         |
          +--------------+--------------+
          |                             |
          v                             v
+--------------------+       +--------------------+
| Detection Signal 1 |       | Detection Signal 2 |
| Predictability     |       | Stylometry         |
+--------------------+       +--------------------+
          |                             |
          +--------------+--------------+
                         |
                         v
              +----------------+
              | Confidence     |
              | Calculation    |
              +----------------+
                         |
                         v
              +----------------+
              | Transparency   |
              | Label Generator |
              +----------------+

                        

Creator
  |
  v
POST /appeal
  |
  v
Appeal Storage
  |
  v
Status = under_review
  |
  v
Audit Log
```

Submission flow:

A platform sends content to Provenance Guard. The system runs multiple detection signals, combines them into a confidence score, generates a transparency label, and stores the decision.

Appeal flow:

Creators submit disputes through the appeal endpoint. The system stores their reasoning, updates the content status, and records the event in the audit log.

---

# AI Tool Plan

## M3: Submission Endpoint + First Signal

Sections provided:

* Detection signals
* Architecture diagram

AI request:

Generate:

* Flask application skeleton
* POST /submit endpoint
* First detection signal function

Verification:

Test the detection function directly using several examples before connecting it to the API.

---

## M4: Second Signal + Confidence Scoring

Sections provided:

* Detection signals
* Uncertainty representation
* Architecture diagram

AI request:

Generate:

* Second signal function
* Confidence scoring logic
* Classification thresholds

Verification:

Check that scores vary meaningfully between:

* Clearly AI examples
* Clearly human examples
* Ambiguous examples

---

## M5: Production Layer

Sections provided:

* Transparency labels
* Appeals workflow
* Architecture diagram

AI request:

Generate:

* Label generation logic
* POST /appeal endpoint
* Audit logging support

Verification:

Test:

* All three labels are reachable
* Appeals update status correctly
* Logs contain attribution decisions and appeals
