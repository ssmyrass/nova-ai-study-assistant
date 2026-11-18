# Engineering decisions

This document records the choices behind Nova and, more usefully, what each one cost.
It is written for someone reviewing the project who wants to know *why*, not *what*.

---

## 1. Why a local model instead of a cloud API

**Decision.** The model runs in Ollama on the user's own machine (`qwen2.5:3b` by default, `qwen2.5:7b` optional).

**Why.** Course notes are personal material, and a student assistant that ships notes to a third party is a harder thing to defend than one that does not. Running locally also removes the per-request cost, which matters for a product used daily by people who do not pay.

**What it cost.** A 3B model is markedly weaker than a hosted frontier model. Everything in section 3 exists because of this decision: the schema binding, the evidence check and the second-opinion pass are all compensation for a small model. It also puts a floor under the hardware: ~8 GB of RAM, and answers are 2-3× slower on CPU than on a GPU.

**What would change it.** A hosted model would make the answers better and the phone experience independent of a laptop. It would also mean an account system, a bill, and notes leaving the device. For this product the trade was worth taking; for a paid product aimed at exam candidates it would be worth revisiting.

---

## 2. Why n8n holds the business logic

**Decision.** Every endpoint is an n8n workflow: webhook → validate → authorise → work → respond. There is no hand-written backend service.

**Why.** The logic is visible. A reviewer can open the editor and watch a request walk through authentication, the model call, the validator and the database write. Error paths are branches you can see rather than exception handlers you have to read. Retries, fallbacks and the separation between "model failed" and "model timed out" are all wiring rather than code.

**What it cost.** Three things. Workflows drift when edited by hand, so they are produced by a generator script (kept private; available on request) and checked in CI — the editor is for reading, the generator is the source of truth. Node names become API: a reference like `$('Quiz: build the prompt')` breaks silently when a node is renamed, so every rename is verified against the exported JSON. And n8n's licence makes this a fine choice for a project of one's own and a poor one for a product you intend to sell; a sellable version would replace these flows with a small service of its own.

---

## 3. How a 3B model is made to produce reliable quizzes

This is the part of the project with the most engineering in it, because it is where a small model fails most visibly.

**Layer 1 — bind the shape.** Ollama's structured output (`format`) takes a JSON schema, so the response is always parseable. This removes an entire class of failure: no prompt pleading for "valid JSON only", no regex rescue.

**Layer 2 — validate the semantics.** A schema cannot say *the correct answer must be one of the options* or *no two questions may be the same*. Code checks those: four distinct options, `correct_answer` matching one of them character for character, no duplicate stems.

**Layer 3 — demand evidence.** Every question must carry an `evidence` sentence copied verbatim from the note. Code then checks that the sentence really occurs in the note. A question whose evidence is invented is dropped before the student ever sees it. This is also what makes the explanation on the result screen trustworthy: it is a real sentence from the student's own note.

**Layer 4 — ask again, blind.** The model answers its own questions a second time *without seeing the correct answers*. Where the two passes disagree, the question is dropped. Questions that are ambiguous or wrong tend not to survive this.

**Layer 5 — retry with the errors.** If validation fails, the model's own output and the list of errors are sent back with an instruction to fix them. One retry, then the request fails honestly with `QUIZ_INVALID` rather than returning something broken.

**The cost.** Two extra model calls per quiz and generation that takes longer. Two spare questions are generated per quiz so that the drop-outs do not leave the student short.

---

## 4. Context, memory and why the model is never reloaded

Three settings do most of the performance work:

- `OLLAMA_KEEP_ALIVE=24h` plus a warm-up request at startup, so the first question of a session does not pay for a model load.
- One `num_ctx` (6144) shared by every call. Ollama reloads the model when the context length changes, so a single value across all prompts is worth more than tuning each call.
- RAG instead of whole notes. Notes are chunked to ~700 characters and only the four passages closest to the question are sent. Beyond the speed, this is what keeps the model from wandering away from the note.

The vision model (`qwen2.5vl:3b`) is the exception: it loads only when a photo is uploaded and is released immediately afterwards, with the text model reloaded in the background so the summary that follows does not wait.

---

## 5. Why the correct answers never reach the client

The quiz endpoint returns questions and options only. `correct_answer` and `explanation` stay in MongoDB until `quiz/evaluate` runs.

This is not about cheating in a student's private study tool — it is about where the truth lives. Grading happens in one place, on the server, deterministically, and the client cannot disagree with it. The same instinct puts authorisation on every query: asking for someone else's note returns `404`, not `403`, so the existence of the record does not leak either.

---

## 6. Why fallbacks are rule-based rather than retried forever

When the model cannot write study advice or a weekly plan, code writes a reasonable version instead: the weak topics are known, the score is known, the chosen study days are known. The user gets something usable and a `source: 'rules'` marker records what happened.

The reasoning is that a demo, a classroom and an exam week all share one property — nobody cares *why* the screen is empty. A rule-based plan is worse than a model-written one and far better than an error.

---

## 7. Why there is no state-management package in the Flutter app

`ChangeNotifier` and `ListenableBuilder` from the framework, four dependencies in total (`http`, `shared_preferences`, `file_picker`, `google_fonts`).

The app has one user, one session and a handful of screens. A state container would have added vocabulary without removing a problem. The cost is honest: if the app grew another ten screens with shared, overlapping state, this choice would need to be made again.

---

## 8. Localisation as a call-site decision

Every user-visible string is written next to its translation where it is used:

```dart
Text(L.t('Upload a note', 'Ders notu yükle'))
```

No key files, no ARB, no codegen. A missing translation is visible in the diff rather than discovered at runtime, and a reviewer reading the code reads both languages at once. The cost is that this does not scale past two or three languages — at that point the strings belong in resource files and this layer should be replaced rather than extended.

---

## 9. One-click setup as a product feature

`start.bat` / `start.sh` detects Docker, generates secrets, picks a GPU compose file, builds the web bundle if needed, starts eight services in dependency order, imports and activates the workflows, pulls and warms the model, waits for the health check and prints the LAN address.

This exists because the project is handed to other people and run on machines nobody prepared. The same reasoning produced the offline USB package, the health-check wait loop instead of a fixed sleep, and `clean.bat`, which removes every trace of Nova and nothing else.

---

## 10. Things deliberately left undone

- **No tablet layout yet.** The screens are built for phone width; on a tablet they are correct but wasteful. The fix is a breakpoint and a two-column study screen, not a rewrite.
- **No push notifications, no account recovery.** Both need infrastructure that contradicts "runs on your own machine".
- **No multi-device sync beyond the LAN.** A phone reaches Nova over the local network; from elsewhere it needs a tunnel or a hosted deployment, which is a different product.
- **Screenshots in the documentation are from the Turkish build.** They are being recaptured from the English build.

---

<sub>© 2026 Sümeyra Kiğılı Şengüner · Phoenixa Analytics</sub>
