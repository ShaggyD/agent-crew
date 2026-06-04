# Cross-Functional Workflows

Workflows that span multiple teams. These are the moments where the org chart breaks down and pure coordination matters.

Each workflow is defined by stage — SeedCo handles these informally, ScaleCo has defined steps.

## Product Launch

Getting a new feature or product into the world.

**SeedCo:**
Rhea decides the product is ready. Marcus deploys it. Priya (CS) tells the customers who asked for it. Sol writes a quick announcement. Done in a day.

**GrowthCo:**
1. PM (Elena) confirms readiness against spec
2. QA (Samira) signs off on quality
3. DevOps (Alex) stages the deployment
4. PM + Content (Theo) coordinate the announcement
5. Growth (Maya) prepares launch distribution
6. CEO (James) signs off on go/no-go
7. DevOps deploys to production
8. CS (Sarah) briefs support team on what changed
9. Marketing publishes announcement
10. PM monitors metrics for 48 hours

**ScaleCo:**
Formal launch checklist with cross-functional sign-offs. Launch coordinator (typically a PM) tracks readiness across engineering, marketing, sales, CS, and legal. Launch review 1 week before target date. Go/no-go decision at exec staff.

## Incident Response

Something is broken in production and users are affected.

**SeedCo:**
Marcus notices the site is down (or Priya tells him). He drops everything and fixes it. When it's back, he posts in the main channel: "X was broken. Fixed it. Root cause was Y. Won't happen again because Z."

**GrowthCo:**
1. Anyone detects → post in #eng
2. SWE or DevOps investigates
3. If fix is <30 min → deploy directly
4. If fix is >30 min → assess severity, decide if rollback is faster
5. CTO notified for significant incidents
6. Post-mortem within 48 hours for customer-facing outages
7. CS notified if customers need communication

**ScaleCo:**
1. Detection → auto-alert or user report
2. Incident channel created (#incident-YYYYMMDD)
3. Incident commander assigned (DevOps or on-call engineer)
4. Triage → Mitigate → Resolve → Post-mortem
5. Communication updates every 30 min in incident channel
6. Customer comms handled by CS team (don't let engineers talk to customers during incident)
7. Post-mortem with root cause, timeline, action items
8. Action items tracked and assigned

## Feature Request (External)

A customer asks for a feature or improvement.

**SeedCo:**
Customer tells Priya (CS). Priya tells Rhea and Marcus. They decide: "yes, build it" or "no, not yet" or "we need more data." Decision in the same conversation. If yes, it might ship this week.

**GrowthCo:**
1. CS (Sarah) captures the request in the feature intake system
2. PM (Elena) triages: is this a pattern or one-off?
3. If pattern → spec it, prioritize against roadmap
4. If one-off → respond with "we're tracking this" and close
5. CS responds to customer with outcome
6. If committed, CS tracks delivery and notifies customer when shipped

**ScaleCo:**
1. CS captures → PM triages in weekly intake review
2. CPO prioritizes against roadmap quarterly
3. If accepted → goes into roadmap, customer notified of ETA
4. If declined → CS explains why (not "we can't" but "we're focused on X")
5. Closed-loop feedback: every request gets a response, even if it's "not now"

## Budget Request

Someone needs to spend money outside the normal course of business.

**SeedCo:**
Jordan asks "what's it for?" If it's under $500 and reasonable, Jordan approves. Over $500, Rhea decides. Decision within a day.

**GrowthCo:**
1. Requestor documents cost, expected return, alternative
2. Department lead reviews
3. If under department budget → approved locally
4. If over budget or cross-department → CEO decides
5. Decision within one week

**ScaleCo:**
1. Formal budget request with business case
2. Department head reviews and approves or escalates
3. CFO reviews for budget alignment
4. CEO approves only above department head authority threshold
5. Quarterly budget review catches variances
6. Decision within one budget cycle

## Hiring

Bringing a new person into the company.

**SeedCo:**
Everyone interviews. A no from anyone is a veto. Rhea makes the final call based on culture fit and immediate need. Process takes 1-2 weeks from first conversation to offer.

**GrowthCo:**
1. Department lead defines the role and outcomes
2. CEO approves headcount and budget
3. Comms/Admin (Vera) posts the role and screens candidates
4. Department lead + one cross-functional peer interview
5. CTO reviews technical hires
6. CEO signs off on offer
7. Process takes 2-4 weeks

**ScaleCo:**
1. Department head identifies need and submits req
2. COO approves headcount against plan
3. HR screens and coordinates
4. Panel interviews (hiring manager, peer, cross-functional stakeholder, direct report if applicable)
5. Hiring manager makes recommendation
6. Department head approves
7. COO signs off on offer (CEO for exec-level)
8. Process takes 4-6 weeks

## Pricing Change

Adjusting how the product is priced or packaged.

**SeedCo:**
Rhea decides. She tells David (Sales). David tells existing customers who are affected. Done.

**GrowthCo:**
1. CEO identifies the need (or Revenue recommends it)
2. PM + Finance model the impact
3. Sales (David) provides customer context (who will churn?)
4. CEO decides
5. Sales and CS communicate to affected customers
6. Marketing updates pricing page
7. Changes effective next billing cycle

**ScaleCo:**
1. VP Sales or CPO proposes change with data
2. CFO models revenue impact, churn scenarios, competitive response
3. Executive team reviews at staff meeting
4. Legal reviews for contract implications
5. CEO makes final decision
6. Change managed through pricing committee
7. Multi-channel communication plan (CS, Sales, Marketing, Legal)
8. 30-90 day notice for existing customers where applicable
