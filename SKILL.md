---
name: apply
description: Generates ATS-optimized resume and cover letter in LaTeX from a job description URL or pasted text.
argument-hint: <job description text or URL>
allowed-tools: WebFetch, Read, Write, Bash, Edit, AskUserQuestion
---

# ATS-Optimized Resume & Cover Letter Generator

Generate a tailored LaTeX resume and cover letter from a job description, optimized for ATS screening while sounding authentically human.

> **Setup before first use:** Fill in `assets/profile.md` with your own background (work history, skills, education, awards). The shipped `profile.md` is an empty template with section scaffolding only. Optionally edit `assets/resume-template.tex` to replace the placeholder contact block with your name and links. `assets/lessons.md` starts empty and grows as you give feedback (Phase 8).

---

## Phase 1: Input Acquisition

Determine how to obtain the job description:

- If `$ARGUMENTS` is empty or missing, ask the user to provide a job description (URL or pasted text).
- If `$ARGUMENTS` starts with `http://` or `https://`, use WebFetch to retrieve the page content. If WebFetch fails (login wall, paywall, 403, etc.), inform the user and ask them to paste the JD text directly.
- Otherwise, treat `$ARGUMENTS` as the raw job description text.

---

## Phase 2: JD Analysis

Extract the following from the job description:

- **Company name**
- **Role title**
- **Required skills** (must-haves)
- **Preferred skills** (nice-to-haves)
- **Key responsibilities**
- **Culture keywords** (values, team style, mission language)
- **Seniority level** (inferred from title, years of experience, scope)

Present a short summary to the user and ask them to confirm or correct before proceeding:

> **Company:** X | **Role:** Y | **Level:** Z
> **Must-have skills:** ...
> **Nice-to-have skills:** ...
> **Key themes:** ...
>
> Does this look right? Any adjustments?

**Spotting hidden optional skills:** Some JDs blur the line between required and preferred. When a sentence like "Experience or interest within the above-mentioned areas is a good foundation" follows a skill list, those skills are effectively **optional**, not hard requirements. Read the prose, not just the "Required / Preferred" headings, and classify accordingly in your summary.

---

## Phase 3: Profile & Template Loading

Read the following files from `${CLAUDE_SKILL_DIR}/assets/`:

1. **`profile.md`** — Personal information and experience inventory. If this file is missing or mostly empty, STOP and tell the user:
   > Your profile is not set up yet. Please fill in `assets/profile.md` with your information first.
2. **`resume-template.tex`** — LaTeX resume template.
3. **`cover-letter-template.tex`** — LaTeX cover letter template.
4. **`lessons.md`** — Past feedback and learned preferences. If missing or empty, skip silently — this is expected for first-time use.

When `lessons.md` has entries, treat every rule in it as a hard constraint during generation.

**Note on multilingual profile content:** `profile.md` may be written in a language other than English (or a mix). Read and fully understand all of it — do not skip or skim sections because they are not in English. When generating English output, translate accurately and preserve all details, including metrics, project names, technical specifics, and competition results.

---

## Phase 3.5: Pre-Generation Preview (MANDATORY)

Before writing any file, present the planned approach to the user and wait for explicit confirmation.

The preview must cover:

**Resume strategy:**
- Positioning angle of the Summary (which identity/background to lead with)
- Which experience is the core (most relevant), which to down-weight or omit
- How the Skills section is grouped and which skills are emphasized
- Whether Certifications are kept and how they're presented

**Cover letter strategy:**
- Opening positioning (identity + career-arc sentence if the path is non-linear + strongest match + why this company)
- The 2–3 core JD requirements the body covers, the experience mapped to each, and its impact (what you did → what changed)
- Personal-connection detail (if any, capped at 1–2 sentences, noting which paragraph it goes into)
- Closing handling (language ability, availability, etc.)

