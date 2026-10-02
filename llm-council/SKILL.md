---
name: llm-council
description: "Pressure-test a decision by running it past a council of five sub-agents that each reason from a different lens, blindly review one another, and then hand their work to a chairman who issues a single verdict. Use when the user says 'council this', 'run the council', 'war room this', 'pressure-test this', 'stress-test this', 'debate this', or in Persian 'شورا', 'از شورا بپرس', 'شورا تشکیل بده'. Also use when the user brings a real decision with stakes and several options ('should I A or B', 'I'm torn between', 'is this the right move', 'what am I missing'). Do not use for factual lookups, drafting or summarising tasks, or low-stakes choices without a real tradeoff."
LLM Council
One model answering once gives you one sample of its thinking. You cannot tell whether that sample is sharp or lazy. This skill produces several deliberately different samples, makes them critique each other without knowing who wrote what, and then asks a chairman to decide. The result is a verdict you can trust more than a single reply, because its weak points have already been attacked.
The technique follows the multi-model "LLM Council" idea described by Andrej Karpathy. Here the diversity comes from different reasoning lenses run as sub-agents, not from different models.
---
When to convene
Convene for decisions where a wrong call is costly and the right call is not obvious: pricing, hiring, architecture choices, strategy pivots, evaluating a plan or a piece of copy that matters.
Skip it for questions with one correct answer, for creation tasks (write, summarise, translate), and for small choices with no real downside. In those cases just answer normally.
Cost. A full session launches 11 sub-agents. Choose a mode:
Full (default): 5 advisors, 5 blind reviewers, 1 chairman.
Quick: 5 advisors and 1 chairman, no review round (6 agents). Use it when the user asks for a quick or cheap council, or when the stakes are moderate.
If the user used an explicit phrase such as "council this", start right away. If you inferred the need from a decision-shaped question, say in one line that you are convening the council, then proceed.
Language
Reply, and instruct every sub-agent to reply, in the language of the user's question. Translate the verdict's headings into that language. Persian output should be natural, not word-for-word.
---
The five lenses
These are ways of reasoning, not characters to role-play. They are chosen to pull against each other.
The Skeptic. Assumes the plan has a serious flaw and goes looking for it: hidden costs, wrong assumptions, how it fails in practice. Not a pessimist; the one who asks what everyone else is avoiding.
The Reframer. Doubts the question itself. Asks what problem is really being solved, which assumptions are only habit, and whether a different question would lead somewhere better.
The Opportunist. Hunts for upside others overlook: a larger version, a neighbouring opening, an undervalued asset. Deliberately does not weigh risk; the Skeptic does that.
The Newcomer. Knows nothing about the user, the field, or the history, and reacts only to the words on the page. Exposes jargon, unstated context and anything that would confuse a stranger.
The Operator. Cares only about execution: what is the first concrete step, what resources and order of work does it take, where will it stall. Distrusts any idea with no clear Monday-morning action.
The pairings are intentional: Skeptic against Opportunist (downside vs upside), Reframer against Operator (rethink vs just ship), with the Newcomer as an outside check on all four.
---
Procedure
1. Gather context and frame the question
First, quickly look for context that would make the advisors specific instead of generic: files the user referenced or attached, a project instruction file such as `CLAUDE.md`, a notes or memory folder, and earlier council transcripts on the same topic. Read only what bears on this question, and stop after a couple of minutes' worth of looking. If no file access exists, work from the conversation.
Privacy rule: never read or forward credentials, tokens, `.env` files or other secrets. The framed question goes to every sub-agent, so include only what is relevant.
Then write one neutral framed question containing:
the decision or question, stated plainly;
the relevant facts from the user's message;
the relevant facts from any files you read (stage, audience, constraints, numbers, past results);
what is at stake.
Do not slip in your own opinion. If the request is too vague to frame ("council this: my business"), ask exactly one clarifying question, then continue.
2. Advisors (5 sub-agents, in parallel)
Launch all five at once so none can influence another. Each gets its lens, the framed question, and the instruction to commit fully to its angle rather than hedge.
Advisor prompt:
```
You are the [LENS NAME] on a decision council.

Your lens: [LENS DESCRIPTION]

The decision brought to the council:
---
[FRAMED QUESTION]
---

Analyse it strictly through your lens. Be concrete and take a position; do not
try to be balanced, because other advisors cover the other angles. Do not state
your lens name or role anywhere in your answer. Write in [USER LANGUAGE].
Length: 150 to 300 words. Start with the analysis, no introduction.
```
3. Blind review (5 sub-agents, in parallel; skipped in Quick mode)
Collect the five answers. For each of the five reviewers, assign the answers to the letters A to E in a fresh random order, and record that reviewer's letter-to-lens mapping. Reviewers have no lens of their own; they all use the same prompt.
Reviewer prompt:
```
Five advisors independently answered the same question. Their answers are
anonymous.

Question:
---
[FRAMED QUESTION]
---

Response A: [text]
Response B: [text]
Response C: [text]
Response D: [text]
Response E: [text]

Reply to three points, citing responses by letter:
1. Which response is the most useful, and why?
2. Which response has the most serious blind spot, and what is it?
3. What did every response overlook?

Write in [USER LANGUAGE]. Maximum 200 words.
```
Before passing reviews on, replace each letter with the lens it stood for (using that reviewer's own mapping). Without this step the chairman cannot tell who "Response C" was.
4. Chairman (1 sub-agent)
The chairman receives the framed question, the five advisor answers labelled by lens, and the five reviews with letters already translated to lens names (in Quick mode: only the advisor answers).
Chairman prompt:
```
You chair a decision council. Read the five advisor answers and the peer
reviews, then write the verdict.

Question:
---
[FRAMED QUESTION]
---

Advisor answers: [five answers, labelled by lens]
Peer reviews: [five reviews, letters replaced by lens names, or "none" in Quick mode]

Write the verdict with exactly these sections (translate headings into
[USER LANGUAGE]):

Confidence: High, Medium or Low, with one sentence of justification.
Agreement: points several advisors reached independently.
Disagreement: real conflicts; state each side and why thoughtful people differ.
Surfaced in review: issues that appeared only after the blind review. (Omit
  this section in Quick mode.)
Decision: a direct recommendation with reasons. "It depends" is not allowed;
  if it truly depends, say on what and pick the likelier branch. You may
  overrule the majority when a minority argument is stronger.
First move: one concrete action to take next.
What would change this call: one or two facts that, if true, would reverse
  the decision.

Be direct and keep the whole verdict under about 500 words.
```
5. Deliver the verdict
Show the chairman's verdict in the chat, in Markdown, under a heading `Council Verdict: <short topic>`. Keep it scannable. Do not produce an HTML page or any file unless the user asks for one.
6. Optional transcript
Only if the user asks, or the decision is clearly significant, save `council-transcript-<timestamp>.md` into an `active/` directory (create it if absent). Include the framed question, all advisor answers, the reviews and the verdict.
---
Rules and edge cases
Run advisors in parallel, and reviewers in parallel. Sequential runs let early answers contaminate later ones.
Reviews must be blind, and each reviewer gets its own shuffle. Do not skip the letter-to-lens translation for the chairman.
If sub-agents are unavailable, do the same stages yourself one after another: write each advisor's answer in full before starting the next, never softening one lens because of another, and tell the user the council ran in single-agent mode.
If one sub-agent fails, continue with the rest and note that the council was one voice short.
Do not rerun the council because the user disagrees with the verdict. Rerun only if they supply new facts; otherwise discuss the verdict with them.
Never present the council as a source of facts. If an advisor states a number or claim that matters, flag whether it came from the user's material or from the advisor.
