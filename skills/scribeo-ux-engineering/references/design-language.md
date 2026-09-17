# Working With the Design Language

Load when applying an established visual direction to structure, or when diagnosing a result that reads as generic.

> **Scope.** Deriving the visual direction — palette, typeface selection, aesthetic point of view, visual composition — belongs to Anthropic's `frontend-design` skill. Use this file for what that direction means *structurally*: the evidence it should rest on, the UX consequences it carries (density, pacing, trust), and the structural causes of a generic outcome. Do not originate a palette or typeface here.

## Why derivation, not preference

Scribeo's standard is premium, modern, editorial, sophisticated. That describes **quality of decision-making**, not a look. Applying one house style to every client is the same failure as applying a template — it just fails more expensively. Two sites that both meet the standard should be unmistakably different.

## The derivation sequence

Work in this order. Later inputs refine earlier ones; they never override them.

**1. Brand assets (strongest signal)**
Logo construction, existing typefaces, palette, photography, print collateral, packaging. A logo set in a high-contrast didone is telling you something a brief never will. Extract, don't invent — if the brand has one accent colour, the site has one accent colour.

**2. Industry convention — honour or break, deliberately**
Every sector has a visual grammar users rely on to feel safe. Identify it, then decide consciously:
- **Honour it** where trust is the bottleneck — finance, healthcare, legal, insurance.
- **Break it** where differentiation is the bottleneck and the audience is sophisticated — fashion, hospitality, creative services.
Breaking convention is a strategy with a cost. Pay it on purpose, never by accident.

**3. Audience expectation**
Age, device, context, technical comfort, emotional state on arrival. Someone comparing four contractors after a burst pipe needs a phone number above the fold. Someone browsing a villa rental for next summer wants immersion and time. Same "premium" standard, opposite interfaces.

**4. Content shape**
Design for the content that exists. Count it: how many services, how long are the names, how many images, what quality and aspect ratio, is there real copy or placeholder?
A three-card grid is a decision only when there are three things. With five, it is a mistake you are now styling around.

**5. Business goal**
See `conversion.md`. Goal sets density, CTA frequency, and how much of the page is persuasion versus information.

**6. Supplied references**
Read them for *intent*, never for copying. The client sends a site because of one quality — the whitespace, the type, the photography, the calm. Identify that quality and name it. Then achieve it with their brand, not the reference's.

## State it before you build

Write the direction in plain prose and confirm it. Two examples, both meeting the Scribeo standard, sharing nothing:

> **Heritage law firm.** Convention honoured — this audience reads deviation as risk. Transitional serif headings, humanist sans body, deep navy and warm off-white, no pure black. Photography is people and place, never stock handshakes. Generous measure, slow vertical rhythm, no motion beyond state feedback. Density low; authority comes from restraint and specificity.

> **Independent coffee roaster, DTC.** Convention broken — the category is beige minimalism, the brand is loud. Condensed grotesque display at large sizes, tight tracking, editorial grid with deliberate asymmetry. Palette from packaging: burnt orange, cream, ink. Product photography full-bleed and warm. Density high above the fold — this audience scans and buys.

If you cannot write that paragraph, you do not yet know what you are building. Ask, or propose two distinct directions — never average them into something inoffensive.

## Premium versus expensive-looking

| Premium | Merely decorated |
| --- | --- |
| Restraint with one deliberate focal moment | Every section competing for attention |
| Type doing the work — scale, weight, measure | Effects doing the work — gradients, glass, glow |
| Whitespace as composition | Whitespace as leftover margin |
| One accent, used rarely and meaningfully | Multiple accents diluting each other |
| Photography treated as content | Photography treated as texture |
| Specific, concrete copy | Superlatives and abstractions |
| Motion that clarifies state | Motion that announces itself |

Cheapness reads as *undifferentiated*, rarely as *plain*. Plain and specific beats elaborate and generic every time.

## The generic failure modes

Each has a cause and a fix.

| Symptom | Actual cause | Fix |
| --- | --- | --- |
| "Looks like every SaaS site" | Sections chosen from a mental template, then filled | Derive sections from this content and this goal |
| "Looks AI-generated" | Even density, no focal hierarchy, no brand-specific decision anywhere | Establish one focal moment; delete unreasoned elements |
| "Feels cheap" | Type scale too flat, spacing scale inconsistent, stock imagery | Widen type contrast, unify the scale, fix the imagery |
| "Feels cold" | No brand voice in copy, no human imagery, geometric everything | Specific copy, real photography, warmth in palette |
| "Feels busy" | Every element at similar visual weight | Demote 80% of it — hierarchy is subtraction |
| "Doesn't feel trustworthy" | Vague claims, no specificity, invented-looking proof | Concrete detail, real proof, or no proof section |

## Hard rule on proof

Never fabricate testimonials, statistics, client logos, review counts, user numbers, awards, trust badges, urgency, or scarcity. Not as placeholder, not as demo, not "for now".

If the client has not supplied proof: build the empty slot with a clear note, or omit the section and strengthen what is true. Specific real detail ("trading since 1974, four staff, one workshop") outperforms invented scale every time — and invented proof is a lie published in the client's name.
