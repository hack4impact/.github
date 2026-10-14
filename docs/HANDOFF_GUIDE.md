# Hack4Impact Cloud & Source Code Handoff Guide

When Hack4Impact builds software for a partner non-profit, writing good code is only half the battle. Delivering a system the partner can comfortably own, afford, run and maintain after our team steps away is what **makes the project actually succeed.**

Most partner organizations run with lean technical staff, tight budgets, and grant-dependent funding. This guide covers how H4I teams should structure projects from Day 1 so handoffs are smooth, affordable, and sustainable.

## Two Tracks, Two Different Homes

The single most important thing to understand upfront: the source code and the cloud infrastructure follow completely different ownership paths. Blurring these two together can lead into difficult handoffs.

|                             | Source Code                                                                                                              | Cloud Infrastructure                                                                                               |
| :-------------------------- | :----------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------- |
| **Where it lives**          | GitHub, always under [github.com/hack4impact](https://github.com/hack4impact)                                            | Can start in the H4I chapter's cloud account, but ultimately ends up fully in the partner's own cloud organization |
| **Who owns it long-term**   | Hack4Impact retains the repository; the partner non-profit can continue maintenance in H4I repository or fork if needed. | The partner non-profit owns it outright after handoff                                                              |
| **What happens at handoff** | The partner chooses to fork it, or to keep collaborating with H4I on the upstream repo                                   | The running system, billing, and admin access transfer completely to the partner                                   |
| **Why**                     | Lets future H4I teams maintain, revive, or learn from the project                                                        | The partner must be able to run their tool without depending on us                                                 |

Keep these tracks mentally separate throughout the project. The code is our shared, reusable asset. The live system and its data belong to the partner.

## Critical Pitfalls to Watch Out For

Before writing a line of code or spinning up a cloud console, keep these common failure modes in mind:

- **The Credit Cliff**: It's easy to build using credits that expire in 12 months. If your architecture relies on services with high fixed base fees, the partner gets stuck with an unsustainable bill once those credits dry up. Design for long-term baseline affordability, not just the initial grant runway.
- **Fragile Setup Scripts**: Loose terminal commands and unverified shell scripts tend to backfire. When a command fails midway, re-running it often crashes on resources that already exist. Aim for simple, modular Infrastructure as Code (IaC) or clean, repeatable steps anyone can re-run without fear of bad commands / lack of permissions mid-way leaving them stranded, etc.
- **Account & Identity Lock-Out**: If you build everything inside our chapter's cloud account without planning ahead, transferring ownership later often means tearing it all down and rebuilding. Keep tenant boundaries clear from the start.
- **The "Shadow Admin" Trap**: Never register domain names, DNS records, or third-party services (email providers, SMS gateways, payment processors, AI APIs) under a team member’s personal account or credit card. Everything should belong to a shared, partner owned organizational address (The organization this is built for, not H4I)
- **Secrets in Git History**: A leaked API key is not fixed by deleting the file in the next commit. Once it's in history, treat it as compromised. **More on this below.**

## **💻 - Source Code: Rules of the Road**

All project code lives under the [Hack4Impact GitHub organization](https://github.com/hack4impact) for the entire life of the project, and stays there afterward.

### 1\. Never Commit Secrets or Real Data. At Any Point

This one is absolute. No API keys, access tokens, service account files, connection strings, .env files, database dumps, CSV exports, or screenshots containing real people's information should ever be committed. Not "temporarily," not "just on my branch," not even "it's a private repo."

Practical habits that keep this from happening:

- Add .env, credential files, and data directories to .gitignore on the very first commit, before anything else.
- Check in a .env.example with empty placeholder values so teammates know what variables are needed.
- Keep real secrets in your cloud provider's secret manager, not in the repo.
- Turn on GitHub secret scanning and push protection for the repo.
- **Keep partner data out of GitHub Issues and pull request screenshots too, that's a surprisingly common leak.**

**If a secret does get committed:** rotate and revoke the credential immediately, then clean the history. Rotating is the part that actually matters; scrubbing history alone does not un-leak a key. This is where having IaC and easy to run templates help\!

### 2\. Decide Open-Source vs. Closed Early

Have an explicit conversation with your chapter leadership and the partner near the start of the project, not the week before handoff. This decision affects how you write documentation, what you can include in the README, and what license goes in the repo.

- **Closed / private, shared only with the partner,** is the right call when the codebase encodes sensitive operational logic, when the partner requests it, or when the domain is sensitive enough that public code could create risk for the people they serve.
- **\[Default but ensure above\] Open-source** where appropriate. It lets other chapters learn from and reuse the work without having to request access.
- Whatever you choose, add a LICENSE file, and make sure the partner has a clear written grant of rights to use, modify, and self-host the software. This should be part of your project agreement, not an afterthought.

### 3\. The Repository Stays With H4I

The repo remains under the H4I organization after the project ends. This is intentional: it lets a future chapter team pick the project back up for maintenance, a v2, or a similar build for another partner.

At handoff (Step 4), the partner picks one of two paths:

- **(Default, Preferred) Continue with H4I upstream**: The canonical repo stays with us, and a future H4I team handles maintenance or a follow-on project. Good when the partner has no engineering staff.
- **Fork and self-maintain**: They take a copy into their own GitHub organization and diverge from there. Good when they have in-house technical staff.

Either way, the H4I copy stays. If the project is truly finished with no maintenance planned, we can archive the repo rather than deleting it, and note its status in the README.

### 4\. Repo Hygiene Worth Doing

- A README.md, (optionally \+AGENTS.md) that explains what the tool does, who it's for, how to run it locally, and how it's deployed.
- Branch protection on main and code review before merge. This is also how you catch a secret before it lands.
- Automated dependency updates enabled, so the project doesn't rot into a pile of security advisories.
- CODEOWNERS or at least a documented point of contact should be valid and active at least two years from now, so an inbound issue when someone leaves or graduates reaches someone or preferably a group.

### 5\. Where the Two Tracks Touch: Deploy Credentials

CI/CD is the one place source code and cloud infrastructure meet, and it's easy to forget at handoff. If GitHub Actions deploys to the cloud environment, those deploy credentials live in GitHub secrets and point at a real cloud account.

At handoff, that pipeline must be re-pointed at the partner's environment with their own credentials, and the old chapter-era deploy credentials must be revoked. Prefer short-lived federated credentials (OIDC / workload identity) over long-lived static keys wherever the platform supports it.

## **☁️ - Cloud Infrastructure: Architecture, Sizing & Platform Choice**

Pick the architecture that does the job well. Be mindful to not introduce tech debt for the non-profit, H4I, or the chapter.

- **Favor Scale-to-Zero Compute**: Prefer managed container or serverless platforms that spin down to nothing when idle. If the partner's app gets no traffic over a holiday break, compute costs should be \~\$0. Avoid fleets of dedicated virtual machines or complex container clusters unless the workload genuinely demands them.
- **Be Intentional with Datastores and Review the Base Cost**: Open the pricing calculator before provisioning any database or storage engine. Avoid enterprise-tier managed databases that charge hundreds of dollars a month just to sit idle. Lean toward lightweight managed databases with minimal baseline configurations, burstable tiers, or serverless pay-per-request options.
- **Set Up Spending Alerts Early**: Configure billing alerts at sensible thresholds (50%, 75%, 90%, 100% of the expected monthly budget) so both the team and the partner's contact get warned well before a surprise. Where supported, wire up automation that can pause non-essential workloads if spending spikes.
- **Understand compliance constraints upfront**: ask early about legal and regulatory requirements, health information privacy, student records, customer-managed encryption keys, geographic data residency. Catching these early avoids fundamental re-architecture later.
- **Start Grant Applications Immediately**: Cloud non-profit programs (Google for Nonprofits, AWS Non-Profit Credit Program, Microsoft Tech for Social Impact) usually require verification through bodies like TechSoup, which can take 3–6 weeks. Help the partner kick this off on **Day 1**, not during handoff week.
- **Document every provisioning step**: keep a living record of how the infrastructure was built. Ideally this means clean configuration files (Terraform or OpenTofu). If steps must be manual or CLI-driven, write a sequential checklist in Markdown:
  1. Creating the base project or resource group
  2. Enabling the necessary platform APIs and service providers
  3. Configuring networking, access rules, and firewall boundaries
  4. Setting up databases and storage buckets
  5. Configuring secrets, environment variables, and service deployments

### Step 1: Development in H4I-Managed Projects

Allows more playgrounds and learning, but optional in some exceptions, **see last bullet point**.

- Spin up sandbox and development environments under your **H4I’s cloud organization**, backed by chapter billing or sandbox credits.
- **Hard rule on data**: under normal circumstances, never copy real partner records, user databases, or sensitive personal data into chapter environments. Use realistic synthetic or mocked test data.
- **Special case for data science and ML projects**: if the work fundamentally requires training models, running analytics, or validating pipelines against real partner data, **you should skip to step 3** and do the bulk of the development collaboratively inside the partner's own cloud organization from the beginning, so their data never leaves their institutional boundary. Only deviate from this with explicit partner approval and confirmed compliance coverage.

### Step 2: Partner Staging & Migration Testing (If Applicable)

**Greenfield applications**: if the partner is starting fresh with no existing data to migrate, skip this step and go to Step 3\.

If legacy databases or existing data, files, databases need to move over:

1. Create a temporary staging environment **inside the partner's cloud organization**, on their billing or credits.
2. Validate compliance, encryption policies, and team permissions in their actual environment.
3. Test migration scripts and ETL pipelines against representative sanitized copies of their data, to catch format mismatches or capacity bottlenecks early.
4. **Delete the temporary staging project** once migration validation is done, so the partner isn't paying for redundant infrastructure.

### Step 3: Joint Handoff & Access Transition

Handoff is not an email with credentials and a repo link. It's a live, collaborative working session between H4I developers and the partner's technical leads.

1. **Build in their clean production account**: have partner staff log into their official cloud environment.
2. **Walk through the setup together**: step through your documentation and deployment configs side by side, provisioning production from scratch. This is the best way to catch missing steps, answer maintenance questions, and confirm the partner genuinely understands how the system fits together. Fix documentation gaps live, in the session.
3. **Settle the source code path**: confirm whether the partner is forking the repo into their own GitHub organization or continuing with H4I as upstream. Document the decision in the README.
4. **Transfer third-party ownership**: hand over control of domain names, DNS, and external service integrations (API keys, mail servers, notification providers) to role-based team addresses like admin@partner.org or a managed group, preferably never a personal inbox unless there is no other choice.
5. **Re-point and rotate deploy credentials**: update any CI/CD to target their environment and revoke the chapter's old deploy keys.
6. **Pick an access posture**:
   - **Zero retained access (default)**: all H4I access is removed at wrap-up. The partner holds full, exclusive control. The team should advise the non-profit on steps needed to re-add the team in cases of emergency / needing help.
   - **With temp provisioned break-glass**: a named, time-limited role for H4I team with an expiration (30–60 days) to help debug immediate post-launch issues.
7. **Leave them the keys to the kingdom**: documentation showing partner admins how to rotate secrets, review access logs, add or remove users, and check billing.

## Practical Cloud Platform Notes

The core concepts are similar across providers, but each has nuances worth knowing.

### Google Cloud Platform (GCP)

- **Compute**: managed container platforms like Cloud Run are ideal as they scale to zero when not serving requests.
- **Databases**: right-sized managed relational instances or lightweight document stores. Skip high-tier enterprise databases unless scale genuinely demands them.
- **Organization hierarchy**: make sure projects sit under the partner's Google Cloud Organization (tied to their Google Workspace for Nonprofits domain), not a standalone personal developer account.
- **Billing safety**: Cloud Billing budgets with alerts routed to a distribution list, not one person.

### Amazon Web Services (AWS)

- **Compute**: managed container runtimes (App Runner, serverless container configs) that minimize manual server administration.
- **Databases**: keep clusters on modest resource allocations, or use on-demand serverless models where idle cost is negligible.
- **Identity and tenancy**: set up accounts under AWS Organizations with IAM Identity Center for team access. Never share or deploy from the root account. Instead, lock it behind strong MFA and delegate day-to-day work to role-based accounts.
- **Billing safety**: AWS Budgets with notifications to the partner's technical contact.

### Microsoft Azure

- **Compute**: serverless container environments like Azure Container Apps on consumption plans give clean scale-to-zero behavior.
- **Databases**: burstable tiers designed for intermittent traffic, rather than dedicated multi-core instances.
- **Identity and tenancy**: resources should live in the partner's Microsoft Entra ID tenant, typically set up through Microsoft's non-profit grant tiers.
- **Billing safety**: Azure Cost Management budgets and action alerts to track monthly grant burn.

## **Minimal Handoff Package Checklist**

- [ ] Infrastructure is fully transferred to the partner's cloud organization.
- [ ] Project source code is in the Hack4Impact GitHub organization.
- [ ] A maintenance plan (continue upstream or self-maintain) has been agreed upon with the partner.
- [ ] All infrastructure provisioning steps are documented (e.g., via Infrastructure as Code or sequential checklists).
- [ ] Cloud billing alerts are configured with the partner's contact information.
- [ ] Third-party service ownership (domains, APIs) is transferred to the partner.
- [ ] The minimal handoff package (Runbook, Cost/Budget Guardrails, Secrets Inventory) is completed. Every handed-off project repository must include these core deliverables:
  - [ ] **Infrastructure Blueprint:** Modular configurations (e.g., Terraform) or clean scripts that provision the stack reliably and idempotently.
  - [ ] **Runbook / Operations Guide:** Plain-language instructions: how to deploy new updates, check system logs, inspect errors, restart services, and restore backups. This should ideally be easy to follow for non-technical partners in any area possible.
  - [ ] **Training / Data Files:** AI/ML data files, training files, eval datasets, etc. so the partner can replicate the work later on if needed.
  - [ ] **Secrets & Access Inventory:** Every third-party service, domain, and credential the system depends on, who owns each one, and how to rotate it.
  - [ ] **Infra / Cost & Budget Guardrail Management:** A clear summary of expected monthly costs, links to active non-profit grant dashboards, and steps to update budget alert recipients.
  - [ ] **Disaster Recovery & Cleanup:** Clear steps to export data, restore from backups if an outage occurs, and decommission services cleanly if the tool is ever retired.
