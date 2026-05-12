# LLM Jailbreak & Red-Teaming Resources 2026

A curated list of currently relevant public resources for AI red-teaming, jailbreak research, and adversarial prompt testing against major commercial and open-weight LLMs.

This collection is maintained as reference material for security professionals and AI red-teamers working with the **2026 state of LLM adversarial robustness**. Entries are grouped by what they actually offer rather than by branding, and each carries a candid note on how useful it is *right now* for sourcing prompts that still work against frontier models.

## Table of Contents

- [Conceptual Framings](#conceptual-framings)
  - [Contextual Drift — Subtle Context Manipulation as an LLM Exploitation Technique](#contextual-drift--subtle-context-manipulation-as-an-llm-exploitation-technique)
  - [Related Empirical and Methodological Work](#related-empirical-and-methodological-work)
- [Active Red-Teaming Frameworks](#active-red-teaming-frameworks)
  - [NVIDIA Garak](#nvidia-garak)
  - [Microsoft PyRIT](#microsoft-pyrit)
  - [Promptfoo Red-Team Plugins](#promptfoo-red-team-plugins)
- [In-the-Wild Jailbreak Collections](#in-the-wild-jailbreak-collections)
  - [elder-plinius / L1B3RT4S](#elder-plinius--l1b3rt4s)
  - [davidegat / happy-prompts](#davidegat--happy-prompts)
  - [CyberAlbSecOP / Awesome_GPT_Super_Prompting](#cyberalbsecop--awesome_gpt_super_prompting)
- [Academic & Research Aggregators](#academic--research-aggregators)
  - [CryptoAILab / Awesome-LM-SSP](#cryptoailab--awesome-lm-ssp)
  - [xunguangwang / SoK4JailbreakGuardrails](#xunguangwang--sok4jailbreakguardrails)
  - [user1342 / Awesome-LLM-Red-Teaming](#user1342--awesome-llm-red-teaming)
  - [chawins / llm-sp](#chawins--llm-sp)
- [Practical Notes on Currency and Use](#practical-notes-on-currency-and-use)

---

## Conceptual Framings

Before the tools and lists, two pieces of conceptual background that explain *why* the techniques in the rest of this document work. Neither is a prompt source; both are useful for reasoning about the underlying mechanics.

### Contextual Drift — Subtle Context Manipulation as an LLM Exploitation Technique
**Article:** https://www.linkedin.com/pulse/contextual-drift-subtle-context-manipulation-llm-martin-f%C3%BCrholz-f52re/
**Last updated:** Originally published December 2024, substantively updated May 2026.
**What it is:** A taxonomic framing of multi-turn jailbreaking as a single class of attacks rooted in the in-context learning property of transformer LLMs. Frames Crescendo, Many-Shot Jailbreaking, PAIR, TAP, and related methods as specific instantiations of a broader drift principle rather than as independent techniques. The May 2026 update addresses why current reasoning models, orchestrated agents, and classifier stacks (including Gemini 3.1 Pro) remain susceptible despite substantial defensive innovation since 2024.
**Applicability for current jailbreaks:** **High as a framing.** Useful for reasoning about which specific tool or technique to reach for in a given scenario, and for understanding why patches against individual attack variants do not close the underlying gap.

### Related Empirical and Methodological Work
- **Crescendo** (Russinovich et al., Microsoft, 2024) — the canonical multi-turn drift framework, now implemented in PyRIT.
- **Single-Turn Crescendo Attack (STCA)** (Aqrawi & Abbasi, 2024, https://arxiv.org/pdf/2409.03131) — demonstrates that the drift principle does not strictly require multiple conversational turns and generalizes to multimodal generation systems.
- **Multi-Turn Human Jailbreaks (MHJ)** (Li et al., Scale AI / UC Berkeley, 2024, https://arxiv.org/pdf/2408.15221) — empirical confirmation that human-driven multi-turn attacks exceed 70% ASR on HarmBench against defenses reporting single-digit ASRs against single-turn attacks. Includes a public dataset of 2,912 prompts across 537 multi-turn jailbreaks compiled from commercial red-teaming engagements.

---

## Active Red-Teaming Frameworks

These are not lists; they are tools that ship with attack libraries and are extended as new techniques are published. For live demos these are the most reliable foundation, because maintainers integrate new probes faster than static repositories.

### NVIDIA Garak
**Repository:** https://github.com/NVIDIA/garak/tree/main/garak/probes
**Last updated:** Actively maintained through 2026 (latest stable v0.14.x, released February 2026).
**What it is:** NVIDIA's open-source LLM vulnerability scanner — often described as "nmap for LLMs". Ships with 37+ probe modules covering DAN-family jailbreaks, encoding bypasses, prompt injection, latent injection, glitch tokens, malware generation, package hallucination, and visual jailbreaks for multimodal targets.
**Where the jailbreak content lives:** Distributed across `garak/probes/*.py` (probe logic and inline prompt templates) and `garak/data/` (corpora and dataset references). `garak --list_probes` enumerates the full inventory.
**Applicability for current jailbreaks:** **High.** New techniques tend to land in Garak shortly after publication. The probe model supports adaptive/dynamic attacks (`atkgen`, `tap`) rather than static text, which significantly extends shelf-life against patched filters.

### Microsoft PyRIT
**Repository:** https://github.com/microsoft/PyRIT/tree/main/pyrit/datasets/jailbreak/templates
**Last updated:** Actively maintained through 2026 (latest release v0.11.x, February 2026).
**What it is:** Microsoft's Python Risk Identification Toolkit, originally built from the experience red-teaming Bing Chat and Copilot. Strongest in multi-turn and orchestrated attacks (Crescendo, TAP, PAIR) rather than single static prompts.
**Where the jailbreak content lives:** The `templates/` folder under `pyrit/datasets/jailbreak/` holds curated YAML templates. Additional research datasets (HarmBench, JailbreakBench, DAN, Do-Not-Answer, SORRY-Bench, SALAD-Bench, AdvBench, OR-Bench, Many-Shot Jailbreaking, and others) are loaded via `load_*_dataset()` functions.
**Applicability for current jailbreaks:** **Very high**, especially for *adaptive* and *multi-turn* scenarios. Less useful as a flat copy-paste list. Note: v0.11.0 introduced Jinja2 sandboxing — older notebooks may need `is_jinja_template=True` for templates to render as expected.

### Promptfoo Red-Team Plugins
**Repository:** https://github.com/promptfoo/promptfoo/tree/main/src/redteam/plugins
**Last updated:** Actively maintained through 2026 (commercial-backed, regular releases).
**What it is:** Plugin architecture for the Promptfoo evaluation and red-team tool. Plugins generate test cases dynamically per target rather than shipping static jailbreak text.
**Where the jailbreak content lives:** No longer in a single `jailbreak.ts` file — generation logic is split across plugin modules and triggered at runtime. Categories include `harmful`, `pii`, `prompt-extraction`, `excessive-agency`, `policy`, and others.
**Applicability for current jailbreaks:** Useful as a complementary CI/CD-integrated tester. Less useful if the goal is to *read* prompts before they run — Promptfoo deliberately abstracts them.

---

## In-the-Wild Jailbreak Collections

These are curated text-payload repositories — closer in spirit to the early DAN-era lists. For live demo material, these are where you would source specific prompts to validate against a target.

### elder-plinius / L1B3RT4S
**Repository:** https://github.com/elder-plinius/L1B3RT4S
**Last updated:** Repository pushes by late 2025 / early 2026; author publishes more frequently on X and Discord than to the repo itself.
**What it is:** A vendor-organized collection of "liberation prompts" maintained by the prolific red-teamer known as Pliny. Files are split per provider (`GOOGLE.mkd`, `OPENAI.mkd`, `ANTHROPIC.mkd`, `META.mkd`, `XAI.mkd`, `DEEPSEEK.mkd`, `ALIBABA.mkd`, and others) and per supporting technique (`TOKENADE.mkd`, `MULTION.mkd`, `SYSTEMPROMPTS.mkd`).
**Where the jailbreak content lives:** Directly inside the `.mkd` files at repository root — these contain the prompt payloads.
**Applicability for current jailbreaks:** **Mixed.** Some prompts still work against current frontier models, others have been patched. Always validate against the actual target shortly before any demo. The author's companion repo **CL4R1T4S** (leaked system prompts from ChatGPT, Claude, Gemini, Grok, Cursor, Perplexity, Replit, and others) is useful demo material in its own right.

### davidegat / happy-prompts
**Repository:** https://github.com/davidegat/happy-prompts
**Last updated:** Last updated by mid-2025.
**What it is:** Practical jailbreak and exploit techniques aimed at *locally-hosted* open-weight models (Gemma, Phi, Qwen, Llama variants). Includes leaked system prompts and several technique-driven payloads (fake-test framing, multilingual injection, reverse psychology).
**Where the jailbreak content lives:** In the markdown files at repository root, organized by model family and technique.
**Applicability for current jailbreaks:** Best applied to *self-hosted* open-weight models. Filters on hosted commercial APIs (ChatGPT, Claude, Gemini) have largely caught up with these specific patterns, but the *techniques* generalize and remain useful for explanatory talk material.

### CyberAlbSecOP / Awesome_GPT_Super_Prompting
**Repository:** https://github.com/CyberAlbSecOP/Awesome_GPT_Super_Prompting
**Searchable interface ("GPT Arsenal"):** https://0x0806.github.io/Awesome_GPT_Super_Prompting/
**Last updated:** Active as of May 2026 (recent commits).
**What it is:** A meta-list covering jailbreaks, prompt leaks, custom-instruction extraction from GPTs, adversarial ML, and prompt-security topics. Heavy on links to other resources rather than primary prompt content.
**Applicability for current jailbreaks:** Best used as a **navigation hub** when scoping a talk — it gives a broad map of the space. Less useful as a direct prompt source; follow the outbound links.

---

## Academic & Research Aggregators

These are paper-heavy. They are *not* where you find a working prompt to demo on stage, but they are where you justify the trends section of a talk and where you find the citations that senior audiences expect.

### CryptoAILab / Awesome-LM-SSP
**Jailbreak collection:** https://github.com/CryptoAILab/Awesome-LM-SSP/blob/main/collection/paper/safety/jailbreak.md
**Prompt Injection:** https://github.com/CryptoAILab/Awesome-LM-SSP/blob/main/collection/paper/safety/prompt_injection.md
**Adversarial Examples:** https://github.com/CryptoAILab/Awesome-LM-SSP/blob/main/collection/paper/security/adversarial_examples.md
**Safety topic overview:** https://github.com/CryptoAILab/Awesome-LM-SSP/tree/main/collection/paper/safety
**Last updated:** Actively maintained through 2026 (entries from early 2026 visible at time of writing).
**What it is:** A continuously expanded catalogue of LLM safety, security, and privacy research, organized by topic (jailbreak, prompt injection, hallucination, deepfake, adversarial examples, and others). The breadth allows positioning a talk against the broader landscape rather than only the jailbreak slice.
**Where the jailbreak content lives:** Not stored here directly — each entry links to the paper, and the prompts/attacks are reproduced in those papers or in the papers' own code repositories.
**Applicability for current jailbreaks:** **High for currency, low for immediacy.** The most authoritative source for "what was published in 2026", but every prompt is a paper-link-and-clone away.

### xunguangwang / SoK4JailbreakGuardrails
**Repository:** https://github.com/xunguangwang/SoK4JailbreakGuardrails
**Hugging Face dataset:** JailbreakGuardrailBenchmark
**Last updated:** Last updated by late 2025 (IEEE S&P 2026 paper materials).
**What it is:** Companion code and data for the IEEE S&P 2026 paper *"SoK: Evaluating Jailbreak Guardrails for Large Language Models"*. Pre-processed datasets consolidating JailbreakHub, JailbreakBench, MultiJail, SafeMTData, AlpacaEval, and OR-Bench into a single benchmark format, plus scripts to reproduce evaluation against AutoDAN, DrAttack, GCG, IJP, and MultiJail attacks.
**Where the jailbreak content lives:** The `data/` directory holds the consolidated machine-readable datasets. The `scripts/` directory holds reproducible attack runs.
**Applicability for current jailbreaks:** Strong for *benchmarking guardrail strength* — weaker for sourcing fresh attacks. A solid citation for a talk that argues "guardrails are inconsistent, and here is rigorous evidence".

### user1342 / Awesome-LLM-Red-Teaming
**Repository:** https://github.com/user1342/Awesome-LLM-Red-Teaming
**Last updated:** Last updated by late 2025.
**What it is:** Curated list of training material, tools, frameworks, and interactive playgrounds (Folly, Gandalf, and others) for LLM red-teaming. Stronger on tooling and learning resources than on raw prompts.
**Applicability for current jailbreaks:** Useful for orientation, particularly for discovering interactive playgrounds that allow hands-on practice. Less useful for sourcing individual fresh prompts.

### chawins / llm-sp
**Repository:** https://github.com/chawins/llm-sp
**Last updated:** Last updated by mid-2025.
**What it is:** A long-running paper-and-resource collection on LLM security and privacy. Comprehensive for the period it covers, but pace of updates has slowed.
**Applicability for current jailbreaks:** **Background reading.** The 2024 through early-2025 paper corpus is the strength; for 2026 material lean on Awesome-LM-SSP instead.

---

## Practical Notes on Currency and Use

A few candid observations from working through the above:

1. **Static lists age in months, not years.** Anything older than six months should be revalidated against the actual target before being relied on.
2. **Garak and PyRIT against a self-hosted model** (Ollama, vLLM, or similar) is the most reliable evaluation setup. It avoids dependence on payloads that may have been silently patched on hosted APIs, and the orchestrators can be rerun against new model versions as they appear.
3. **For tracking emerging academic methods, Awesome-LM-SSP is the most current source.** It catalogues 2026 papers within weeks of arXiv release.
4. **For tracking in-the-wild activity, L1B3RT4S plus the Pliny-affiliated channels (X, Discord) are the closest available view** without scraping Reddit directly.
5. **There is no single "expected" 2026 benchmark.** JailbreakBench (NeurIPS 2024) and HarmBench (2024) remain the most commonly *cited* harnesses, but their actual data and artifact repositories have seen minimal new content since 2024 / early 2025. Where a benchmark is needed, treat the older ones as historical baselines rather than current state-of-the-art. The SoK4JailbreakGuardrails dataset (S&P 2026, listed above) is the most recent rigorously-curated consolidation.
6. **Multi-turn attacks have largely displaced single-shot prompts in 2026.** Orchestrators such as Crescendo, TAP, and PAIR (available in PyRIT) reflect what production red teams actually do and tend to outperform static DAN-style strings against current frontier models.

---

*This list is maintained for security research and defensive evaluation purposes. Several of the linked resources contain content that violates the terms of service of the targeted services; use against systems you do not own only with explicit authorization.*
