# NextStep: An AI-Native CRM that Does the Work

> Pitch for *Build The Next Big Thing with AI* (Devmarketplace x NED University)
> Team: Amna Faisal and Ahmed Raza · NED University · 2 members

**One-line pitch:** An AI-powered CRM for small-business owners and salespeople that turns a quick note, typed or pasted in English, Urdu or Roman Urdu, into updated records, ranked leads and next steps, so no follow-up gets forgotten.

## 1. Problem and impact

**The problem.** Small businesses run sales from phone calls, chats and memory. A CRM only helps if someone types everything in, and in a small team nobody has the time. Notes stay in chats, follow-ups are forgotten, and the owner cannot tell which leads deserve attention today. As a result, most small businesses have no usable customer record at all.

**The user.** The owner or salesperson of a small service or retail business (a tutoring centre, a solar installer, a clothing seller, a clinic) who handles customers from a phone between other work and has no admin staff.

**Why it matters.** A forgotten follow-up is a lost sale, and these businesses cannot afford to lose leads. Existing CRMs make the user do the data entry. Spreadsheets and chat apps give no guidance on who to contact next. NextStep makes the AI do the entry and the prioritising, so the record is kept without extra effort. *(Time saved is our hypothesis; we will measure it during the build.)*

## 2. The product

The user says or pastes what happened. The AI agent updates the records, scores the leads, and tells the user who to contact next. It always shows a preview before anything is saved.

> *"Ahmed sahab ne kaha budget 2 lakh hai, Proposal bhejna hai, kal 11 baje follow-up."*
> → Preview: *Update Ahmed Khan: budget 200,000, stage "Proposal". Create task "Send proposal / follow up", tomorrow 11:00. Lead score 82 (budget confirmed, active this week).* → Confirm → Saved and logged.

The agent understands English, Urdu and Roman Urdu, so it fits the way people here actually write and talk.

## 3. Core user journey (MVP)

1. Sign up, then set up 2FA with an authenticator app.
2. Add a contact, or paste a note, a chat message or call notes.
3. The AI agent calls a hosted language model and returns structured changes: fields to update, tasks to create, a lead score and a suggested next step.
4. The app shows a preview. The user confirms or edits.
5. The changes are saved to the database and written to an action log.
6. The next day the user opens **Today**: leads ranked by the AI, each with a reason and a suggested action.
7. The user signs out and back in. Everything is still there, and a second account sees none of it.

## 4. AI features

Each feature makes a **live call to a hosted language model** (Gemini or OpenAI, chosen on Day 1 after testing). The model proposes; the user confirms.

| AI feature | What it does | Model input → output |
|---|---|---|
| **A1. Command agent** | Understands a plain-language instruction and turns it into CRM actions, in English, Urdu or Roman Urdu | Sentence + the user's matching records → JSON actions (update, create task, add note) |
| **A2. Notes to records** | Reads pasted meeting notes, chats or call transcripts and extracts budget, needs, objections, dates and next steps | Free text → fields, tasks and a short summary |
| **A3. Lead scoring and next best action** | Scores each lead from 0 to 100 using its history (recency, budget, engagement, stage), explains why, and suggests the next step | Contact history → score, reason, suggested action |
| **A4. Follow-up drafting** | Writes a follow-up message in the right language and tone, using the record's context | Record + instruction → draft text (never sent automatically) |
| **A5. Ask your CRM** | Answers questions about the user's own data, such as "who haven't I contacted this week?" | Question + the user's records → answer citing the records used |

**Guardrails.** The model can only propose. Output is validated against a fixed schema, and nothing is written without confirmation. Each call and its result is stored in the action log. Answers use only the signed-in user's own rows. No responses are hardcoded.

## 5. Feature list (MVP)

| # | Feature | Done when |
|---|---|---|
| F1 | Sign up, sign in, TOTP 2FA (managed auth provider) | Two accounts see only their own data |
| F2 | Contacts, deals with pipeline stages, tasks, notes | Data survives restart and sign-out |
| F3 | A1 Command agent (typed) | 10 scripted commands work end to end |
| F4 | A2 Notes to records | 5 sample notes produce correct fields and tasks |
| F5 | A3 Lead scoring and the **Today** view | Leads appear ranked, each with a reason and a next step |
| F6 | Preview, confirm, action log, undo | Every change is previewed and reversible |
| F7 | A4 Follow-up drafting | A draft appears in the chosen language, for approval |
| F8 | A5 Ask your CRM | 10 test questions answered correctly from the data |