Present this as a concise bullet-point outline (in the user's working language), then ask:

> Does this approach look right? Once you confirm, I'll start generating the files.

**List every work-experience entry from `profile.md` and state how each is handled (kept / down-weighted / omitted) with a reason.** Do not silently drop a role, even a low-relevance one — name it and explain, and let the user decide whether to include it.

**Do not write any `.tex` file until the user confirms.**

---

## Phase 4: Resume Generation

### Content Strategy

- Review ALL experience entries in `profile.md`. Select and prioritize bullets that best match the JD's required and preferred skills.
- For each selected experience entry, rewrite bullets to naturally incorporate JD keywords — do NOT keyword-stuff.
- Quantify achievements wherever possible: numbers, percentages, scale, team size, dollar amounts.
- Include 4-6 bullets for the most relevant role, 2-4 for others. Omit irrelevant roles entirely if the resume is too long.
- **Years of experience in Summary:** Distinguish total industry experience from domain-specific experience. Do not conflate the two — e.g., write "8 years of software industry experience, including 3 years in backend development" rather than "8 years of backend experience" if the profile does not support the latter.
- **Certifications/Awards must use flat single-level bullet lists.** Never use nested `\cvitemstart`/`\cvitemend` inside an award entry. Combine the award name and its contextual note into a single bullet using a comma (e.g., `Example Cert: 390/500 (top 1\%), \textit{national-level timed exam.}`). Do NOT use `---` as a separator.
- Skills section should lead with JD-matched skills, then list remaining relevant ones.
- **Bullet traceability (all sections):** Every factual claim in Experience and Projects bullets must trace to a specific entry in profile.md. This includes tools, techniques, outcomes, and scope. If profile.md describes a task as investigative or in-progress (e.g., a thesis), write only what is explicitly stated — do not infer concrete outcomes or add technical specifics that are not mentioned. When in doubt, omit the detail rather than infer it.

### ATS Compliance Rules

- Single-column layout only
- Standard section headers exactly as: **Experience**, **Education**, **Skills**, **Projects**
- No tables, images, graphics, headers/footers, or text boxes
- No columns created with tab stops or multiple spaces
- Use standard bullet characters
- Dates in consistent format: `Mon YYYY -- Mon YYYY` or `Mon YYYY -- Present`

### Anti-AI Writing Rules

**Banned words/phrases** — never use these:
- leveraged, spearheaded, synergy, cutting-edge, passion for, utilizing, facilitated, orchestrated, streamlined, holistic, paradigm, ecosystem (unless referring to an actual software ecosystem), drive/driving (as buzzword), stakeholders (prefer "users", "team", "clients"), robust, scalable (unless discussing actual system scale)
- **Em dashes** (`—`, Unicode em dash, or LaTeX `---`) anywhere in body text — they are a strong AI writing signal. Rewrite using a comma, colon, parentheses, or split into two sentences. En dashes (`--`) in date ranges are fine.

**Required style:**
- Use concrete verbs: built, wrote, shipped, reduced, migrated, designed, tested, debugged, automated, deployed, refactored, integrated
- Vary sentence length — mix short punchy bullets with longer detailed ones
- Every bullet must contain at least one specific detail (tool name, metric, team size, technology, or concrete outcome)
- No two consecutive bullets should start with the same word
- Write in past tense for past roles, present tense for current role

### Skills Validation (mandatory before writing the file)

After drafting resume content, verify every skill listed in the Skills section against `profile.md` only:

- **Only include a skill if it is explicitly named in profile.md.** The resume template's skills section is a formatting placeholder — it is not a source of truth for what skills to include.
- For each skill listed, find the exact word or phrase in profile.md. If you cannot point to a specific line, remove the skill.
- **When in doubt, exclude.** If you're thinking "this seems plausible given their work" or "they probably used this", that is not a basis for inclusion. The user will add missing skills themselves; a fabricated skill on a resume is far more harmful than a missing one.
- Do not infer skills from job duties. Working on DNN inference does not mean PyTorch. Working on embedded targets does not mean a specific framework. Only include what the profile explicitly states.
- **Interview-defensibility test:** Don't list a skill just because the word appears literally in profile.md. Course names, project context, brief exposure, and tech merely present in a collaboration environment should not automatically become standalone skills. Before listing a skill, ask "would the user be comfortable being grilled on this for 10 minutes and answering independently?" If unclear, default to leaving it out (it can still live in a specific Experience/Project bullet with weaker, accurate wording).
- This applies to all categories: languages, frameworks, tools, protocols, and platforms.
- **Skills are scannable keyword terms, not sentences.** Each entry is a recognizable skill noun (e.g., `Prompt Engineering`, `Parallel Programming`, `Distributed Systems`), not a descriptive phrase. Don't let a category label over-claim breadth beyond the terms actually listed on that line (e.g., don't label a line "Full-Stack" if it only lists backend tools). De-noise per target role: drop terms irrelevant to the current JD (they stay in the relevant Experience bullet, just not in the Skills scan band).
- **Before writing the file**, produce an internal checklist: for each skill you plan to include, write `[skill] → profile.md line: "[exact quote]"`. Any skill you cannot fill in must be removed.

