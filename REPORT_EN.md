# From customer reviews to specific, checkable insights

Review documentation with figures and complete examples below. [Русский разбор](README.md). This repository contains documentation only; the code submission is packaged separately.

This prototype turns 5,000 banking reviews into specific observations, groups related observations, and links them to the supplied themes. Each result can be traced back to the customer's words.

The full saved-data pipeline runs locally without a model API key. It produces **7,310 aspects and 3,834 groups**. All candidate groups have model review coverage. The final theme mapper is a **lexical baseline**, not a completed LLM mapping experiment. This is an inspectable prototype, not a validated production system.

## 1. The task

The [assignment](https://github.com/chattermill/llm-challenge/blob/8ab5811999692cac9e9de238cd8061e8b7c27d83/README.md) asks for LLM aspect extraction with sentiment, coherent insights, hierarchical theme mapping, and evaluation. The reviews have no correct labels to train or test against.

This is not just a 15-class classifier. A review can praise interest rates, complain about login, and describe an older version that worked well.

| Term | Meaning in this solution |
|---|---|
| Review | One complete customer comment |
| Aspect | One specific observation and the sentiment toward it |
| Group or insight | A description supported by its member aspects; it can be recurring, a single case, or general feedback |
| Theme | One of the 15 supplied broad topics |
| Category | The parent of a theme, such as Online Experience |

An insight is not automatically a proven recurring problem. Positive feedback and single cases are retained too.

## 2. The approach

![The complete pipeline](assets/pipeline.png)

**Extract before assigning themes.** Browser ChatGPT Pro reads each review and returns specific aspects. Themes are not mandatory extraction classes. Clause discovery is followed by whole-review consolidation, so three consequences of one event do not have to become three separate aspects.

Each aspect has a description, sentiment, a literal quote, and three context fields: what product it refers to, when it happened, and whose experience it describes. The description and sentiment come from the task; evidence and context are my design choices. Context labels are model predictions, not verified facts.

**Validate the saved answers.** Code checks IDs, allowed values, duplicates and literal quotes. An unprocessed review is never replaced with an empty result. IDs are zero-based source CSV rows. Six working texts were privacy-redacted; quotations are checked against the bundled texts.

**Propose groups, then inspect their meaning.** E5-small-v2 turns aspect descriptions into vectors. Complete-linkage clustering requires even the most distant pair in a proposed cluster to be within cosine distance 0.15. This limits chains of loosely related observations. Explicit context separates incompatible referents and times. The threshold is an engineering choice, not a measured optimum.

Browser Pro reviewed initial candidates. A saved, budgeted Sonnet API pass completed the rest, reading full reviews and splitting different situations. Every extracted aspect remains in exactly one group, including 16 rejected extraction hypotheses retained for inspection. Rejected entries are excluded from theme mapping. Different original candidates are not consolidated afterward, so similar insights can remain fragmented.

**Map themes last.** The submitted mapper applies explicit word patterns to aspect descriptions. It records which aspects and reviews support each theme. Multiple themes, or no theme, are allowed. Non-target observations do not contribute to target-theme counts. Category counts use the union of review IDs, not the sum of theme counts.

Two supporting aspects from one review count once. Reviews are not people. The corpus includes different banks and times, so pooled counts are not prevalence for one bank.

## 3. Experiments and decisions

These experiments informed the design. None establishes an accuracy winner without independent labels.

| Question | Test | Observation | Decision |
|---|---|---|---|
| Do visible themes reduce detail? | Same 50 reviews: themes visible, hidden, and hidden with clause discovery | 134, 141 and 146 aspects; inspection found missing detail and over-splitting | Extract without mandatory themes, then consolidate within the review. More aspects do not prove better recall |
| Can theme-led search help exploration? | BM25 keyword search, E5 semantic search and RRF rank fusion, top 20 per theme | BM25 and E5 shared 0–8 results per theme, averaging 3.6 | Use search for diagnostics; the main pipeline processes every review |
| How aggressively should aspects be merged? | Complete linkage at 0.15, 0.20 and 0.25 on 146 pilot aspects | 84, 42 and 12 groups; larger distances produced broader groups | Use the conservative threshold plus context and review, accepting fragmentation |
| Should the nearest theme vector decide the label? | Dense nearest-theme baseline | Account Access reached 4,031 unique reviews, including unrelated guidance praise | Reject that configuration after inspecting the errors; use an explicit lexical fallback |
| Can corpus exploration reveal useful scenarios? | Separate Pro analysis with quoted examples | 12 candidate patterns, such as recovery help inside an inaccessible app | Treat these as hypotheses, not exhaustive membership or measured prevalence |

BM25 finds matching words. E5 finds vector similarity. RRF combines the ranks of two lists. Retrieval is not clustering, and top-20 results do not establish how common a problem is.

The strong claim that visible themes suppress unusual aspects was **not established**. The pilot supported a practical design choice, not a measured causal conclusion. It used 30 pilot, 10 short and 10 long diagnostic reviews, not a representative benchmark. Saved pilot answers and diagnostic summaries are included.

A compact LLM theme mapper was implemented and schema-tested. Full inference was **not run** because the challenge proxy reported exhausted credit during preflight. It is not the submitted method.

## 4. Five examples

![A mixed review produces separate observations](assets/mixed-review.png)

Review 2617 praises the old app, then reports that the new one signs the user out immediately. Old-app praise is positive and past. Logout is negative and current. Only the latter joins review 1013's current logout complaint.

![Similar wording still needs context](assets/grouping.png)

All members of these five groups were checked: ten unique reviews and ten member aspects. These are selected assistant diagnostics, not an independent evaluation sample. [Complete reviews, members and actual theme outputs](docs/EXAMPLES.md) are included.

| Group and review IDs | Supported observation | Actual mapping and caveat |
|---|---|---|
| F00005: 2879, 3259 | Instructions and tips help reviewers use the app | Navigation & Design, supported by both |
| F00016: 1001, 1668 | Credentials believed correct are rejected, followed by setup or reset | Account Access for both. Credential correctness is not technically established |
| F00029: 4527, 4688 | Good rates remain positive inside critical reviews | Pricing & Fees for both, Core Banking Features only for 4688: incomplete savings-interest support |
| F00055: 1013, 2617 | Immediate logout after moving to the new app | Access, performance and version links for both. Incidental transaction and balance words also trigger broad payment and banking links for 1013 |
| F00063: 679, 1017 | Mobile access removes the need for a laptop | Core Banking Features only for 679. The differently worded 1017 observation is missed; its privacy detail must not be generalized to 679 |

One final wording correction narrowed API099_009: both reviewers criticize a €25 express delivery fee, but only one says it was unrequested. The model answer and separate correction log are preserved. This assistant review is not human gold.

Review 4242 also exposes an extraction omission: praise for keeping track of finances was not retained separately. Group review cannot recover an aspect that was never extracted. This remains a known error, not measured recall.

## 5. Evaluation and limitations

| Result | What it means |
|---|---|
| 5,000 valid saved review outputs; no missing IDs | Complete processing coverage, not extraction of every thought |
| 7,310 aspects; 31 explicit empty reviews | Actual output sizes; empty rows are answers, not failed requests |
| 3,026 candidates; 3,834 final groups; none awaiting review | Complete model review coverage, not proven group correctness |
| Each aspect in exactly one group | No membership loss or duplication |
| 16 rejected hypotheses retained | Inspectable exclusions, not mapped themes |
| 1,412 groups without a theme | Baseline abstentions, including rejected entries; not necessarily new topics |
| 137 automated tests | Code and invariant checks, not semantic accuracy |

Sentiments are 4,257 positive, 2,687 negative and 313 neutral. Another 28 are flagged mixed and 25 uncertain. These ambiguity states are preserved rather than silently treated as neutral; they need adjudication for a strict three-label evaluation.

Checks cover literal evidence, hierarchy, partition membership, supported counts and deterministic replay. Selected examples reveal extraction, context, grouping and mapping errors. **Human reference annotation was not performed**, so precision, recall, F1 and semantic accuracy are not reported. Model agreement does not replace that reference.

The main weaknesses are omissions, incorrect context predictions, fragmented or broad groups, and lexical mapping errors. Next I would annotate a small independent sample, evaluate extraction, sentiment, group coherence and mapping separately, then calibrate mapping and consolidate duplicate insights. I would not start with a larger platform.

## 6. Run the full saved-data pipeline

Python 3.12 was tested on macOS ARM64. From the unpacked archive:

~~~bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -c requirements.lock.txt '.[test]'
.venv/bin/chattermill run --mode replay --complete --out outputs/reproduced
.venv/bin/python -m pytest -q
~~~

Replay needs no model API key, embedding download or sibling project. It rebuilds extraction, groups, mappings and counts from saved answers. Dependency installation may need internet access.

**Replay is not fresh LLM generation.** Extraction was performed in browser Pro. Prompts and saved answers are provided, but a one-command fresh extraction API stage is not implemented. [Optional commands and artifact locations](docs/RUNNING.md) explain the boundary.

The ZIP contains code, tests, pinned constraints, working reviews, taxonomy, prompts, saved model answers, experiment evidence, all stage outputs and evaluation notes. The manifest lists file hashes. Private delivery notes and recruiter correspondence are excluded.

### Budget

The proxy reported **$20.00 used against its $20.00 cap**. Completion usage estimates total $15.05, or $15.43 including an unresolved reservation. The discrepancy is unresolved: the local ledger is not a provider invoice, and token-count requests were not proven free. The API funded group review and mapping pilots or preflight, not a completed full LLM mapping pass. Further paid calls were stopped. No personal credential was used or bundled. Browser subscription compute is separate from the challenge credit.

## Appendix: full example reviews

## Five examples, checked against the saved outputs

This is a selected assistant review, not a random sample or independent human gold.
Each case contains all group members and their complete bundled review texts.
Suggested interpretations below are not silently substituted for the actual theme output.

### Helpful guidance — F00005

Readable instructions or helpful tips help reviewers understand and use the app.

**Support:** 2 unique reviews: 2879, 3259.

Both authors praise guidance that makes the app easier to use.

**Complete reviews and extracted members**

#### Review 2879

> So helpful. Really great app. Easy to use and easy to read instructions if you are a bit lost.

- 2879:0: Easy app with readable helpful instructions when needed — positive, current, target
  Evidence: So helpful. Really great app. Easy to use and easy to read instructions if you are a bit lost.

#### Review 3259

> Really easy to understand. Helpful tips and no phaf

- 3259:0: Easy-to-understand app with helpful tips and little friction — positive, current, target
  Evidence: Really easy to understand. Helpful tips and no phaf

**Actual theme output**

| Category | Theme | Supporting reviews |
|---|---|---|
| Online Experience | Navigation & Design | 2879, 3259 |

**What not to overclaim:** Do not expand this into a claim that neither reviewer had any problems.

### Credentials believed to be correct — F00016

Reviewers report rejection of credentials they believe are correct, requiring account setup or PIN reset; the correctness claim is theirs, not technically verified.

**Support:** 2 unique reviews: 1001, 1668.

Both report rejected credentials followed by setup or reset.

**Complete reviews and extracted members**

#### Review 1001

> Not a very good app, I know I pressed the right code to login, after 5 attempts I have to set up my account again.

- 1001:0: Reportedly correct login code rejected until five attempts force account setup again — negative, current, target
  Evidence: Not a very good app, I know I pressed the right code to login, after 5 attempts I have to set up my account again.

#### Review 1668

> Good when it works. But most of the time it don't let you log in, says your pin is incorrect, when I know 100 per cent it is. Then have to reset it. On main website. For it just to keep doing it over and over

- 1668:0: Repeated rejection of reportedly correct PIN forces website resets without lasting fix — negative, current, target
  Evidence: Good when it works. But most of the time it don't let you log in, says your pin is incorrect, when I know 100 per cent it is. Then have to reset it. On main website. For it just to keep doing it over and over

**Actual theme output**

| Category | Theme | Supporting reviews |
|---|---|---|
| Account Management | Account Access | 1001, 1668 |

**What not to overclaim:** Their belief is evidence of their experience, not proof that the credentials were valid.

### Good rates inside critical reviews — F00029

Reviewers praise savings/account interest rates while separately criticizing aspects of account service or the app.

**Support:** 2 unique reviews: 4527, 4688.

Praise for rates remains positive even when the same review contains complaints.

**Complete reviews and extracted members**

#### Review 4527

> Good rates but there are a number of things that annoy me that mean I will switch when my account matures...
> 1. App stops me logging in until I update whenever there is a new version available 
> 2. Constantly nags me to switch on notifications, I don't want notification and NEVER will so this is a terrible user experience
> 3. Atom do not provide a yearly tax statement - life is too short to trawl through interest transactions to work it out, especially when you've had more than one account in a tax year (and you are doing it on a phone screen!)
> 4. The document vault is a complete mess - it's almost impossible to work out what each document is for, they have meaningless titles, do not show which product they are for and don't include dates
> 5. The app is slow, you constantly get the spinning circle when moving between different screens

- 4527:0: Good rates — positive, current, target
  Evidence: Good rates

#### Review 4688

> Opened the account without an issue but despite choosing monthly interest it was added to my account.  Maybe I should have seen the options but it certainly was not clear.   However, what was very disappointing is that when chatting with them online, they would not pay that first interest installment to my bank.  You would think, in good spirit, they would be able to put this right considering today is  the day the payment is due.    Very poor service in my opinion.  The rates are good and we were going to open another account but with no help or flexibility to correct a simple matter we will go elsewhere.  Poor

- 4688:3: Good interest rates — positive, current, target
  Evidence: The rates are good

**Actual theme output**

| Category | Theme | Supporting reviews |
|---|---|---|
| Product & Features | Core Banking Features | 4688 |
| Company & Brand | Pricing & Fees | 4527, 4688 |

**What not to overclaim:** The lexical mapper assigns Pricing & Fees to both members, but Core Banking Features only to 4688. This is incomplete support for the savings-interest interpretation.

### Immediate logout in the new app — F00055

After moving from the old app to the new one, immediate logout after login/account viewing prevents further account tasks.

**Support:** 2 unique reviews: 1013, 2617.

Two current negative observations describe immediate logout. Old-app praise is separate.

**Complete reviews and extracted members**

#### Review 1013

> Does not seem to work at all, I can't see any transactions or anything except the balance. Keeps telling me my session has expired and I've been logged out after I've just logged in! Wish I could still use the old app, never had any problems with it.

- 1013:0: Immediate session-expiry logouts prevent viewing transactions beyond balance in new app — negative, current, target
  Evidence: Does not seem to work at all, I can't see any transactions or anything except the balance. Keeps telling me my session has expired and I've been logged out after I've just logged in! Wish I could still use the old app, never had any problems with it.

#### Review 2617

> I have been using the old app for a while now with no issues. Having been advised that the old app will no longer be valid  I have downloaded the new version. Once I log on and check my account details it immediately signs me out before I can do anything. No use whatsoever !

- 2617:1: New app immediately signs out after viewing account details, preventing tasks — negative, current, target
  Evidence: Once I log on and check my account details it immediately signs me out before I can do anything.

**Actual theme output**

| Category | Theme | Supporting reviews |
|---|---|---|
| Account Management | Account Access | 1013, 2617 |
| Online Experience | App Performance | 1013, 2617 |
| Account Management | Cards & Payments | 1013 |
| Product & Features | Core Banking Features | 1013 |
| Online Experience | Updates & Versions | 1013, 2617 |

**What not to overclaim:** The mapper adds Cards & Payments and Core Banking Features to 1013 because its label mentions transactions and balance. Those extra links are broad and need adjudication.

### Mobile banking replaces a laptop — F00063

Mobile account/finance access removes the reviewer’s need to use a laptop for online banking.

**Support:** 2 unique reviews: 679, 1017.

Both authors say mobile access removes the need to use a laptop.

**Complete reviews and extracted members**

#### Review 679

> Does everything I need and more. Brilliant app keeps me on top of my finances without have to use my laptop and it's great to use if you need to contact barclays.

- 679:0: Mobile finance management removes need for a laptop — positive, current, target
  Evidence: Brilliant app keeps me on top of my finances without have to use my laptop

#### Review 1017

> Very handy and becomes very personalised as compared to big screen on laptop where one can easily read all your detail.... 25/01 I have not used my laptop for my online account..Its very good app

- 1017:0: Convenient personal mobile access replaces laptop and feels less exposed to others — positive, current, target
  Evidence: Very handy and becomes very personalised as compared to big screen on laptop where one can easily read all your detail.... 25/01 I have not used my laptop for my online account..Its very good app

**Actual theme output**

| Category | Theme | Supporting reviews |
|---|---|---|
| Product & Features | Core Banking Features | 679 |

**What not to overclaim:** Only 1017 mentions feeling less exposed on a smaller screen. The mapper misses 1017 for Core Banking Features, illustrating a paraphrase gap.
