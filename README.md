LLM Council Skill
فارسی | English
A Claude skill that pressure-tests a decision. Instead of one answer, you get five independent analyses from deliberately different lenses, a blind peer-review round, and a chairman's single verdict.
Works in Claude Code and in Claude (claude.ai) wherever skills and sub-agents are available. Replies in the language you ask in, including Persian.
How it works
```
your question
     |
 frame it (+ context from your files)
     |
 5 advisors in parallel:  Skeptic | Reframer | Opportunist | Newcomer | Operator
     |
 5 blind reviewers (answers shuffled and anonymised per reviewer)
     |
 chairman -> verdict: confidence, agreement, disagreement,
             decision, first move, what would change the call
```
Two modes:
Mode	Agents	When
Full (default)	11 (5 + 5 + 1)	Decisions where a wrong call is expensive
Quick	6 (5 + 1)	Moderate stakes, or you want it cheap
Good and bad questions
Good: "Should we build this module in-house or buy it?", "Which of these three offers is strongest?", "Here is my plan; where does it break?"
Not for: factual lookups, writing or summarising tasks, small choices with no real tradeoff.
Install
Claude (claude.ai): zip the `llm-council` folder (the zip must contain the folder with `SKILL.md` inside) and upload it in Claude's Skills settings.
Claude Code: copy the `llm-council` folder into `~/.claude/skills/` (personal) or `.claude/skills/` inside a project.
Use
Say one of: `council this`, `run the council`, `war room this`, `pressure-test this`, `stress-test this`, `debate this`. Add "quick" for the cheaper mode. Persian triggers also work: `شورا`, `از شورا بپرس`.
Notes
The skill may read a project instruction file (such as `CLAUDE.md`) or notes you reference, to give the advisors context. It is instructed never to read or forward secrets.
Output is shown in chat. A transcript is saved only if you ask.
A council improves reasoning; it does not verify facts. Check any numbers that matter.
Credits
The council-with-peer-review technique comes from Andrej Karpathy's llm-council. Ole Lehmann published an earlier Claude-skill adaptation of the idea, aiwithremy/claude-skills-llm-council, which inspired this project. This repository is an independent rewrite with its own wording, lenses, modes and procedure.
License
MIT, see LICENSE.