### Draft Checkpoint

At this point the resume content exists as a confirmed draft **in context only**. Do NOT write any `.tex` file yet — file writing happens in Phase 7, after HR approval.

---

## Phase 5: Cover Letter Generation

### Core Principle: Delivery Document, Not Essay

The cover letter is a **delivery document**, not a personal essay. It has two jobs in sequence:
1. **HR screening (first 30 seconds):** "Does this person meet our requirements?" — HR must be able to answer YES without re-reading.
2. **Hiring manager decision:** "Would this person be an asset to my team?" — the manager needs to see concrete past impact that predicts future value.

Everything in the letter must serve one of these two readers. Content that serves neither gets cut.

### Content Strategy

- **Length:** 250–350 words, 3–4 paragraphs. Shorter is better. Every sentence must earn its place.
- **Opening (1 paragraph, 2–3 sentences):** State who you are, your most relevant qualification, and why this specific company. Be direct: "I have X years doing Y, and [specific thing about this company] is why I'm applying." If you have a prior connection (internship, event, used their product), mention it in one sentence here, not as a separate paragraph.

  **Career-arc sentence (MANDATORY when the profile shows a non-linear path):** If the candidate's trajectory is not self-evident — e.g., years of industry experience → returned for a master's → job market, or an experienced engineer who did a summer internship — the opening MUST include exactly one confident sentence framing the arc as deliberate capability-building. Not an apology, not a defense. Without it, the reader cannot tell whether they are looking at a new grad or a senior engineer, and that confusion alone sinks the application. Example (generic — adapt to the candidate's real arc): "After several years building production systems in industry, I went back for a master's to deepen the foundations I had been working around, and I am now looking to combine both." One sentence only; do not over-explain. This lives in the cover letter, never the resume (the resume stays clean and lets the timeline speak).
- **Body (1–2 paragraphs):** This is the core. Map 2–3 key JD requirements to your experience using the **impact formula**:

  > **Impact formula:** [What you did] + [in what context] + [what changed as a result]
  >
  > Bad: "I developed the service in language X across platforms A, B, and C" (process only)
  > Good: "I owned the service across platforms A, B, and C, shipping it to 4 clients" (process + outcome)
  > Better: "I owned the service that shipped to 4 clients, cutting integration time by designing a cross-platform abstraction layer" (process + outcome + how)

  For each JD requirement you address, the reader should understand: (a) you have the skill, (b) you used it at scale, (c) something measurable improved. If you can't show (c), at least show (b).

  **Scannable body option (recommended default):** For technical roles with clear skill requirements, carry the 2–3 core matches in short bullets, each with a bold topic heading (e.g., "From prototype to production", "Depth across platforms"), so HR/managers can see "why we need this person" in 30 seconds instead of reading prose word-by-word. Each bullet = what you did + result/scale. Alternatively, use inline bold markers for key skills in prose (e.g., "My **C/C++** experience comes from..."). Don't overdo bold — 2–3 terms max.

- **Closing (2–3 sentences):** Express interest in a conversation. If `profile.md` mentions a relevant local-language ability **and the JD lists a language requirement**, place it here naturally and honestly (state the real level — e.g., "I have been actively learning the local language", not "fluent enough for casual chat" unless that is true). By default, if the JD has no language requirement, do not mention language learning — it reads as filler. Mention availability. No restating credentials, no contact info (already in header).

### What NOT to Include

- **Standalone personal anecdote paragraphs:** A personal connection to the company (an event, product usage, campus visit) is worth 1–2 sentences max, integrated into the opening or closing. Never a full paragraph. It doesn't answer "does this person meet our requirements?"
- **Process descriptions without outcomes:** "I took ownership from architecture design through delivery" is a job description, not an achievement. Always pair process with result.
- **Passive value signals:** Don't write "Your job description describes something very similar" or "that is a dynamic I find productive." These ask the reader to draw the conclusion for you. Instead, state the value directly.
- **Ending on a weakness:** Don't close by mentioning gaps in your experience. If you must acknowledge a gap, do it briefly mid-letter and immediately pivot to how you'll address it, then close on strength.
- **Industry-specific jargon the target reader won't share:** Domain-specific protocols/interfaces from a previous industry are noise to an HR reader or an engineer in a different field. Translate them into general capability (project management, cross-functional coordination, building reliable systems). Keep only technical terms the *target* role's reader is expected to know.
- **Academic/thesis insider terms:** Jargon like specialized topology names or runtime-internals terms is not intuitive to HR or out-of-field engineers. Swap for plain, concrete phrasing while keeping genuinely role-relevant technical nouns and real project names (those are verifiable evidence, not jargon).

### Anti-AI Writing Rules

Same banned words as resume. Additionally:
- No "I believe I would be a great fit" or similar filler
- No restating the JD requirements back at the reader
- Vary paragraph length
- Use contractions sparingly but naturally (1-2 max)

### Tone & Culture

> **This section ships with a Nordic / Swedish-market default (*lagom* tone) as a worked example. It is replaceable.** If you are applying in a different market, swap this block for your target market's norms (e.g., a more assertive, achievement-forward register is expected by many US tech companies). You can also let Phase 2's culture keywords drive the register. Keep the underlying principle in all cases: plainness of language must not mean absence of argument.

The default tone is *lagom* — balance, competence without bravado, collective contribution over individual heroics. But lagom does NOT mean vague or directionless. The letter must have a clear through-line: "Here is specifically why I am right for this role."

**Tone principles (these are the DEFAULT for the example market, not optional adjustments):**
- **Plain language, clear direction:** Write like a competent person explaining their fit for a role, not like someone crafting a literary piece. "I'm very used to" is better than "I bring extensive expertise in." But every sentence must push toward the conclusion "this person fits." Plainness of language does not mean absence of argument.
- **Humility through facts:** Don't insert humility statements. Let the register stay naturally understated. Actively downgrade areas where you're not strong ("some experience from X"). Let strong areas speak through specific facts and outcomes, not adjectives.
- **Avoid intensity signals:** Phrases like "no margin for error", "no deadline missed", "relentless" read as aggressive. Rephrase to show care without drama.
- **Team context is natural, not performative:** Mention collaboration where it actually happened ("working with the hardware team, I..."), not as a virtue signal.
- **Confidence through specifics, not claims:** State what you did and what resulted. Never write "I am a strong match" — let the evidence make that obvious.
- **Curiosity signals:** One specific, authentic observation about the company's technology or mission signals genuine interest. But keep it brief — it's a supporting detail, not a paragraph.

**Mandatory checks before finalizing:**
1. **Specificity test:** "If I replaced the company name, could this letter work elsewhere?" If yes, add one more company-specific anchor.
2. **30-second scan test:** "Can HR identify my top 3 JD-matching qualifications by skimming?" If not, restructure for scannability.
3. **Direction test:** "Does every paragraph push toward 'hire this person'?" If any paragraph is just storytelling or atmosphere, cut or restructure it.
4. **Tone check:** "Does this match the register the target market expects?" If it sounds like a TED talk or a college essay where understatement is expected (or vice versa), rewrite.

### Draft Checkpoint

Cover letter content is now a confirmed draft **in context only**. Do NOT write any `.tex` file yet. Both resume and cover letter drafts are now ready for HR review in Phase 6.

---

## Phase 6: HR Screening Review (MANDATORY)

**⛔ Hard precondition: this phase reviews the in-memory draft content; nothing has been written to disk yet. Do NOT write any `.tex` file or run `pdflatex` before the HR review passes.**

Review the resume and cover letter drafts from a strict HR / hiring-manager perspective (based on the Phase 2 JD analysis and the Phase 4/5 drafted content). The goal is to find and fix problems before writing files.

### Review Dimensions

1. **Hard-requirement match** — Go through the JD's must-have skills/requirements one by one; for each, assess whether the resume concretely demonstrates it.
2. **Nice-to-have coverage** — Go through the nice-to-have skills one by one and compute the coverage ratio.
3. **Years-of-experience match** — JD's required years vs. years evidenced in the resume; any over/under-qualification risk?
4. **Seniority consistency** — Does the resume's summary positioning and bullet granularity match the JD's seniority level?
5. **Keyword density** — Do the JD's core keywords appear naturally (not stuffed)? Any important keyword missing?
6. **Red-flag check** — Look for these signals:
   - Employment gaps (over 3 months with no explanation; note: an in-progress degree is itself a reasonable explanation for a gap and should not be listed as an unexplained-gap risk)
   - Frequent job-hopping (average under 1 year)
   - Mismatched degree/credentials
   - Personal info on the resume (photo, age, marital status) — should not appear in regions governed by GDPR / anti-discrimination norms (this is an EU/Nordic default; adjust per target market)
   - Over-self-promotional phrasing ("relentless", "no deadline missed", "best-in-class") — read as a warning sign rather than a plus in understated-tone markets
7. **Cover-letter targeting** — Is the opening direct and clear? Does the body respond specifically to JD requirements rather than generically? Any obvious template feel?
8. **Two-reader check** — Does the first half let HR quickly confirm the JD match? Does the second half let the hiring manager see the candidate's incremental value (concrete outcomes/impact)?
9. **Impact-formula check** — Does each experience description in the body contain "what you did + in what context + what changed as a result"? If it's process-only with no result, flag it for improvement.
10. **Scannability** — Can HR identify the candidate's 3 core matches in 30 seconds by skimming? If the letter is pure prose requiring word-by-word reading to extract key info, flag it.
11. **Personal-connection proportion** — Is the personal connection to the company (events, product usage, etc.) kept within 1–2 sentences? If there's a whole paragraph of personal anecdote that doesn't serve the JD match, flag it for trimming.
12. **Direction check** — Does the whole letter have a clear line of argument ("I fit this role because X, Y, Z")? Or is it just listing experiences / telling stories? Does every paragraph push toward the "hire this person" conclusion?
13. **Tone check (per target market, default lagom)** — Scan for these negative signals:
   - Hero narrative (crediting outcomes to the individual, ignoring the team)
   - Intensity/pressure words ("relentless", "no margin for error", "driven by results at all costs")
   - Over-confident assertions ("I will single-handedly...", "I am the best candidate...")
   - Statements clearly at odds with the company values implied by the JD (work-life balance, inclusion, sustainability, etc.)
   - Language that is too polished/crafted (AI feel) — it should read like a person speaking, not like an essay
   - (Note: if the target market expects a more assertive tone, relax the hero/over-confidence judgment accordingly)
14. **Values alignment** — If the JD mentions sustainability, inclusion, flat hierarchy, work-life balance, or similar culture keywords, check that the cover letter echoes them naturally (not excessively, but not entirely absent).
15. **Targeting check** — Run the "company-name swap test": replace the company name in the cover letter with another company; if the letter still fully applies, targeting is insufficient. There should be at least one piece of customization grounded in this company's specific tech/product/culture.
16. **Understatement check (per target market)** — Does the overall tone match the target market's expectation? For understated-tone markets, check whether it stays plain and non-exaggerated; if any bullet stacks adjectives to claim ability (instead of showing it with facts), rewrite it.
17. **Language-ability check** — Only when the JD lists a language requirement AND profile.md has the relevant language ability, check that the cover letter mentions it (and states the level honestly). If the JD has no language requirement, mentioning it is not required.
18. **Identity / career-arc clarity (mandatory for non-linear paths)** — If the candidate has a non-linear path (industry → back to school → job market, or an experienced engineer doing a summer internship, etc.), check that the cover letter opening contains one confident, non-defensive sentence framing the career arc. If missing, flag as 🔴 needs revision: the reader can't tell new grad from senior, and identity ambiguity directly sinks the application. Note: this does not conflict with the resume "not explaining gaps" — the resume stays clean and lets the timeline speak; the arc is carried by the cover letter.

### Output Format (present in the user's working language)

```
## HR Review Report

**Overall verdict:** 🟢 Pass / 🟡 Optimization suggested / 🔴 Needs revision

### Hard-requirement match
| JD requirement | Resume evidence | Status |
|----------------|-----------------|--------|
| xxx            | xxx             | ✅/⚠️/❌ |

### Nice-to-have coverage
Covers X/Y items (XX%)

### Risk items
- [issues found, or "none"]

### Cover-letter assessment
- [targeting, appeal, template-feel, etc.]
- [where applicable: tone / values-alignment assessment]

### Improvement suggestions
- [specific, actionable suggestions in priority order]
```

### Follow-up Behavior Rules

**⛔ Hard gate: until the HR review is complete and the conditions below are met, do NOT write any `.tex` file or run `pdflatex`. This is a mandatory wait point and must not be skipped.**

- **🔴 Needs revision:** Revise the draft content (still in memory), re-run this review step until the result rises to 🟡 or 🟢, then proceed to Phase 7.
- **🟡 Optimization suggested:** After showing the report, **stop and wait for an explicit user response** ("optimize" or "compile as-is"). Do not perform any subsequent step before receiving a response. If the user confirms optimization, revise the draft and re-review; if they skip, proceed to Phase 7.
- **🟢 Pass:** After showing the report, proceed to Phase 7 to write files and compile.

---

## Phase 7: File Writing & Compilation

> **Precondition:** Only run this phase after the Phase 6 HR review is complete (🟡 user asked and an explicit response received, or 🟢 straight pass). This is the first time files are written to disk.

### Step 1: Write files

Create the output directory `~/Documents/job/{company}-{role}/` (kebab-case, lowercase, ASCII-only) if it doesn't exist. (Adjust the base path to wherever you keep applications.)

Write both files now:
- Resume → `~/Documents/job/{company}-{role}/resume-{company}-{role}.tex`
- Cover letter → `~/Documents/job/{company}-{role}/cover-letter-{company}-{role}.tex`

### Step 2: Compile

Check if `pdflatex` is available:

```bash
command -v pdflatex
```

- **If available:** Compile both `.tex` files to PDF, running `pdflatex` twice each (for references). Use `-interaction=nonstopmode`. Report any compilation errors clearly. After successful compilation, delete auxiliary files. Note: in zsh, a single `rm -f *.aux *.log ...` aborts if any one glob has no match (`nomatch`). Use a `find` form that won't error on no-match:
  ```bash
  find . -maxdepth 1 \( -name "*.aux" -o -name "*.log" -o -name "*.out" -o -name "*.toc" \
    -o -name "*.fls" -o -name "*.fdb_latexmk" -o -name "*.synctex.gz" \) -delete
  ```
- **1-page verification (resume only):** After compilation, parse the pdflatex output for the page count (e.g., `Output written on ... (N pages, ...)`). If the resume is more than 1 page, fix the `.tex` file and recompile — apply these levers in order until it fits:
  1. Flatten any remaining nested lists to single-level bullets
  2. Reduce bullets for least-relevant roles (trim to 1-2 bullets each)
  3. Tighten spacing: increase negative `\vspace` values by 1-2pt each
  4. Recompile and re-check. Repeat until exactly 1 page. Never delete substantive content (achievements, metrics) to fit — formatting adjustments come first.
- **If not available:** Tell the user:
  > `pdflatex` not found. To compile:
  > - macOS: `brew install --cask mactex-no-gui` or `brew install basictex`
  > - Ubuntu: `sudo apt install texlive-latex-base texlive-latex-recommended`
  > - Or upload the `.tex` files to Overleaf.

---

## Phase 7.5: Job Summary

**⛔ Hard gate: this step must run after Phase 7 compilation completes and before Phase 8 begins, without exception. No matter how many review iterations Phase 6 went through, do not skip this step.**

In the resume output directory `~/Documents/job/{company}-{role}/`, generate `job_summary.md` summarizing this application. No user confirmation needed — write it directly.

Content structure:

```markdown
# {Company} - {Role}

## Application info
- **Posting link:** {original JD URL; if the user pasted text, write "provided directly by user"}
- **Company:** {company}
- **Role:** {role}
- **Level:** {seniority level}
- **Date applied:** {YYYY-MM-DD}

## Requirements summary

### Must-have skills
{extracted from Phase 2 analysis}

### Nice-to-have skills
{extracted from Phase 2 analysis}

### Key responsibilities
{extracted from Phase 2 analysis}

## Resume strategy
{the resume strategy confirmed in Phase 3.5: Summary positioning angle, core experience selection, Skills grouping emphasis, Certifications handling}

## Cover letter strategy
{the cover letter strategy confirmed in Phase 3.5: opening hook angle, JD requirements covered in the body and the experience mapped to each, closing personalization detail}

## HR review notes
{the full Phase 6 review report: overall verdict, hard-requirement match table, nice-to-have coverage, risk items, cover-letter assessment, improvement suggestions}
```

When done, proceed to Phase 8.

---

## Phase 8: Feedback & Learning

After presenting the results, ask:

> Any feedback? I can adjust the resume/cover letter, or you can tell me preferences to remember for next time.

When the user provides feedback that should persist (e.g., "never use the word X", "always put projects before education", "I prefer shorter bullets"):

1. Read the current `${CLAUDE_SKILL_DIR}/assets/lessons.md`
2. Append a new entry at the top (below the header), formatted as:

```markdown
### YYYY-MM-DD: [Short description]
- [Specific rule or preference]
```

3. Write the updated file back.
4. If the feedback is about the current output, also regenerate the affected file with the fix applied.
