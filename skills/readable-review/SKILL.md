---
name: readable-review
description: Review code by running the code-review skill behind the scenes, then present its findings as a plain-language, numbered, action-grouped report that a person can decide from in one read. Use this skill whenever the user asks for a readable, plain-language or easy-to-follow review of code, a branch, a PR or recent changes, or types /readable-review, or says a review's output is hard to read, jargon-heavy or badly formatted. Also use it to re-present review findings that already exist (a /code-review output, reviewer subagent reports, pasted notes). It checks the important claims against the actual code, groups findings into Fix now / Needs your decision / Low priority, and ends with a concrete proposal.
---

# Readable review

Review findings are usually written by reviewers (human or model) for other reviewers: terse, full of internal vocabulary, ranked by severity but not by what the reader has to do. The reader of this report is deciding what to fix, what to think about and what to ignore. Write for that decision. A good report lets them answer "which of these do I act on?" without re-reading anything or opening the code.

This skill gets its findings from the `code-review` skill and presents them. The review skill does the finding; this skill does the checking and the writing. If you notice a real problem while verifying a claim, add it as a finding; otherwise stay with what the review reported.

## Process

### 1. Get the findings

**Default: run the review.** Invoke the `code-review` skill with the Skill tool and pass along whatever the user typed after the command (a PR number, branch, path or commit range; an effort level; `--max-findings`). Add nothing of your own. In particular, never add `--comment` (it posts to a PR) or `--fix` (it edits files) unless the user typed it, because this skill's job ends at presenting. Follow that skill's instructions to produce the findings. If the Skill tool reports that `code-review` is not available, tell the user this skill needs a review skill installed (the `code-review` skill that ships with Claude Code, or another one they name) and offer to present existing findings instead; do not review the code yourself in its place.

**Use existing findings instead when the user points at them**: "rewrite that review", pasted output, reviewer subagent reports already in the conversation, and no target typed. Re-running a long review only to reformat it wastes the user's time.

**Use a different review skill when the user names one**, for example the two-axis `mattpocock-skills:code-review`. If its findings come in separate axes (such as "Standards" and "Spec"), keep them under separate headings and prefix the numbers (S1, P1) so one axis never masks the other.

If the review has nothing to look at (for example a clean working tree and no target given), say so and ask what to review: a commit range, a branch or a PR. A guessed range risks reviewing the wrong thing. If the review returns no findings, say that plainly, along with what was reviewed. Presenting nothing is better than inventing findings.

The readable report is what the user reads. Do not also paste the review's raw findings as text, since that gives them the hard-to-read version this skill exists to replace.

### 2. Verify before presenting

Review output is a set of claims, and some will be wrong or stale: a rule that was already changed, a scenario the code can't reach, a behaviour the project chose on purpose. Presenting an unchecked claim as fact sends the reader off to fix something that isn't broken.

Spot-check in proportion to the stakes. Prioritise:
- findings that drive a "fix now" recommendation,
- findings that sound surprising, or contradict a decision the user made earlier,
- findings that name a specific line, constant, file or behaviour you can confirm with a quick read or search.

Do this with the cheapest tool that settles it (open the file, grep, read the test). Then:
- If the code confirms the claim, say so.
- If the code disagrees, say what you saw and downgrade or drop the finding. Disagreeing with the review is part of the job.
- Findings you did not check stay in the report, labelled as unchecked, so the reader knows how much weight each carries.

Do not run the full test suite or builds unless a claim depends on it.

### 3. Sort each finding into an action group

Group by what the reader should do, since that is the question they came with:

- **Fix now**: a real problem, and the fix is clear and fairly small. No product or design choice is left open.
- **Needs your decision**: a real issue where the right answer depends on a choice only the owner can make, or it conflicts with something they decided earlier. Lay out the options and recommend one.
- **Low priority**: real but cheap, rare or consistent with how the codebase already works, plus anything that is by design, already settled, or where the review's claim turned out wrong or overstated. Say which, and say why in a phrase.

An optional closing group, **Checked and fine**, holds a one-line list of things the review explicitly confirmed are correct. Include it when it reassures the reader about areas they might worry about.

