# Llama 3.1 Community License — key terms for downstream use

source_url: https://www.llama.com/llama3_1/license/
fetched: 2026-05-07

## Headline

Royalty-free, worldwide, non-exclusive license to **use, reproduce, distribute, copy, create derivative works of, and modify** the Llama Materials. Commercial use is **permitted** by default.

## Hard restrictions

1. **700M MAU clause.** If on the release date of the relevant Llama version your products/services (incl. affiliates) had >700M monthly active users in the prior month, you must request a separate license from Meta. Meta may grant or deny "in its sole discretion." Threshold is per-corporate-group, not per-feature.
2. **Acceptable Use Policy.** Mandatory compliance with Meta's AUP — bans use for illegal activity, violence, hate, weapons, surveillance/biometrics violations, infrastructure attacks, regulated professional advice (legal, financial, medical) without disclosure, etc.
3. **Attribution.** Must include "Built with Llama" in derivative product UI/marketing; must redistribute the LICENSE file with any redistribution.
4. **Naming convention.** Derivative model names must include "Llama" at the start.
5. **Trademark.** No use of Meta marks except as required for attribution.

## Implications for tool-calling specialists

- Hermes-3 (all sizes), Hermes-4-70B, Hermes-4-405B → Llama 3.1 derivatives, license = **commercial-OK with the above restrictions**.
- Llama-xLAM-2-8b-fc-r → Llama 3.1 8B derivative, **but** Salesforce stacked **CC-BY-NC-4.0** on top → effectively non-commercial.
- Functionary v3.x → Llama 3.1 8B derivative under MIT (MeetKai's added license is permissive), so it inherits Llama Community License terms but is otherwise commercial-OK.
