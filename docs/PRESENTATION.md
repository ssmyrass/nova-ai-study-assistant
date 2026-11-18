# Nova · Presentation guide

**File:** [`docs/Nova-Presentation.pdf`](Nova-Presentation.pdf) · 16 slides · 16:9
**Timing:** ~9 minutes of talking + ~8 minutes of live demo

The deck stands on its own; nothing else needs to be open while presenting.
There is little text on the slides — the talking is below, two or three sentences per slide.

---

**1 · Cover**
> "I'm Sümeyra. Nova turns a student's course note into a summary, a quiz and a study plan built around what they got wrong."

**2 · The problem**
> "A student has notes. What they do not have is a summary, questions to test themselves, or anything that tells them where the gaps are."
> "Everyone fills the gap between reading a note and actually studying it on their own. That gap is what Nova fills."

**3 · What Nova does**
> "A course note becomes studying in five steps: sign up, upload the note, take a quiz, read the result, revise."
> "These are the screens; we will go through all of them live in a moment."

**4 · What stands out**
> "Nova installs in one click, checks every question it writes against a sentence in the note, and shows the source of every answer it gives."
> "It reads a handwritten page from a photo, drills with memory cards, and produces a weekly plan and a weekly report."

**5 · Requirements**
> "All eight required items and all six bonus items are in place; the right-hand column is what I added on top."

**6 · One-click setup**
> "The usual route means installing Node, MongoDB, n8n and Ollama one by one and importing the flows by hand. With Nova you double-click `start.bat`: it opens Docker, generates the configuration, brings the services up in order, imports the flows and warms the model."

**7 · Why the setup matters**
> "Copy the folder and the same Nova runs on another machine; with no network it installs from a USB package; an NVIDIA card is picked up automatically; `clean.bat` removes every trace of it."

**8 · Architecture**
> "Flutter owns the screens, n8n the business logic, Ollama the model, MongoDB the data, and Docker Compose starts all of it with one command. Each part has exactly one job."

**9 · Flows 00 and 01**
> "I'll walk through all eight flows in order. The first is identity: signing up hashes the password one way and seeds three courses; signing in mints a session token."
> "The second is question answering: the notes are fetched, code picks the four passages closest to the question, and only those go to the model; the source sentence is shown under the answer."

**10 · Flows 02 and 03**
> "The summarisation flow turns the file into text, asks the model for a schema-bound summary, and stores the note in chunks for search."
> "In the quiz flow no question reaches the student unchecked: the evidence sentence is verified, the questions are answered again with the answers hidden, and whatever disagrees is dropped."

**11 · Flows 04 and 05**
> "In grading, the score is computed in code and the model writes advice for the topics that were missed; if the model cannot answer, a rule-based fallback takes over."
> "The dashboard flow prepares the numbers on the home screen, the per-course scores, the weekly plan and the weekly report."

**12 · Flows 06 and 07**
> "The health flow reports whether n8n, MongoDB, Ollama and the models are ready; the status in Settings comes from here."
> "Nova Memory generates cards from a note and verifies each one against it; on review, SM-2 computes when the card comes back."

**13 · Where the data lives**
> "This shows which collection every student action writes to. The password itself is never stored, only an irreversible hash, and every query is filtered by the user's id."

**14 · Security, speed and tests**
> "Security on the left, performance in the middle; on the right, 54 end-to-end tests that replay the whole path from sign-up to the weekly report, failure cases included."

**15 · Using it from a phone**
> "Nothing is installed on the phone: on the same Wi-Fi you type the laptop's address into the browser. Several students can be signed in at the same time, each with their own account."

**16 · Closing → live demo**
> "Now let's open a brand-new student account and do all of it live."
> **Demo order:** sign up → pick a course → upload a note → summary → quiz (easy/medium/hard) → result and advice → ask Nova (source sentence) → Nova Memory → plan and report.

---

### Checklist before presenting

1. Run `start.bat` about 20 minutes beforehand; upload a note and generate one quiz so the model is warm.
2. Check that `http://localhost:8080` opens.
3. Keep the sample notes to hand: `materials\` (PDF, text, photo).
4. Have the demo e-mail ready — open a fresh account for every presentation.
5. Backup: `docs\Nova-Demo.mp4` (a live recording).