**Stretch (only after F1 to F8 pass):** voice input through the browser's speech recognition, with editable text.

**Not in the MVP (roadmap):** phone-call agent, Facebook Lead Ads and Messenger, WhatsApp, email sending, industry templates, payments, team dashboards.

## 6. Data model

All tables carry `user_id`, and every query is filtered by the signed-in user.

- **users:** id, email, created_at (auth and 2FA handled by the provider)
- **contacts:** id, user_id, name, phone, email, source, score, score_reason, next_action, last_contacted_at
- **deals:** id, user_id, contact_id, title, stage (New, Contacted, Proposal, Won, Lost), amount, updated_at
- **tasks:** id, user_id, contact_id, deal_id, title, due_at, done
- **notes:** id, user_id, contact_id, body, summary, created_at
- **agent_actions:** id, user_id, feature (A1 to A5), input_text, model_output (JSON), status (proposed, confirmed, rejected, undone), applied_changes (JSON), created_at

## 7. Technical approach

- **App:** responsive web app built in our Devmarketplace workspace, deployed to a public URL, with source in the public GitHub repo.
- **Database:** hosted relational database (final choice confirmed at kickoff, based on what the workspace offers).
- **Auth:** managed provider with email sign-in and TOTP 2FA.
- **AI:** server-side calls to a hosted LLM so API keys never reach the browser. Prompts ask for JSON, which is validated against a schema before use. Only the relevant records are sent to the model, keeping prompts short and costs low.
- **Evaluation:** a small test set of commands, notes and questions in English, Urdu and Roman Urdu. Results are published in the README.

## 8. Code gigs (published during the build)

1. **`crm-command-parser`:** a natural-language sentence (English, Urdu or Roman Urdu) in, a structured CRM action out (type, target, fields, resolved date and time), with a test set.
2. **`lead-scorer`:** takes a contact's history and returns a score, a reason and a suggested next action, with tests.

Gig 1 will be published on 20 Oct and gig 2 on 21 Oct. Both are used in the app's core journey.

## 9. Reuse plan and declarations

- **Reuse:** managed auth provider, a standard UI component library, a hosted database, and open-source libraries for date parsing. We will check Devmarketplace for existing gigs (auth, UI, date handling) before writing our own.
- **Original work:** the AI features, prompts and action schema, the two gigs, the data model and the CRM screens.
- **Pre-existing work:** none. The repo contains documentation only until the build starts on 19 Oct.
- **Disclosure:** all AI tools and open-source components will be listed with their licences in `AI_DISCLOSURE.md`.

## 10. Risks

| Risk | Plan |
|---|---|
| Model output is wrong or inconsistent | Schema validation; preview and confirm before every write; a test set with published accuracy |
| Urdu and Roman Urdu understanding is weaker than English | Test on Day 1; the user can edit the text in the preview |
| Lead scores feel arbitrary | Always show the reason; use a simple, transparent set of signals |
| Only 2 people and 4 days | Core F1 to F8 only; stretch work starts after their "done when" tests pass |

## 11. Plan (19–22 Oct, daily commits)

- **19 Oct:** workspace, database tables, auth with 2FA. Test models; fix prompts and the action schema.
- **20 Oct:** contact, deal and task screens; A1 command agent with preview; publish gig 1.
- **21 Oct:** A2 notes to records; A3 scoring and the Today view; action log and undo; publish gig 2.
- **22 Oct:** A4 drafting, A5 ask your CRM, deployment, two-account test, accuracy test, demo video, README and AI_DISCLOSURE.

## 12. Team readiness

- **Amna Faisal:** [role and skills, e.g. full-stack web, databases, UI]. Owns auth, database, screens and deployment.
- **Ahmed Raza:** [role and skills, e.g. backend, AI integration, testing]. Owns the AI features, prompts, both gigs and accuracy testing.
- Both of us are NED University students (BS Computer Science, third year). We have registered on Devpost and will have Devmarketplace accounts before kickoff.
