---
name: make-ai-toolkit
description: Make AI Toolkit modules — make-ai-extractors (document/image/audio extraction) and make-ai-web-search — module IDs, use cases, and content pipeline patterns for use alongside AI agents.
---

# Make AI Toolkit

Make provides two built-in AI utility apps for content processing. These are commonly used to pre-process inputs before passing them to an AI agent, or as module tools attached directly to an agent.

## `make-ai-extractors` — Extraction Modules

| Module ID | Label | Description |
|---|---|---|
| `make-ai-extractors:extractADocument` | Extract text from a document | Extracts text from any document type (PDF, DOCX, etc.) |
| `make-ai-extractors:extractTextFromAnImage` | Extract text from an image | Extracts printed or handwritten text (OCR). For non-image formats use `extractADocument` |
| `make-ai-extractors:describeAnImage` | Describe an image | Returns a detailed description of image content |
| `make-ai-extractors:captionAnImage` | Generate a caption for an image | One-sentence caption of image content |
| `make-ai-extractors:captionAnImageAdvanced` | Generate captions for an image (advanced) | Up to 10 captions for different image regions — each is a separate bundle |
| `make-ai-extractors:detectObjectInAnImage` | Detect objects in an image | Returns detected objects and their approximate locations |
| `make-ai-extractors:getImageTags` | Generate image tags | Returns a list of descriptive words for the image |
| `make-ai-extractors:extractAnInvoice` | Extract information from an invoice | Structured extraction of invoice fields |
| `make-ai-extractors:extractAReceipt` | Extract information from a receipt | Structured extraction of receipt fields |
| `make-ai-extractors:transcribeAnAudio` | Transcribe an audio file | Speech-to-text transcription |
| `make-ai-extractors:translateAnAudio` | Translate an audio file | Translates audio to English text |

## `make-ai-web-search` — Web Search Module

| Module ID | Label | Description |
|---|---|---|
| `make-ai-web-search:generateAResponse` | Generate a response | Generates a response grounded in live web search. Can also fetch and process specific URLs |

## Content Pipeline Patterns

### Document → Agent

Pre-extract text before sending to an agent. Avoids file input token overhead for text documents:

```
[File source] → make-ai-extractors:extractADocument → ai-local-agent:RunLocalAIAgent
  agent mapper: message: "{{N.text}}"
```

### Image → Agent

```
[Image source] → make-ai-extractors:describeAnImage → ai-local-agent:RunLocalAIAgent
  agent mapper: message: "{{N.description}}"
```

### Binary files (PDF, image) as agent file input

For cases where the agent itself needs to reason over the raw file (not just extracted text), pass the buffer directly via the agent's `files` field:

```json
"files": [
  {
    "fileName": "{{N.fileName}}",
    "data": "{{N.data}}"
  }
]
```

Supported agent file input types: JPG, PNG, GIF, PDF only. For all other formats, use `make-ai-extractors:extractADocument` first.

### Extractor or web search as a Module Tool

Attach `make-ai-extractors` or `make-ai-web-search` modules directly as module tools on an AI agent when you want the agent to decide when to invoke them:

```
ai-local-agent:RunLocalAIAgent
  tools:
    - name: "Extract document text"
      description: "Extracts plain text from a document file. Use when given a PDF or DOCX to analyze."
      flow: [make-ai-extractors:extractADocument]
    - name: "Search the web"
      description: "Returns a grounded answer from live web search for current events or facts."
      flow: [make-ai-web-search:generateAResponse]
```

## Gotchas

- **`extractADocument` vs `extractTextFromAnImage`** — use `extractADocument` for PDFs and Office files; use `extractTextFromAnImage` only for pure image files with text (scanned pages, photos of text).
- **`captionAnImageAdvanced` produces multiple bundles** — each caption is a separate bundle processed by the rest of the scenario. If you want a single output, aggregate after this module.
- **`make-ai-web-search:generateAResponse`** can fetch specific URLs as well as run general web searches — useful when you need to retrieve and summarize a live webpage without a dedicated HTTP module.
- **App version** — always use version `1` for both apps when building blueprints.
