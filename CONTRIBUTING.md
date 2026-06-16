# Contributing to the Protocol Almanac

How to add protocols, fix data, challenge rankings, and improve the methodology.

## Table of Contents

1. [Ways to Contribute](#ways-to-contribute)
2. [Adding a New Protocol](#adding-a-new-protocol)
3. [Fixing Data](#fixing-data)
4. [Challenging a Ranking](#challenging-a-ranking)
5. [Improving the Methodology](#improving-the-methodology)
6. [Code of Conduct](#code-of-conduct)
7. [License](#license)

---

## Ways to Contribute

You can contribute to the almanac in several ways:

| Contribution Type | What you do | Impact |
|-------------------|-------------|--------|
| **Add a protocol** | File an issue with a new protocol or SDK | Expands the roster |
| **Fix data** | Correct incorrect metadata or spec details | Improves accuracy |
| **Challenge a ranking** | Provide evidence that a score is wrong | Drives quality |
| **Share experience** | Write about integrating a protocol in production | Adds real-world context |
| **Improve methodology** | Propose a better conformance test or scoring rubric | Improves fairness |
| **Build an adapter** | Implement the adapter for a new protocol | Enables testing |
| **Review an edition** | Proofread, fact-check, suggest improvements | Improves quality |
| **Document spec ambiguity** | Identify and explain ambiguous spec behavior | Helps the community |
| **Spread the word** | Share the almanac with your community | Grows the ecosystem |

## Adding a New Protocol

### Before you submit

Check if the protocol meets the triage criteria:

1. **Seriousness**: Is it a real protocol with a real spec, real implementation(s), or real adoption? Not a toy or demo.
2. **Activity**: Has it had a spec update, SDK release, or implementation push in the last 6 months?
3. **Documentation**: Does it have a spec, README, or landing page explaining what it does?
4. **Accessibility**: Is it testable (open spec, free SDK, or evaluation license)?
5. **Scope**: Does it fit the category definition (AI protocols, standards, authentication, or integration frameworks)?

### How to submit

**Option 1: GitHub Issue (preferred)**

File an issue with this template:

```markdown
## Protocol Request: [Protocol Name]

### Standard Category
[Which standard category? e.g., MCP, A2A, Authentication, Payments]

### Protocol URL
[Spec URL or GitHub repo URL]

### License
[e.g., MIT, Apache-2.0, Proprietary, CC-BY-4.0 (for specs)]

### Governance
[e.g., Vendor (Anthropic), IETF, Open Community, Consortium]

### Transport
[e.g., WebSocket, SSE, HTTP, gRPC, Multiple]

### Description
[What does it do? One paragraph. What problem does it solve?]

### Why it should be on the roster
[Evidence of adoption, production usage, implementations, or technical merit.]

### Evidence
- GitHub stars (if applicable): [N]
- Known implementations: [list of SDKs, servers, or frameworks]
- Last release/spec update: [date]
- Notable users: [companies, if known]
- Standards-body status: [IETF draft, W3C working group, etc.]

### Tier suggestion
[A, B, or C — and why]
```

**Option 2: Pull Request**

If you want to add the protocol directly:

1. Fork the repo
2. Edit `data/roster.json` to add the protocol
3. Update the relevant `standards/<standard-key>/README.md` if the protocol is Tier A
4. Update `README.md` if the protocol is Tier A
5. Submit a PR with the same template as above

### What happens after submission

1. **Triage**: We check if the protocol meets criteria (within 7 days)
2. **Smoke gate**: We run the protocol through the 3-turn scenario (initialize, exchange, verify) (within 14 days)
3. **Decision**: Accepted, rejected, or deferred with a note
4. **Publication**: If accepted, it appears in the next edition

## Fixing Data

### If you find incorrect metadata

File an issue with:

```markdown
## Data Correction: [Protocol Name]

### Current (incorrect) data
[What does the roster say?]

### Correct data
[What should it say?]

### Evidence
[Link to the source that proves the correct data: spec page, GitHub repo, release notes.]
```

### Common corrections

| Field | Common errors | How to verify |
|-------|--------------|---------------|
| License | Wrong SPDX identifier | Check the repo's LICENSE file or spec license notice |
| Stars | Out of date | Check the GitHub API |
| Last push | Wrong date | Check the GitHub repo or spec revision history |
| Tier | Wrong tier | Check the tier rules in IMPLEMENTATION.md |
| Notes | Outdated description | Check the protocol's homepage or spec |
| Transport | Wrong or missing transport | Check the spec's transport section |
| Governance | Wrong governance model | Check the spec's governance notice or standards-body membership |

### Spec ambiguity corrections

If you discover that a spec is ambiguous or has been clarified:

```markdown
## Spec Ambiguity: [Protocol Name]

### Ambiguous behavior
[What is unclear or contradictory in the spec?]

### Proposed interpretation
[How should the adapter behave?]

### Evidence
[Link to spec section, issue discussion, or authoritative clarification.]
```

### What happens after submission

Data corrections are reviewed and applied in the next edition cycle. We don't edit editions retroactively; we correct the data and note it in the next edition.

## Challenging a Ranking

### If you believe a score is wrong

File an issue with:

```markdown
## Challenge: [Protocol Name] on [Dimension]

### Current score
[What does the almanac say?]

### Your evidence
[What data do you have?]

### What you did to verify
[Steps you took to reproduce or verify.]

### Suggested resolution
[What should change? Re-run? Different score? Methodology update?]
```

### What evidence is valid

| Evidence Type | Strength | Example |
|---------------|----------|---------|
| Raw results JSON analysis | Strong | "I re-analyzed the JSON and found that schema validation incorrectly failed on optional fields" |
| Independent reproduction | Strong | "I ran the harness and got 98% compliance, not 87%" |
| Documentation of a spec bug | Medium | "The protocol has a known spec ambiguity that affects this test" |
| Vendor claim | Weak | "The vendor says Z" — but we already test vendor claims |
| Anecdote | Weak | "It worked for me" — not reproducible |

### What happens after submission

1. **Review**: We review the evidence (within 7 days)
2. **Reproduction**: If the claim is reproducible, we re-run the test
3. **Update**: If the re-run confirms the challenge, we update the score
4. **Publication**: The update appears in the next edition

## Improving the Methodology

### If you want to propose a methodology change

File an issue with:

```markdown
## Methodology Proposal: [Title]

### Current state
[What does the methodology say now?]

### Proposed change
[What should it say?]

### Rationale
[Why is this better? What problem does it solve?]

### Impact
[Which protocols/standards would be affected?]

### Backward compatibility
[Can old results be re-scored with the new method?]
```

### Methodology change process

1. **RFC**: The proposal is posted as an RFC for public comment (30 days)
2. **Discussion**: Community feedback is collected
3. **Decision**: ArdurAI makes the final decision based on feedback
4. **Announcement**: If accepted, a public announcement is made with a transition plan
5. **Implementation**: The change is implemented in the next edition cycle
6. **Re-run**: Affected benchmarks are re-run with the new methodology

### What kinds of changes are accepted

| Change Type | Likelihood | Example |
|-------------|------------|---------|
| Bug fix in harness | High | "The adapter incorrectly handles optional schema fields" |
| New conformance test | Medium | "Add a test for MCP `sampling/createMessage` compliance" |
| New stress test | Medium | "Add a federation broadcast test for 500+ agents" |
| Weight adjustment | Medium | "Increase ops burden weight from 15% to 20%" |
| New dimension | Low | "Add a 'security posture' dimension" |
| Remove dimension | Very low | "Remove latency as a dimension" |

### What kinds of changes are rejected

- Changes that favor a specific vendor or protocol
- Changes that reduce reproducibility
- Changes that increase complexity without clear benefit
- Changes that are not backward-compatible without a migration plan
- Changes that reward spec ambiguity instead of spec clarity

## Code of Conduct

### Be respectful

This is a collaborative project. Treat others with respect, even when you disagree about protocol design or spec interpretation.

### Be evidence-based

Claims should be backed by evidence. "I think X is better" is not enough. "I measured X and found Y, and here's the message trace" is.

### Be constructive

Criticism is welcome if it's constructive. "This protocol is wrong" is not helpful. "This protocol is non-compliant on section 4.2 because of Z, and here's the message trace" is.

### Be patient

The almanac is maintained by a small team. Responses may take time. Repeated pings are not helpful.

### No spam

Don't submit the same protocol multiple times. Don't submit protocols that clearly don't meet criteria. Don't use the almanac for marketing your protocol or SDK.

## License

By contributing to the almanac, you agree that your contributions are licensed under CC BY 4.0 for content and MIT for code.

## Attribution

Contributors are recognized in the edition notes. If you make a significant contribution (e.g., adding 5+ protocols, fixing major data issues, improving methodology, documenting spec ambiguity), you will be listed as a contributor in the next edition.

## License

Content: CC BY 4.0  
Code: MIT
