# LLM 14: Multi-Modal Basics

Text-only LLMs are the past. Every major model shipped in 2025–2026 is multi-modal: text, images, audio, sometimes video. This session covers the architectural ideas and the patterns for building with them.

## 1. What "Multi-Modal" Actually Means

A model is multi-modal if it accepts or produces multiple input/output modalities beyond text. Common combinations:

| Capability | Example models |
| --- | --- |
| Vision-in, text-out | Claude, GPT-4o, Gemini, Llama-3.3 Vision |
| Text-in, image-out | DALL·E, Imagen, Flux, Stable Diffusion |
| Audio-in, text-out | Whisper, Canary, Gemini |
| Text-in, audio-out | ElevenLabs, OpenAI TTS |
| Audio-in, audio-out (voice) | GPT-4o Realtime, Gemini Live |
| Video-in, text-out | Gemini 1.5, Gemini 2 |
| Any-to-any | GPT-4o, Gemini (partial) |

Most products in 2026 use a combination: Whisper for ingest, a vision-language model for reasoning, TTS for output, for example.

## 2. How Vision-Language Models Work

The canonical architecture:

```
image → vision encoder → image tokens → concat with text tokens → decoder-only LLM → text
```

- **Vision encoder** — CNN or (more often) ViT (Vision Transformer). Patches an image into tiles, produces an embedding per patch.
- **Projection layer** — maps visual embeddings into the LLM's input embedding space.
- **Shared attention** — the decoder attends over both text and image tokens.

Training: interleaved image-text data (captions, documents, screenshots, diagrams), same next-token prediction objective.

Variants:

- **Flamingo / LLaVA** — frozen vision encoder + trainable projection + LLM.
- **Fuyu** — no vision encoder; raw image patches go directly into the LLM.
- **Qwen-VL, InternVL, Molmo** — strong open-weight vision-language models.

## 3. Image Tokenization and Cost

Images are expensive. A single 1024×1024 image becomes roughly 1K–2K tokens in most APIs. A PDF page can consume 2K–5K tokens.

Cost implications:
- OCR + text extraction is usually cheaper than sending raw images if the information is textual.
- Low-resolution vision (fauxHD downsample) works for UIs and simple layouts.
- High-resolution matters for charts, fine print, handwriting.

Provider-level knobs:
- **Claude:** resolution modes (no direct knob; accepts images up to ~8K tokens each).
- **OpenAI:** `detail=low|high|auto` on image inputs.
- **Gemini:** tokens-per-image scales with resolution.

## 4. Patterns for Vision Use

### 4a. Extraction

"Given this invoice image, return `{vendor, total, line_items[]}`." Pair with structured output (session 08). Vision-language models are remarkably good at this.

### 4b. Layout understanding

Charts, dashboards, web pages. Ask for elements, spatial relationships, extracted text. Better than pure OCR for complex layouts.

### 4c. Visual Q&A

"Is there a person in this image?" "What's the mood of this photo?" Simple, reliable.

### 4d. Screenshot-driven agents

Browser agents, desktop automation, UI testing. Model sees a screenshot, decides what to click. Part of `10-agentic-ai/`.

### 4e. Diagram + code

"This diagram describes a system. Generate the equivalent architecture as a Mermaid graph." High-utility, surprisingly reliable.

## 5. Audio: Whisper and Friends

**Whisper** (OpenAI, 2022, open-weight): encoder-decoder speech model, state-of-the-art for many years. Replacements (2025–2026): Distil-Whisper for speed, NVIDIA Canary, Gemini audio.

Typical pipeline:

```
mic → VAD (voice activity detection) → audio chunks → ASR → text → LLM → TTS → speaker
```

Key architectural decisions:

- **Streaming vs batch ASR** — real-time conversations need streaming; dictation can batch.
- **Language detection** — most models need it, some auto-detect.
- **Speaker diarization** — who is talking? Separate model (pyannote, WhisperX).
- **Word-level timestamps** — useful for editing / subtitles.

## 6. Realtime Voice Agents

GPT-4o Realtime API and Gemini Live both expose **any-to-any** audio streaming: you send audio; it emits audio. The model handles ASR, reasoning, and TTS internally.

Implications:

- **Turn-taking** is non-trivial. When is the user done speaking?
- **Interruption handling** — "wait, stop" must cut off the model mid-speech.
- **Voice personas** — model picks voice; you configure timbre.
- **Function calling + audio** — agent can use tools while conversing.

For 2026 voice products, realtime APIs are the bar. Traditional ASR→LLM→TTS pipelines have 1–3s latency that feels laggy.

## 7. Text-to-Image

Covered lightly here — it's a separate field (diffusion, flow matching, score-based).

From an architect's perspective:
- **Providers** — OpenAI (DALL·E/gpt-image), Black Forest Labs (Flux), Stability, Google Imagen, Adobe Firefly, Ideogram.
- **Cost** — $0.01–0.05 per image typically.
- **Control** — prompts, seeds, reference images, ControlNet-style conditioning.
- **Quality caveat** — text-in-image is often poor; charts should be real code, not generated images.

## 8. Video

Gemini 1.5 introduced long-context video understanding (hours of video in context). Useful for:
- Meeting transcription + Q&A
- Sports analysis
- Security surveillance review
- Educational content indexing

Video tokens are expensive. 10 minutes of video at low FPS ≈ 100K tokens. Plan accordingly.

## 9. Multi-Modal RAG

Retrieval over image+text corpora:

- **Separate embeddings per modality** — index images and text separately, retrieve each.
- **Joint embeddings** — CLIP-style models embed both into a shared space.
- **Multi-modal reranking** — retrieve broadly; rerank with a VL model.

Covered in `09-rag/13_multi_modal_rag`.

## 10. When NOT to Use Multi-Modal

- The information is natively textual (logs, code, databases). Stay text-only.
- Strict cost or latency budgets and the text extract works.
- Regulatory constraints on image data (faces, PHI).
- Any case where OCR + a text LLM is good enough. Often it is.

## 11. Architect Heuristics

- **Extract to text when you can.** Cheaper, more debuggable.
- **Use vision for layout-sensitive content** (invoices, charts, screenshots).
- **Budget image tokens** as you budget text tokens.
- **Realtime voice is a different architecture.** Plan for it separately.
- **Keep multi-modal pipelines observable** — each modality transition is a failure surface.

## Prereqs / Connections

- `02_transformer_architecture` — vision encoders are transformers over patches
- `03_tokenization` — images become tokens
- `08_structured_outputs` — extracting typed fields from images
- `09-rag/13_multi_modal_rag`
- `10-agentic-ai/` — screenshot-driven agents

## Further Reading

- Alayrac et al. 2022, "Flamingo: a Visual Language Model for Few-Shot Learning"
- Liu et al. 2023, "Visual Instruction Tuning" (LLaVA)
- Radford et al. 2022, "Robust Speech Recognition via Large-Scale Weak Supervision" (Whisper)
- Radford et al. 2021, "Learning Transferable Visual Models from Natural Language Supervision" (CLIP)
- OpenAI Realtime API docs
- Google "Gemini 1.5: Long-Context Multimodal" technical report