### 4. Write the report

Use this structure. Number findings continuously across groups (or with axis prefixes) so the reader can reply "do 1, 3 and 5".

```
<One or two sentences: what was reviewed, what was checked against the code, what was not, and whether tests/builds were run.>

## At a glance

| # | Area | In one line | Verdict |
|---|------|-------------|---------|
| 1 | <feature or file area> | <the problem in plain words> | **Fix now** |
...

## Fix now

**1. <The problem as a plain sentence a non-specialist could follow.>**
- **What's wrong:** <the mechanism, in one or two sentences>
- **When it happens:** <a concrete scenario: who does what, and what goes wrong>
- **Recommendation:** <the specific change, sized: "a two-line change", "touches three files">

## Needs your decision

**2. <Plain sentence.>**
- **What's wrong:** ...
- **When it happens:** ...
- **Options:** (a) ... (b) ... (c) ...
- **My recommendation:** <pick one and say why in a clause>

## Low priority

**3. <Plain sentence.>**
- **My take:** <why it is low: rare, by design, matches existing code, the review overstated it>
- **If you want it fixed:** <the cheap option, or "leave">

## Summary
<Counts per group and the one or two things that matter most.>

<Concrete proposal: "I suggest fixing 1, 3 and 5 now; 2 needs your call. Want me to do those?">
```

Writing guidance, each point with its reason:
- **Plain words first.** Replace reviewer shorthand ("Feature Envy", "TOCTOU", "N+1") with what actually happens, or add a short gloss in parentheses. The reader should not need the reviewer's vocabulary.
- **Scenarios, not abstractions.** "A person with a pending application posts to the other form and gets a second role" is checkable and memorable; "insufficient validation on submit" is not.
- **Size every recommendation.** "Two-line change" versus "needs work in another repo" is half the decision.
- **One verdict per finding**, stated up front in the table so the report can be skimmed.
- **Say who decided what.** If a finding contradicts something the user decided earlier, say so and cite the decision, so they are not asked to re-litigate it by surprise.
- **Keep each finding to roughly three to five lines.** Detail belongs in the code, not the report. Reference files and lines where it helps, as clickable paths if the environment supports them.
- **Stay honest about uncertainty.** "Looks right from reading the code; not run" beats false confidence.

### 5. Close with a proposal, then wait

End with a specific suggestion for what to do first and an invitation to choose. Presenting is the job; do not edit code, commit or start fixing until the user says which findings to act on. When several findings need the user's call, ask one precise question per decision.

## Example (abbreviated)

> I reviewed the last three commits with `code-review` and checked findings 1, 2 and 4 against the code; 1 and 2 hold up, 4 was overstated. I did not run the tests.
>
> ## At a glance
> | # | Area | In one line | Verdict |
> |---|------|-------------|---------|
> | 1 | Form submit | The "who may apply" rule is only checked on page load, not on submit | **Fix now** |
> | 2 | English level | Two forms overwrite the same field | **Needs your decision** |
> | 3 | Fit answers | Two simultaneous saves could create two rows | Low priority |
>
> ## Fix now
> **1. The rule is only checked when the page loads, not when the form is submitted.**
> - **What's wrong:** the layout guard doesn't run before a form POST, and the action only checks that the person doesn't already hold *this* role.
> - **When it happens:** someone with a pending application posts to the other role's form and ends up with two pending roles.
> - **Recommendation:** run the same rule check in both actions and redirect home when it fails. Small change, plus two tests.
>
> ## Needs your decision
> **2. Both forms write the same person field, with different allowed values.**
> - **What's wrong:** …
> - **Options:** (a) latest answer wins; (b) only write it when empty; (c) agree an owner with the CRM side.
> - **My recommendation:** (a), since it matches "current level".
>
> ## Low priority
> **3. Two simultaneous saves could create two rows.**
> - **My take:** the CRM already rejects duplicates with a 409, and the disabled submit button narrows the window. Leave it.
>
> I suggest fixing 1 now and deciding 2 together. Want me to start on 1?
