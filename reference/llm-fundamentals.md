# LLM Fundamentals (plain-language reference)

<!-- Filed by triage 2026-07-03 from Karpathy's "Intro to Large Language Models"
     (inbox item 6, video_ref). Written for Eli: enough to follow architecture
     conversations, no jargon dumps. Each section carries atom provenance. -->

## What an LLM actually is
<!-- atom 10 · item 6 · 2026-07-03 -->
An LLM is concretely two files: a parameters file of trained weights (Llama 2 70B ≈ 140GB
of float16 weights) and a small run file (~500 lines of C). All capability lives in the
weights. (Karpathy, "Intro to LLMs")

## Pretraining — where knowledge comes from
<!-- atom 11 · item 6 · 2026-07-03 -->
Pretraining is next-word prediction over massive text and acts as lossy compression — "a
zip file of the internet." Expensive and infrequent (Llama 2 70B: ~10TB text, ~6,000 GPUs,
~12 days, ~$2M). Knowledge comes from this stage — which is why model knowledge is stale
and why agent systems add tools/retrieval instead of relying on weights.

## Training pipeline — behaviour is cheap, knowledge is not
<!-- atom 12 · item 6 · 2026-07-03 -->
Training is staged: expensive pretraining (knowledge), then cheap fine-tuning on curated
Q&A to shape an assistant, optionally RLHF from human comparison labels. Practical
takeaway: behaviour/format is cheap to change (prompts, fine-tunes); knowledge is not —
which is why external memory (wiki, retrieval, tools) is the right place for facts.

## Scaling laws
<!-- atom 13 · item 6 · 2026-07-03 -->
LLM performance improves predictably with more parameters and training data (scaling
laws) — capability gains from newer/larger models are a forecastable trend, not a gamble.
Implication for our builds: don't over-engineer permanent workarounds for current-model
weaknesses; revisit model choice as generations land (see [[../ops/model-cost-reference]]).

## Tool use
<!-- atom 14 · item 6 · 2026-07-03 -->
LLMs become far more capable when orchestrated with tools (browsing, calculator, code
interpreter, image generation) — the model plans and delegates rather than doing everything
from weights. This is the premise the MOMO engine and all our agent workflows are built on.

## The LLM OS mental model
<!-- atom 15 · item 6 · 2026-07-03 -->
Karpathy's LLM OS analogy: the LLM is the kernel of a new operating system — context window
as RAM; tools, files/embeddings, and other resources orchestrated around it. Useful lens
for the MOMO engine: context is the scarce resource, so durable state lives outside the
window (wiki, momo_work Postgres) and idle work stays cheap (heartbeat's cheap-check-first).

## System 1 vs System 2
<!-- atom 16 · item 6 · 2026-07-03 -->
Base LLM sampling is fast System 1 token prediction; the major direction is System 2
deliberate reasoning — trading time/compute for accuracy. This is now real as
reasoning/extended-thinking modes: an accuracy-vs-cost knob we choose per task.

## Limits of self-improvement
<!-- atom 17 · item 6 · 2026-07-03 -->
Self-improvement beyond imitating human data (AlphaGo self-play analogy) is limited in open
language domains by the lack of clear reward functions — why capability gains still come
mainly from scaling and training data, not self-play.
