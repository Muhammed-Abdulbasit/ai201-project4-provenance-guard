# Provenance Guard

Provenance Guard is a backend attribution analysis system designed for creative sharing platforms. It provides a plug-in API that accepts submitted text content, analyzes it using multiple detection signals, returns an attribution classification with an uncertainty-aware confidence score, generates a transparency label for users, and supports creator appeals when a classification is disputed.

The goal of Provenance Guard is not to make absolute claims about authorship. Instead, it provides an explainable attribution estimate with transparency around uncertainty.

---

## Features

### Content Submission Endpoint

Provenance Guard exposes an API endpoint for submitting text-based content.

Endpoint:

```
POST /submit
```

Request:

```json
{
  "content": "A short story excerpt goes here"
}
```

Response:

```json
{
  "id": "content_001",
  "classification": "likely_ai",
  "confidence": 0.91,
  "label": "This content was likely generated with AI assistance. Confidence: High.",
  "signals": {
    "perplexity_score": 0.87,
    "stylometric_score": 0.84
  }
}
```

The response contains:

* Attribution classification
* Confidence score
* User-facing transparency label
* Individual detection signal outputs

---

# Detection Pipeline

Provenance Guard uses multiple signals rather than relying on a single indicator.

The two implemented signals are:

## 1. Language Predictability Signal

This signal measures how predictable the writing style is.

AI-generated text often has:

* consistent sentence structures
* lower variation in word choices
* highly predictable phrasing

The signal produces a value between:

```
0.0 - 1.0
```

Where:

* 0.0 = highly unpredictable human-like writing
* 1.0 = highly predictable AI-like writing

---

## 2. Stylometric Pattern Signal

This signal analyzes writing characteristics including:

* average sentence length
* punctuation patterns
* repeated phrasing
* vocabulary diversity

The signal produces:

```
0.0 - 1.0
```

Where:

* 0.0 = strongly human-like patterns
* 1.0 = strongly AI-like patterns

---

# Confidence Scoring

The system combines both signals into a single confidence score.

Formula:

```
confidence =
(language_predictability * 0.5)
+
(stylometric_score * 0.5)
```

The result is normalized between:

```
0.0 - 1.0
```

Confidence interpretation:

| Score       | Meaning               |
| ----------- | --------------------- |
| 0.85 - 1.0  | High confidence AI    |
| 0.40 - 0.85 | Uncertain             |
| 0.0 - 0.40  | High confidence human |

A score of:

```
0.95
```

represents strong agreement between signals.

A score of:

```
0.51
```

represents uncertainty because the signals do not strongly support either classification.

---

# Transparency Labels

The system provides three possible user-facing labels.

## High-confidence AI

Displayed text:

> This content was likely created with AI assistance. Our analysis found strong indicators of AI-generated writing.

---

## High-confidence Human

Displayed text:

> This content was likely written by a human. Our analysis found strong indicators of human writing patterns.

---

## Uncertain

Displayed text:

> We could not confidently determine whether this content was AI-generated or human-written. Additional review may be needed.

---

# Appeals Workflow

Creators can submit an appeal if they believe their content was incorrectly classified.

Endpoint:

```
POST /appeal
```

Request:

```json
{
  "content_id": "content_001",
  "creator_reason": "I wrote this piece myself and believe it was incorrectly classified."
}
```

When an appeal is submitted:

1. The creator explanation is stored.
2. The original classification decision is preserved.
3. The content status changes to:

```
under_review
```

4. The appeal is added to the audit log.

Automated reclassification is not performed. Appeals are intended for human review.

---

# Rate Limiting

The submission endpoint uses rate limiting:

```
20 requests per minute per IP address
```

Reasoning:

* Prevents abuse from automated clients
* Allows normal creative platform usage
* Keeps detection resources available
* Provides enough capacity for testing and demonstrations

---

# Audit Logging

Every decision creates a structured audit entry.

Example:

```json
{
  "content_id": "content_001",
  "classification": "likely_ai",
  "confidence": 0.91,
  "signals": {
    "language_predictability": 0.87,
    "stylometric_score": 0.84
  },
  "appeal_status": "none",
  "timestamp": "2026-07-01"
}
```

Example log entries:

```json
[
 {
  "content_id": "001",
  "classification": "likely_ai",
  "confidence": 0.91
 },
 {
  "content_id": "002",
  "classification": "likely_human",
  "confidence": 0.22
 },
 {
  "content_id": "003",
  "classification": "uncertain",
  "confidence": 0.56
 }
]
```

---

# API Endpoints

## Submit Content

```
POST /submit
```

Analyzes submitted content.

---

## Submit Appeal

```
POST /appeal
```

Creates a creator appeal.

---

## View Audit Log

```
GET /log
```

Returns attribution decisions and appeal history.

---

# Testing

The system should be tested with:

## Clearly AI-like examples

Expected:

```
classification: likely_ai
confidence: high
```

---

## Clearly human examples

Expected:

```
classification: likely_human
confidence: high
```

---

## Ambiguous examples

Expected:

```
classification: uncertain
confidence: medium
```

The goal is not perfect detection but meaningful confidence separation.

---

# Future Improvements

Possible improvements:

* Machine learning classifier trained on larger datasets
* Human reviewer dashboard
* More linguistic signals
* User feedback loops
* Platform-specific calibration
