# apply — ATS-Optimized Resume & Cover Letter Generator

A skill that turns a job description (URL or pasted text) into a
tailored, ATS-friendly LaTeX **resume** and **cover letter**, written to sound
authentically human rather than AI-generated.

It runs an 8-phase pipeline: JD analysis → strategy preview (you approve before
anything is written) → resume + cover letter drafting → a strict HR/hiring-manager
review gate → file writing + `pdflatex` compilation → a per-application summary →
feedback that it remembers for next time.

## Install

Clone the repo into your skills directory so the skill resolves as `apply`:

```bash
git clone https://github.com/1carusalwayswa/apply ~/.claude/skills/apply
```

(Adjust the destination to wherever your environment loads skills from.)

## Set up before first use

1. **`assets/profile.md`** — Fill in your own background (work history, skills,
   education, awards, projects). This is the single source of truth; the skill
   will not invent any skill or metric that isn't written here. You can write in
   any language — output is generated in English.
2. **`assets/resume-template.tex`** — Replace the placeholder contact block
   (name, email, links) at the top with your own. Keep the section names for ATS
   compatibility. The body is regenerated per job, so you rarely edit it by hand.
3. **`assets/cover-letter-template.tex`** — Optional; the sender block is filled
   per job.
4. **`assets/lessons.md`** — Starts empty. The skill appends your preferences
   here automatically as you give feedback (Phase 8), and treats them as hard
   constraints on later runs.

## Usage

```
/apply <job description URL or pasted text>
```

The skill pauses for your confirmation twice (strategy preview, and the HR
review gate) before it writes or compiles anything.

## Notes on customization

- **Target-market tone:** The cover-letter "Tone & Culture" section ships with a
  Nordic / Swedish *lagom* register as a worked example. It is clearly marked as
  replaceable — swap it for your target market's norms (e.g. a more
  achievement-forward register for many US tech companies), or let the JD's
  culture keywords drive it.
- **Honesty guardrails:** Skills must be traceable to `profile.md` and
  interview-defensible; bullets must pair process with outcome; no em dashes or
  AI-tell phrases. These are baked into the pipeline.
- **Output location:** Generated files go under `~/Documents/job/{company}-{role}/`
  by default (resume, cover letter, and a `job_summary.md`). Change the base path
  in Phase 7 if you keep applications elsewhere.

## License

The LaTeX resume template is based on "Ethan's Résumé Template" by necusjz,
licensed CC-BY-4.0. The attribution header is kept in `resume-template.tex`.
