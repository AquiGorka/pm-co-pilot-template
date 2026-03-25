# Milestone Framework: PoC to Production

A progression framework for taking a product from proof of concept to market. Each milestone has a clear goal and exit criteria. Don't skip milestones — each one builds the foundation for the next.

## The Milestones

### 1. Proof of Concept

**Goal:** Build a tool that showcases that something can be done.

Validate the core idea. Can the fundamental approach work? Build the minimum to prove it — one happy path, end to end. No polish, no edge cases, no infra. Just: does this thing work?

**Exit criteria:**
- Core workflow runs end to end
- You can demo it to the team and stakeholders
- The approach is validated (or killed)

### 2. Release Infrastructure

**Goal:** Build the tools to test the main flow, observe it, and automate deploy/release/delivery so you can move faster.

Before building more features, invest in the machinery that lets you ship reliably. CI/CD pipelines, deploy automation, observability (tracing, metrics, error tracking), documentation, and environment management. This is the foundation that makes everything after it possible.

**Exit criteria:**
- Automated build, test, and deploy pipeline
- Observability in place (you can see what's happening in staging/production)
- Deploy strategy documented and working
- Core documentation written (architecture, key concepts)

### 3. MVP

**Goal:** Build all the tooling around the main flow in a technical approach. Dev teams use the product to understand the components.

Configuration over user experience. The product works, but it's built for people who understand the system — your team, technical evaluators, early technical partners. All the core components exist and can be operated, even if the UX is rough.

**Exit criteria:**
- All core features functional (may require configuration knowledge)
- Dev team can operate the full product
- Components are understood and documented
- Basic telemetry/monitoring active

### 4. Product

**Goal:** Productify the tooling. Make sure to highlight the value added to the end user(s)/persona(s). Your team should be doing QA in the shoes of end users.

Shift from "it works technically" to "it works for users." Every flow should communicate its value. UX goes from "good enough for devs" to "good for the target persona." Your team tests as end users, not as engineers.

**Exit criteria:**
- All user-facing flows highlight the value proposition
- QA done from end-user perspective, not dev perspective
- Persona-driven testing (does this solve their problem?)
- Feature-complete for the target audience

### 5. Beta

**Goal:** Open up the product to a select few for them to break the flows, find the issues, give feedback so you can iterate on the product — not the tech, the product.

Controlled exposure to real users outside your team. The goal is product feedback, not bug reports (though you'll get those too). Are the flows intuitive? Is the value clear? What's confusing? What's missing? Iterate on what users say, not what you assume.

**Exit criteria:**
- External users actively using the product
- Feedback collected and triaged
- Product iterations based on user feedback (not just bug fixes)
- Governance and error handling solid enough for external use

### 6. Hardening

**Goal:** Tech debt, fixes, and automations caught in the Beta.

Everything that surfaced during Beta that isn't a product decision — it's engineering work. Performance, scale, tech debt, automation gaps, reliability issues. Clean up the foundation so it can support what comes next.

**Exit criteria:**
- Tech debt from Beta addressed
- Performance and reliability at production quality
- Automation gaps closed
- Edge cases handled

### 7. GTM Experiment

**Goal:** Find a real person, company, or entity willing to use your product to solve their problems. Prove the product adds value.

Go beyond friendly beta users. Find someone with a real problem your product solves, and help them use it. This is validation that the product works in the wild, not just in controlled conditions.

**Exit criteria:**
- At least one real user/entity using the product for their own needs
- Evidence that the product solves their problem
- Learnings documented (what worked, what didn't, what surprised you)

### 8. GTM Value

**Goal:** Find a real person, company, or entity willing to PAY to use your product. Prove the product has traction — people find value and are willing to pay for it.

The ultimate validation. Someone will exchange money for what you built. This proves product-market fit, not just product-problem fit.

**Exit criteria:**
- At least one paying customer/user
- Pricing model validated
- Revenue path demonstrated
- Traction evidence for stakeholders/investors

---

## Using this framework

These milestones are a guide, not a mandate. Adapt them to your context:

- **Name them for your project.** "Beta" might be "Public Preview" or "Early Access" in your world.
- **Some milestones overlap.** Hardening work starts during Beta. GTM thinking starts during Product. That's fine — the milestone marks when it becomes the primary focus.
- **Track milestones in your task board.** Tag every task with its milestone. This gives you a natural view of progress and helps prioritize.
- **Not every project reaches every milestone.** A PoC might prove the idea doesn't work. That's a success — you learned cheaply.

### Coding thread prompts

When a milestone involves significant implementation work, the PM co-pilot can draft detailed coding thread prompts — comprehensive briefs that capture the full context (what exists, what's needed, architecture constraints, implementation order) so a separate coding thread can execute independently. This is optional but valuable for complex features that span multiple repos or components.
