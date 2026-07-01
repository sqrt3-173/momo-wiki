# LLM API cost + quality reference (July 2026)

> Prices per **1M tokens**, input/output separate. Verified from live sources 2026-07-01 (Anthropic/OpenAI
> docs, OpenRouter, provider pages). **Prices move fast — re-verify before relying on any figure.** Use this
> to route agent work to the cheapest model that meets the quality bar (see the tiering strategy below).

## The chart (most expensive → cheapest)

| Model | In $/M | Out $/M | Tier | Best for |
|---|---|---|---|---|
| **Claude Opus 4.8** | 5.00 | 25.00 | Frontier | Hardest reasoning, agentic coding, audit-council |
| **OpenAI GPT-5.5** | 5.00 | 30.00 | Frontier | Top-end reasoning + broad tool ecosystem |
| **OpenAI GPT-5.4** | 2.50 | 15.00 | Near-frontier | Cheaper flagship-class workhorse |
| **Claude Sonnet 5** | 2.00 | 10.00 | Near-frontier | Best-value production workhorse (intro→Aug 31; then 3/15) |
| **Gemini 3.1 Pro** | 2.00 | 12.00 | Near-frontier | Huge context, multimodal (4/18 above 200k ctx) |
| **Gemini 3.5 Flash** | 1.50 | 9.00 | Strong-mid | Fast multimodal at scale |
| **Claude Haiku 4.5** | 1.00 | 5.00 | Strong-mid | Cheap fast Claude — high-volume classify/extract |
| **GLM-5.2** (Z.ai) | 0.93 | 3.00 | Strong-mid (open) | Strong open coder/agent, low cost |
| **GPT-5.4-mini** | 0.75 | 4.50 | Strong-mid | Cheap OpenAI for routine tasks |
| **DeepSeek R1** | 0.80 | 0.80 | Strong-mid | Cheapest strong reasoning (flat rate); *now superseded by V4* |
| **Gemini 3.1 Flash-Lite** | 0.25 | 1.50 | Bulk-fast | Cheapest Google multimodal |
| **DeepSeek V3** | 0.23 | 0.34 | Bulk-fast | Ultra-cheap general chat/summarize (*V4 is current gen*) |
| **Qwen3-Coder 480B** | 0.22 | 1.80 | Strong-mid (open) | Near-frontier open code quality |
| **GPT-5.4-nano** | 0.20 | 1.25 | Bulk-fast | Cheapest OpenAI — classify/route |
| **gpt-oss-120b** | 0.15 | 0.60 | Strong-mid (open) | Strong reasoning at bulk price (Together/Groq) |
| **Llama 4 Maverick 400B** | 0.15 | 0.60 | Strong-mid (open) | Open MoE flagship, cheap general use |
| **gpt-oss-20b** | 0.05 | 0.20 | Bulk-fast (open) | Tiny open model, cheapest hosted tier |
| **Self-host (M3 Max)** | ~0 | ~0 | Bulk-fast | gpt-oss / Qwen / Llama local — ~$0 marginal (electricity) |

## How much cheaper is open vs Claude? (blended ~3:1 in:out)
- Opus 4.8 ≈ $10/M · Sonnet 5 ≈ $4/M (blended).
- GLM-5.2 (~$1.45): **~7× cheaper than Opus**, ~3× vs Sonnet.
- Qwen3-Coder (~$0.62): **~16× vs Opus**, ~6× vs Sonnet.
- Llama 4 Maverick / gpt-oss-120b (~$0.26): **~38× vs Opus**, ~15× vs Sonnet.
- gpt-oss-20b (~$0.09): **~110× vs Opus**.
- Self-hosted on M3 Max: effectively **∞× cheaper** (fixed hardware).

## Caveats
- **Claude tokenizer:** Opus 4.7+/Sonnet 5 emit **~30% more tokens** for the same text → the effective Claude-vs-open gap is *wider* than sticker rates suggest.
- **DeepSeek current gen = V4** (V4 Flash $0.14/$0.28, V4 Pro $1.74/$3.48). V3/R1 above are prior-gen but still listed.
- **gpt-oss-120b** price varies by host (OpenRouter routes as low as $0.03/$0.15; Together/Groq ~$0.15/$0.60).
- **OpenAI o-series** effectively folded into GPT-5.x — no longer priced standalone.

## Tiering strategy (route agent work by stakes)
- **Frontier (Opus 4.8 / GPT-5.5)** → the genuinely hard reasoning, judgment, audit-council, craft-sensitive + customer-facing output (NUNU's EDMs, nuanced BD calls, final review).
- **Sweet-spot workhorse (Sonnet 5)** → most production work; best value at intro pricing.
- **Cheap mid (Haiku / GLM-5.2 / Qwen3-Coder / gpt-oss-120b)** → high-volume agent grunt work with a quality bar.
- **Bulk (gpt-oss-20b / DeepSeek V3 / local self-host)** → extraction, classification, tagging, first-pass — where local ≈ frontier quality at ~$0. **This is the killer tier for the BD/CRM enrichment pipeline.**

The rule: **cheapest model that clears the quality bar for that task.** Benchmark a cheap/open model on the real task before trusting it — measure, don't assume.
