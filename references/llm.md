# LLM App Guardrails

Applies at every size whenever the app calls an LLM, runs agents or tools, or processes
AI-generated content. These rules exist because LLM apps have a threat class the rest of
the skill doesn't cover: the model itself is a component that can be manipulated through
the text it reads, and its output can't be trusted just because the app generated it.

The one-sentence version: **everything going into the model may be adversarial, and
everything coming out is untrusted input.**

## Prompt injection

- **Data is not instructions.** Retrieved documents, user messages, tool results, file
  contents, web pages - anything the model reads may contain text designed to override
  its instructions ("ignore previous instructions and..."). Assume it will.
- **Separate instructions from data structurally.** System rules live in the system
  prompt; untrusted content gets clearly delimited and labeled as data to be processed,
  never merged into the instruction stream.
- **User content never overrides system rules.** If a user or a retrieved document can
  change what the app is allowed to do, that's a vulnerability, not a feature.
- **Injection resistance degrades, it doesn't hold.** Delimiters and instructions help
  but don't guarantee. Design so that a successfully injected model still can't do real
  damage - that's what the tool rules below are for.

## Model output is untrusted

- **Never execute model output directly.** No eval, no shell execution, no
  auto-applying generated SQL or code without a sandbox or review step. Model-generated
  code gets the same review as third-party code, because that's what it is.
- **Escape before rendering.** Model output rendered into HTML gets the same XSS
  treatment as user input - because via injection, it *is* user input.
- **Schema-validate structured output.** When the app expects JSON, parse and validate
  against the expected shape; handle malformed output, refusals, and empty responses as
  ordinary error states with the floor's async rules. Never assume the model complied.
- **Verify claims before acting on them.** A model's statement about an API, a fact, or
  the state of the world is a hypothesis. If the app takes consequential action based
  on model output, the output gets checked first.

## Tool and agent safety

- **Least privilege.** Tools exposed to a model get the narrowest possible scope -
  read-only where read-only works, allowlisted domains for anything that fetches,
  scoped credentials rather than admin keys.
- **Human approval before consequences.** The floor's confirmation triggers apply with
  extra force: a model never autonomously deletes data, spends money, sends
  communications, or touches production. Approval sits between the model's intent and
  the privileged action.
- **Cap the loops.** Agent runs get a hard iteration limit and a cost/token budget.
  A runaway loop is both a bill and an attack surface.
- **Log tool calls.** Every tool invocation gets logged (respecting the floor's rule on
  sensitive data) so a bad run can be reconstructed.

## Data handling

- **Prompts leak.** Assume anything placed in a prompt - system or otherwise - can be
  extracted by a determined user. No secrets in prompts, ever. API keys live
  server-side; the model gets capabilities through tools, not credentials.
- **Prompts and completions may contain PII.** The floor's logging rule applies to LLM
  traffic: don't ship raw conversations to logs or third-party analytics without
  scrubbing.
- **Know your provider's retention.** If user data flows to a model API, that's a trust
  boundary (floor item 6) - know what the provider stores and say so in the privacy
  story.

## Audit hook

When any audit runs (Critical Sweep, Standard, or Full) on an app with LLM features,
add these hunts to the security pass:

1. Any path where untrusted content reaches the instruction stream unmarked.
2. Any model output that gets executed, rendered, or acted on without
   validation/escaping/review.
3. Any tool a model can call that could destroy data, spend money, or reach internal
   networks without human approval.
4. Any secret present in a prompt template.
5. Missing loop/cost caps on agentic flows.
