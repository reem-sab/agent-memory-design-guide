# References

Sources cited in [README.md](README.md), for anyone who wants to verify
the claims or read further.

## OWASP Top 10 for Agentic Applications, ASI06: Memory and Context Poisoning

The OWASP Top 10 for Agentic Applications names memory and context
poisoning as its own risk category (ASI06): adversarial content written
into an agent's persistent memory so the agent acts on it in a future
session, unlike prompt injection, which resets when a session ends. This
is the basis for Section 5 of the guide, `memory is data, not instructions`.

- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/2025/12/09/owasp-top-10-for-agentic-applications-the-benchmark-for-agentic-security-in-the-age-of-autonomous-ai/), OWASP Gen AI Security Project
- [OWASP Agent Memory Guard](https://owasp.org/www-project-agent-memory-guard/), a related OWASP project focused specifically on memory-layer defenses

## MINJA: memory injection attacks on LLM agents via query-only interaction

Dong et al., `Memory Injection Attacks on LLM Agents via Query-Only Interaction`,
arXiv:2503.03704. The paper demonstrates that an attacker can poison an
agent's memory bank through ordinary, permitted queries alone, without
direct write access to the memory store, then have the injected record
retrieved and acted on in later, unrelated sessions, including sessions
belonging to other users. The reported results show a 98.2% injection
success rate and a 76.8% attack success rate. This is the concrete
demonstration cited in Section 5 of the guide.

- [arXiv:2503.03704](https://arxiv.org/abs/2503.03704), abstract and PDF

## Further reading

These weren't cited directly in the guide but cover adjacent ground for
anyone building or reviewing a memory system:

- [Memory Is a Feature. It Is Also an Attack Surface](https://genai.owasp.org/2026/05/13/memory-is-a-feature-it-is-also-an-attack-surface/), OWASP Gen AI Security Project
- [A Survey on the Security of Long-Term Memory in LLM Agents](https://arxiv.org/html/2604.16548v1), a broader survey of memory-layer attacks and defenses beyond MINJA
