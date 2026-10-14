# 03 — Chat & Document Analysis

Flit provides a high-performance web chat experience designed for everyday enterprise productivity, multi-turn reasoning, and multimodal document analysis.

---

## Selecting a Model or Tier

At the top of the chat stage, the model picker offers three operational modes:

1. **`Flit — Automatic` (`auto`)** *(Default)*:
   Flit automatically evaluates your prompt using the active classification pipeline and directs it to the most cost-effective tier (`low`, `medium`, or `high`) capable of answering your request.
2. **Explicit Tiers (`low`, `medium`, `high`)**:
   Select an explicit tier when you know the required complexity in advance:
   - `low`: Quick summaries, greetings, grammar correction, translation.
   - `medium`: Content generation, business correspondence, structured research.
   - `high`: Deep code synthesis, architecture design, multi-stage logic proofs.
3. **Specific Deployments**:
   If enabled for your account or team, select pinned upstream deployments directly (e.g., `gpt-4o-prod`, `claude-3-5-sonnet`).

---

## Supported File Attachments

Attach documents to your conversation using the paperclip attachment icon or by dragging files directly into the prompt box:

| File Type | Extensions | Parsing Mechanism | Handling Details |
| :--- | :--- | :--- | :--- |
| **PDF Documents** | `.pdf` | Client-Side / WebAssembly | Text and structure are extracted locally in your browser before submission. Keeps file data private. |
| **Spreadsheets** | `.xlsx`, `.csv` | Tabular Markdown Parser | Sheets are converted into clean markdown tables with column headers so the model can inspect rows and calculate figures. |
| **Word Documents** | `.docx` | Rich Text Extractor | Formatted paragraphs, headings, and bullet points are parsed into readable context. |
| **Text & Code Files** | `.txt`, `.json`, `.md`, `.py`, `.ts`, `.sh`, `.yaml` | Raw UTF-8 Reader | Injected as fenced code blocks with language indicators. |
| **Images** | `.png`, `.jpg`, `.jpeg`, `.webp` | Base64 Vision Payload | Sent as multimodal vision messages to models with image understanding. |

> **File Constraints**: Maximum 10 MB per individual file. Attachments are parsed in memory and tied to the active conversation turn.

---

## Conversation Management

- **Chronological Grouping**: Left sidebar groups past threads into **Today**, **Yesterday**, **Previous 7 Days**, and **Older**.
- **Renaming & Deleting**: Automatically assigns a title from the first prompt turn; threads can be manually renamed or removed.
- **Client-Side Persistence**: Chat history is persisted in browser storage keyed to your authenticated session.
- **Keyboard Shortcuts**:
  - `Enter`: Submit prompt.
  - `Shift + Enter`: Insert new line without sending.
  - `Esc`: Dismiss active dropdowns, model selector, or file previews.

---

## Visual Theme & Typography

Controls at the bottom of the chat sidebar allow personal styling adjustments:
- **Theme Cycle**: System (OS default), Light, and Dark modes.
- **Font Size Adjustments**: Increment (`A+`) or decrement (`A−`) font size by up to 6 points from the baseline to accommodate display preferences.

---

## Next Steps

- For administrators: Configure model routing in [MODEL_ADMINISTRATION_AND_TIERS.md](MODEL_ADMINISTRATION_AND_TIERS.md).
- Learn how automatic routing decisions are made in [AUTOMATIC_REQUEST_CLASSIFICATION.md](AUTOMATIC_REQUEST_CLASSIFICATION.md).
