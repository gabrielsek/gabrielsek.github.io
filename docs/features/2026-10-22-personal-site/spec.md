---
status: approved     # draft | approved | building | done | superseded
supersedes:
superseded_by:
ticket:
---
# Personal professional website — gabrielsek.github.io

## Problem

Hiring managers, recruiters, potential clients and peers who look up the owner (Gabriel Sęk, backend
engineer in banking, Warsaw) today find a LinkedIn profile and a GitHub account with five public
repositories. Three of those were last pushed between 2021 and 2024. The account has no bio, no
website and no profile README. These visitors usually decide within a minute whether to look
further, and nothing makes that minute memorable.

The owner wants the site to work **like a business card**. The impression comes first: one clever,
well-made idea that a visitor remembers. Content stays compact. Projects keep their own pages for
the visitors who look further. CV detail is a short section, not the subject of the site.

The CV exists only as a PDF that is shared privately. It carries the owner's personal email and
phone number, and the owner does not want either on the public web because the address already
draws spam.

## Goals

| ID | Goal |
|---|---|
| G-1 | The first screen says who the owner is and what they do, and offers the next step (LinkedIn, GitHub). |
| G-2 | The site is memorable through one idea, the **blueprint layer**, which runs through every page and is discoverable from the first screen. |
| G-3 | Projects are shown as case studies with explorable architecture diagrams. |
| G-4 | Compact CV-level detail (roles, skills, education) is on the page, without exposing email or phone. |
| G-5 | The design emphasis costs nothing in quality: the site is fast, accessible (WCAG 2.2 AA), indexable, shareable, and survives deep links and refreshes on GitHub Pages. |
| G-6 | Adding a project later means editing content files only, with no component changes, in 15 minutes or less excluding writing time. |
| G-7 | The site and the GitHub profile link to each other. |
| G-8 | Without cookies or a consent banner, the owner can see visits per page, where they came from, and whether visitors find the blueprint. |
| G-9 | The site reads as a deliberate design, not generated output: it passes the owner's anti-slop checklist (Design rules). |

## Non-goals

Out of scope. The breakdown and the task runs must not add these:

- An email address, `mailto:` link, phone number or contact form anywhere: on a page, in page source, in metadata, in repository files or in commit metadata.
- A downloadable CV PDF, and a print stylesheet that stands in for one.
- A literal business-card object of any kind (3D card, flippable card, letterpress card). "Business card" describes the impression, not a shape.
- Preloaders, intro sequences, scroll hijacking, smooth-scroll libraries and scroll-triggered reveals.
- WebGL, and any 3D scene.
- Hiding the system cursor or replacing it with a custom one.
- A skill filter, and any other interface for matching against a job spec.
- A writing, blog or notes section. No RSS.
- Case studies of employer work (mBank, BEC). Employer work appears only as role achievements.
- Any language other than English.
- A photo of the owner.
- An availability or "open to work" status line.
- Live running demos of the backend projects.
- A results chart for java-overload-recovery.
- A visitor "note to self" feature.
- The durable-agent-runtime project at launch. It can be added later with the add-project flow.
- A custom domain at launch (see OQ-8).
- Cookies, a consent banner, Google Analytics.
- Any server, server-side rendering at request time, database, CMS, comments or site search.
- Single-key keyboard shortcuts.
- Copying any asset, layout composition, copy or identity element from the reference sites.
- Testimonials, pricing tiers, bento grids, feature-card rows, dot-grid backgrounds, glowing orbs or blobs, and decorative terminal or code windows.
- Gradients, drop shadows, frosted glass and rounded corners (the lens is the only round shape).
- Emoji, sparkle icons, checkmark bullets, and any icon library.
- Generating the profile README from site data.
- Automated dependency-update bots.

## Approach

Angular 22 (Angular CLI), with every route **prerendered to its own HTML file at build time**
by Angular's own SSR/prerender package (`@angular/ssr`). The site is deployed by GitHub Actions to
GitHub Pages from the public repository `gabrielsek/gabrielsek.github.io`. After the first load,
Angular takes over and routes client-side, so the site behaves as a single-page application
(SPA).

**Content pipeline.** Before the Angular build, a **content script** owned by this repository
(Node, no model calls) does four things:

1. Reads the content files.
2. Validates them against the schemas in Data & contracts.
3. Converts each case study's Markdown to HTML.
4. Lays out every diagram, and every variant of it, to SVG.

It writes generated, typed modules that the components import. The same project data drives the
prerender route list and `sitemap.xml`. No Markdown parser and no graph-layout library ever ship to
the browser.

**Blueprint layer.** Each page carries a second rendering of its own content: the same content,
drawn as line-art in a fixed blueprint palette, with annotations. It sits exactly over the finished
page and is decorative, so it is hidden from assistive technology.

- A circular CSS mask that follows the pointer reveals it (the lens).
- A header toggle switches the whole viewport to it.

Annotation values are measured from the finished layout in the browser at runtime, so they are
always true. The blueprint code loads after the route's content has painted. No WebGL.

**Structure.** This is a static content site with no runtime IO apart from an analytics beacon.
Hexagonal ports and adapters would add layers that buy no testability here, so the code uses a
plain three-part shape:

- content (data and Markdown files)
- build-time pipeline (the content script)
- presentation (Angular components, the blueprint layer among them).

### Rejected alternatives

The decision log below lists the rejected alternatives for every decision. These two are for the
overall build shape (D-27):

- **AnalogJS 2.9.1** (an Angular meta-framework with Markdown content routes and static prerender; its peer range covers Angular 22). Rejected because it puts a third-party layer between the site and Angular that may lag Angular majors. Its file-based routing also differs from Angular CLI conventions. Diagram layout would still need custom code.
- **Rendering content in the browser** (fetch Markdown and diagram data at runtime). Rejected because the Markdown parser and the layout library would ship to every visitor. `@dagrejs/dagre` 3.1.1 is 1.4 MB unpacked and `elkjs` 0.12.0 is 8 MB unpacked, which breaks the 500 KB budget.

The blueprint's implementation (D-43) rejected these:

- **WebGL edge-detection shader over the page.** It needs WebGL, it is heavy, and it draws text badly.
- **Canvas screenshot of the DOM, drawn as line-art.** It is slow to capture, goes stale on resize, and can't show true measurements.

### Decision log

Each row is settled. Reopen one only with a reason that the row does not already cover. A
superseded row stays in the log so the reason for the change is visible.

| ID | Decision | Reason | Rejected alternatives |
|---|---|---|---|
| D-01 | The first screen speaks to **hiring managers** first. The headline leads with role and domain. | Recruiters read the same copy. Clients get the same signal without a services section. | Recruiter-first keyword headline: reads flat to engineers. Clients first: needs a services section and an availability line to keep true. Peers first: needs public work and writing that doesn't exist yet. |
| D-02 | Contact is **LinkedIn and GitHub links only**. | The owner's email draws spam, and any address in a page is scraped. | `mailto:` link: exposes the address. Form service: a third-party data processor, spam handling, and a failure mode. Visitor's own note: lives in one browser, is lost on another device, and duplicates the recruiter's ATS (applicant tracking system). |
| D-03 | **No CV PDF** on the site. | The existing PDF prints email and phone. A variant would either drift from the page or need a headless browser in CI. | PDF generated at build: CI complexity. Hand-made web variant: drifts. Full CV as-is: exposes contact details. |
| D-04 | **English only.** | One copy to maintain, no i18n routing. Polish recruiters read English. | English + Polish: double upkeep, plus per-language routes and hreflang. Polish only: rules out remote and EU roles. |
| D-05 | **No availability line.** | It goes stale and lies. The links do the job. | Status line from a data file. "Not looking" framing. |
| D-06 | Launch projects: **chat** and **java-overload-recovery**. | Owner's selection. | Employer case studies: owner excluded them, and the role achievements carry that work. durable-agent-runtime: not selected for launch. |
| D-07 | Each project is a **card plus a case-study route**. The card presentation is superseded by D-49 (sheet index); the route per project stands. | Works for a private repo, and gives each project a deep-linkable, shareable URL. | Card linking to the repo: useless for a private repo. Live demo: JVM backends can't run on Pages. |
| D-08 | Years of experience are **computed from the earliest role start (2021-05)**. | Always matches the CV dates and never needs an edit. | Fixed "5+ years": needs a yearly edit. "7 years": contradicts the dates shown on the page. |
| D-09 | A private repo is shown with an **optional repo link plus a note**. | The case study ships now. The link appears when the repo goes public, with no redesign. | Making the repo public first: thesis-publication rules are unknown. Leaving the project off: a one-project portfolio. |
| D-10 | ~~Sections: hero, projects, experience timeline, skills, education, footer.~~ **Superseded by D-38.** | | |
| D-11 | A **short profile README** in `gabrielsek/gabrielsek` links to the site, and the site links back. | Visitors arriving from GitHub find the site. About 15 minutes of work. | No README: the GitHub profile stays bare. README generated from site data: needs a cross-repo token as a secret for a file that changes twice a year. |
| D-12 | ~~Interactions: skill filter, expandable timeline, clickable architecture diagrams.~~ **Superseded by D-35 and D-38.** Expandable roles and clickable diagrams stay; the skill filter is dropped. | | |
| D-13 | ~~Tone: engineering-minimal.~~ **Superseded by D-40.** | | |
| D-14 | Theme: **follow the OS, plus a toggle that remembers the choice.** | Respects the OS setting and lets the visitor override it. | System only: no override. Dark only: ignores the OS setting and reads worse for some visitors. |
| D-15 | ~~Motion: state changes only, 200 ms or less; nothing on load or scroll.~~ **Superseded by D-41.** | | |
| D-16 | Framework: **Angular 22** (22.2.3 on npm, 2026-10-22). | The owner's stack. A true SPA after first load. The public repo also demonstrates a CV skill. The blueprint is DOM and CSS, which any framework handles. | SvelteKit 3.1.0 static: a framework new to the owner, and it says nothing about CV skills. Astro 7.4.2: multi-page by default, so it meets the SPA constraint only through its client router. Plain HTML/JS: hand-written hash router, no per-route previews, a homegrown framework. |
| D-17 | Routing: **prerender every route to its own file**, with `404.html` as the not-found view. | Deep links and refreshes answer HTTP 200 with content even without JS, and every route gets its own link preview. | `404.html` redirect trick: every deep link first answers HTTP 404, so bots see an error and no-JS visitors see nothing. Hash routing: fragments aren't indexed and there are no per-route previews. |
| D-18 | Deploy: **GitHub Actions** builds, checks and publishes. | Checks gate the deploy, and no build output goes into git history. | Branch deploy: build output in history, and nothing forces the checks to run. |
| D-19 | **No custom domain** at launch. | Free, with automatic HTTPS. | Buying a domain now: a yearly fee, DNS work, and the site breaks if the domain lapses. See OQ-8. |
| D-20 | Analytics: **GoatCounter, hosted, cookieless.** | Gives page views and referrers with no cookies or banner. Free, donation-supported, about 3.5 KB of script. Documents use in SPAs. | None: no visit data at all. Google Analytics: cookies, so a consent banner. Cloudflare Web Analytics: also cookieless and viable; not chosen because GoatCounter is open source, can be self-hosted, and documents SPAs. |
| D-21 | Performance budget: **Lighthouse mobile Performance ≥ 90; first load ≤ 500 KB.** Unchanged by the design-led revision (D-41). | Comfortable for prerendered Angular, and enforced in CI. | ≥ 95 and ≤ 300 KB: tuning time instead of content time. Relaxing to ≥ 70 and 1.5 MB for richer motion: rejected in D-41. |
| D-22 | Without JS: **all content except role achievements**, which stay collapsed and can't be opened (R-24). | Owner's choice. | Native disclosure so achievements open without JS. The owner accepted that no-JS visitors can't read achievements. The text is still in the HTML for crawlers. |
| D-23 | **No photo.** | Nothing to supply; the LinkedIn photo covers it. | Small headshot in the hero. |
| D-24 | Adding a project means **one data entry plus one Markdown file**, checked at build. | No component edits, and mistakes fail the build rather than the page. | Typed TS data only: long prose in TS strings. One component per project: every project is a code change. |
| D-25 | Diagrams are **generated from node/edge data, with layout computed at build.** | Owner's choice. Laying out at build keeps the layout library out of the browser. Diagrams are drawings already, so they suit the blueprint idea. | Hand-drawn SVG plus an id map: more visual control, not chosen. Mermaid: large runtime and limited click handling. |
| D-26 | Role achievements are **trimmed CV bullets**: a ≤ 90-character headline plus the full bullet. | Nothing to write from scratch; the owner reviews the wording. | Verbatim bullets: dense CV prose. Owner rewrite: blocks launch on new copy. |
| D-27 | Build shape: **Angular CLI plus an owned content script.** | First-party Angular only. The pipeline is small and testable. | See Rejected alternatives above. |
| D-28 | This spec lives in the **site repository**, so it is public. | The breakdown and task files sit next to the code. | A private location apart from the code. Consequence: this spec must never contain the owner's email or phone. |
| D-29 | Accessibility target: **WCAG 2.2 AA.** | The owner's default. | AAA: not requested. |
| D-30 | **Fonts self-hosted.** One static social image; every route sets its own preview title and description. | Keeps GoatCounter the only third-party host at runtime. No image-generation step. | Google Fonts: a third-party request. Per-route generated images: an image pipeline in CI. |
| D-31 | **No print stylesheet.** | Follows from D-03: no CV document. | A print-styled CV route. |
| D-32 | **Design and one idea lead; content stays compact.** This replaces the first revision's "portfolio first, CV second" framing. | Owner's direction after the first draft, with ten reference sites given: [letude.group](https://letude.group/), [laxspace.co](https://www.laxspace.co/), [geex-arts.com](https://www.geex-arts.com/), [abhishekjha.me](https://abhishekjha.me/), [madeinuxstudio.com](https://madeinuxstudio.com/), [interaction.hnine.com](https://interaction.hnine.com/), [bindery.co](https://www.bindery.co/), [madebynull.com](https://www.madebynull.com/), and the favourites [eduardbodak.com](https://www.eduardbodak.com/) and [portalone.studio](https://www.portalone.studio/). | Content-led portfolio (the first revision). Raised and accepted: eight of the ten references are design studios, whose look can read as "creative frontend developer" to a backend hiring manager. The project pages carry the engineering signal. |
| D-33 | **"Business card" means the impression, not a card object.** | Owner's correction. | 3D business card, letterpress card, a card you flip or tear. |
| D-34 | **The idea need not relate to the owner's work.** | Owner's call: the references' ideas aren't about their owners' work either, and the project pages carry the engineering. The cost is that the idea has to be excellent on its own merits. | Work-tied ideas: a load simulation of the thesis, an agent learning to reach the links (B.Sc. RL thesis), the site tracing its own load, a breakable saga receipt, state that survives a reload. |
| D-35 | Idea: the **blueprint layer**. Every page has its technical drawing underneath, revealed by a pointer lens and a whole-site toggle. | The cheapest option (CSS masking, no WebGL). Fits the chosen look (D-40). Works with keyboard (toggle) and touch (D-37). | Infinite pannable canvas: gesture handling and crisp text at every zoom level. Light and paper: WebGL shader and a fight with the 500 KB budget. Playable space (like Lax Space or bruno-simon.com): the biggest build, and hiring managers may stop at the list. Throwable letters: a familiar trick. Breakout with the name as bricks: reads as a toy. Portal / gateway: borrows a studio's identity. |
| D-36 | Annotations are **measured live**, plus up to three **hand-written notes** per page. | Measured values are always true and follow the viewport. The notes give the drawing a voice. | Measured only: no voice. Hand-drawn per page: goes stale on every layout change and adds a drawing per new project. |
| D-37 | Touch: **toggle everywhere, plus hold-then-drag lens.** | Phone visitors meet the lens too; the hold keeps normal scrolling intact. | Toggle only on touch: phone visitors never see the lens. Device tilt: iOS motion-permission prompt, and awkward on a desk. |
| D-38 | Content: **intro screen, projects, a compact More section (roles, skills, education), footer.** Supersedes D-10 and the skill-filter part of D-12. | Content supports the design instead of competing with it. Two projects don't need a matching interface. | Keeping the skill filter: more interface competing with the blueprint. Roles and links only: skills and education only on LinkedIn. Full sections below an intro: content upkeep and design effort spread thin. |
| D-39 | The blueprint has a **fixed palette, separate from light and dark**. | The reveal works in both themes. | Blueprint as the dark mode: OS-dark visitors sit in the X-ray view permanently, so the reveal has nothing to reveal. |
| D-40 | Look: **Bodak base plus Portal One's technical detail.** From Bodak: cream ground, near-black ink, heavy grotesk, one loud accent. From Portal One: section numbers, hairlines, line-art. Supersedes D-13. Palette and accent are fixed by D-47. | Warm for recruiters, technical for engineers. The blueprint layer extends the technical detail. | Portal One alone: cold, and close to a studio's own look. Bodak alone: the least "backend". Studio Null's white minimalism: rests entirely on one object. Engineering-minimal (the first revision): not memorable enough for the new emphasis. |
| D-41 | Motion: **the quality bars stay and motion sits on top.** Content paints first, there's no preloader, and reduced motion gets a still frame. Supersedes D-15. | The visitor still decides in under a minute, and the speed and accessibility bars are part of the craft. | Relaxing only speed (Lighthouse ≥ 70, 1.5 MB, short intro): a slower first screen. Full showpiece (preloader, smooth scroll, scroll scenes): fails reduced-motion and keyboard use unless built twice. |
| D-42 | The owner's **anti-slop rules apply at spec time**, not as a fix after the build. | Design is now the point, and design rules cost least before anything exists. | Applying them after the build: rework of the part the owner cares most about. Status: applied in revision 3 (D-46). |
| D-43 | Blueprint implementation: **a decorative second rendering of the page, clipped by a CSS mask**, with annotations measured from the finished layer. | True alignment and true values. No WebGL. Cheap to keep in sync, because both layers come from the same data. | See Rejected alternatives above. |
| D-44 | **No single-key shortcut** for the blueprint. | WCAG 2.2 SC 2.1.4 (Character Key Shortcuts) forbids unmodifiable single-key shortcuts. | A "B" key toggle. |
| D-45 | **The lens hides while the keyboard is in use**, and the focus indicator is drawn in both layers. | Keyboard focus must never be hidden by the reveal. | Leaving the lens up: it can cover the focus indicator. |
| D-46 | The anti-slop rules are applied in three ways. The **design plan** (the rules' Pass 1) is written into this spec. Each checklist item becomes a **CI check** where it is deterministic, or a **design audit** item where it isn't. The audit **target is 0 boxes checked**, and the deploy is blocked above 2 (AC-11). | Checks that run on every push keep the rules from fading after launch. | A review pass after the build (rejected in D-42). Accepting the rules' own "deliberate" band of 0 to 4: looser than this site needs, given it is a design showcase. |
| D-47 | **The palette comes from technical drawing**: drafting paper, graphite, cyanotype blue, and the red pencil a reviewer marks a drawing up with. This refines D-40: the accent is red pencil, not Bodak's pink. | The rules ask for a palette from the subject's own world, and the blueprint idea supplies one. | Bodak's pink: borrowed identity, and close to neon. A generic modern palette. The first draft's pale-cyan line work: a default pastel. |
| D-48 | Type: **Archivo** for display and body, **IBM Plex Mono** for annotations. | Archivo's width and weight axes give both the heavy wide display cut and the reading cut from one variable family, so fewer font files. Plex Mono reads like engineering annotation. Both are OFL with Latin Extended (verified in the google/fonts metadata during revision 3). | Inter, Geist, Space Grotesk: flagged by the rules. A third family for display: more files, no gain. A technical-lettering font in the ISO 3098 style: licensing not verified. |
| D-49 | **Projects are a numbered sheet index**, and each case study is a **drawing sheet with a title block**. Supersedes the card presentation in D-07; the route per project stays. | Comes from the same drawing world, and avoids the card grid that turns into "three cards in a row" once a third project lands. | A card grid: generic, and flagged by the rules. |
| D-50 | **The accent appears only in the signature and in focus**: the lens ring, the Blueprint toggle's pressed state, blueprint notes, and the focus indicator. | "Spend your boldness in one place." The red pencil marks the drawing, as a reviewer's pencil would. | Accent on the display name or headings: spends the boldness twice. |
| D-51 | A **Privacy & terms page** at `/privacy`. | Handles the unglamorous parts the rules check for: what GoatCounter and browser storage hold, content copyright, the code licence, and a note on external links. | A footer line only: flagged by the rules as no privacy policy. A separate terms of service: the site offers no service or account, so a terms-of-use section covers it. |
| D-52 | **No icon library.** The toggles have text labels, arrows are type glyphs, and the only marks are LinkedIn's and GitHub's own. | The rules flag an unmodified icon library used as the whole icon system. | Lucide or any icon set. |
| D-53 | **Square corners everywhere** (radius 0); the lens is the only round shape. | Drafting sheets are square. The rules flag a uniform soft radius. | A soft radius on every element. |
| D-54 | **Defined loading states:** prerendered content, metric-matched font fallbacks, a progress line for slow navigations, and loading and unavailable states on the Blueprint toggle (R-64). | The rules flag content popping in raw. Nothing is fetched after load, so skeleton screens would have nothing to stand in for. | Skeleton screens. No loading states at all. |

## Design plan

This is Pass 1 of the owner's anti-slop rules: written before any code, then checked against the
checklist. The revisions that check produced are listed at the end. The build follows this plan
exactly. Every colour and type decision comes from it.

**Subject.** A personal site for Gabriel Sęk, a backend engineer in banking. It is for hiring
managers first, then recruiters, potential clients and peers. Its single job is to leave, in under a
minute, the impression of an engineer who builds things precisely, and then send the visitor on to
LinkedIn, GitHub or a project.

**World.** The technical drawing: drafting paper, graphite, the cyanotype blueprint, and the red
pencil a reviewer marks it up with. Every choice below comes from it.

**Palette.** Each of the three palettes has four or five named values. Contrast ratios were
computed by script; the method is in Provenance.

| Name | Hex | Palette | Role | Contrast |
|---|---|---|---|---|
| Drafting paper | `#F1ECE1` | light | ground | |
| Graphite | `#1D1C1A` | light | text, display name | 14.45:1 on paper |
| Graphite soft | `#57534B` | light | secondary text | 6.50:1 on paper |
| Paper rule | `#C9C1B1` | light | decorative hairlines | decorative |
| Red pencil | `#B8321A` | light | accent (R-60) | 5.08:1 on paper |
| Night | `#161513` | dark | ground | |
| Paper ink | `#EDE7DA` | dark | text, display name | 14.81:1 on night |
| Paper soft | `#A8A194` | dark | secondary text | 7.12:1 on night |
| Night rule | `#3A3833` | dark | decorative hairlines | decorative |
| Red pencil, night | `#E0603F` | dark | accent | 5.14:1 on night |
| Cyanotype | `#123B66` | blueprint | ground | |
| Blueprint white | `#E8EFF5` | blueprint | line work, annotation text | 9.82:1 on cyanotype |
| Blueprint faint | `#6F94BC` | blueprint | grid hairlines | 3.60:1 on cyanotype |
| Red pencil, blueprint | `#F49A7E` | blueprint | notes, focus | 5.30:1 on cyanotype |

No value is pure white. None goes above 85 % HSL saturation; the highest is red pencil,
blueprint, at 84 %.

**Type.**
- **Archivo** (Omnibus-Type, OFL) for display and body. Its width axis (62 to 125) gives a heavy, wide cut for the name and section numbers, and its normal width carries the reading text. One variable family covers both roles.
- **IBM Plex Mono** (OFL) for annotations, labels, section numbers and code. It reads like engineering annotation.

**Layout.** One scrolling page laid out like a drawing set: a title area, a numbered sheet index of
projects, a compact register of roles, skills and education, then the footer. Each project opens as
its own sheet with a title block.

```text
HOME, desktop
+--------------------------------------------------------------------+
| Gabriel Sęk             Projects   More        Dark theme  Blueprint |
+--------------------------------------------------------------------+
| 01                                                                 |
|   GABRIEL                                                          |
|   SĘK                              (Archivo, heavy, wide)          |
|   Backend engineer building event-driven systems for banking.      |
|   5+ years · Warsaw, Poland                    LinkedIn   GitHub   |
|   Move the pointer to see how this page is built.                  |
+------------------------------------------------------------ rule --+
| 02  Projects                                                       |
|   SHEET 01   chat                       In progress           ->   |
|              Summary line                Java · Kafka · Postgres   |
|   SHEET 02   java-overload-recovery     Thesis work           ->   |
+--------------------------------------------------------------------+
| 03  More                                                           |
|   Roles       mBank S.A.  Software Engineer  Nov 2024 – Present [+]|
|               BEC Financial Technologies  May 2021 – Oct 2024  [+] |
|   Skills      grouped plain lists                                  |
|   Education   two lines                                            |
+--------------------------------------------------------------------+
| LinkedIn  GitHub  Source  Privacy & terms        GoatCounter note  |
+--------------------------------------------------------------------+

CASE STUDY, desktop
+--------------------------------------------------------------------+
| header                                                             |
+--------------------------------------------------------------------+
| SHEET 01 / 02                                                      |
| chat                                                   (h1)        |
| Summary line                                                       |
| +-- TITLE BLOCK ------------------------------------------------+  |
| | Status  In progress | Skills  Java, Kafka ... | Repo  link     |  |
| +---------------------------------------------------------------+  |
| Problem / Design / Decisions / Outcome   (prose, 60-80 characters) |
|   [ diagram ]  [ details panel ]                                   |
| <- Sheet 02            All projects                  Sheet 02 ->   |
+--------------------------------------------------------------------+
```

**Signature.** The blueprint layer: a lens that X-rays the page into its own measured drawing.
Everything around it stays quiet: graphite on paper, one accent used only for the signature and
for focus, square corners, and no decoration that isn't a measurement.

**Revisions after checking the plan against the checklist.** Each of these is what the first
draft would have produced for any similar brief.

| First draft | Revised to | Why |
|---|---|---|
| "One saturated accent", undefined | Red pencil, used only for the signature and focus (D-47, D-50) | Comes from the subject, and keeps the boldness in one place |
| Pale-cyan line work in the blueprint | Blueprint white and blueprint faint | Pale cyan is a default pastel |
| Project cards in a grid | Numbered sheet index and drawing sheets (D-49) | Avoids a card row that becomes "three in a row" |
| Icon toggles from an icon set | Text-labelled toggles, no icon library (D-52) | Lucide-style default |
| Headline default containing an em dash | Rewritten without one (OQ-1) | Em-dash copy |
| A soft radius on cards and buttons | Square corners (D-53) | Uniform soft radius |
| A privacy line in the footer | Privacy & terms page (D-51) | No privacy policy or terms |
| No stated loading states | Defined loading states (D-54) | Content popping in raw |
| For a backend engineer: a decorative terminal in the intro | Excluded (Non-goals) | The most tempting default for this owner |

## Design rules

The owner's anti-slop checklist, applied item by item. "Handled by" says where this spec addresses
the item. "Check" says how it is verified. The design audit (AC-11) scores the built site against
this table: the target is 0 boxes checked, and the deploy is blocked above 2.

| # | Checklist item | Handled by | Check |
|---|---|---|---|
| 1 | Harsh gradients | No gradient anywhere (R-61) | CSS check |
| 2 | Pure white background | No palette value is `#FFFFFF` (Design plan, R-61) | CSS check |
| 3 | Rainbow / multi-colour text | Ink colours only; the accent only where R-60 allows | Accent scan |
| 4 | Purple-and-black scheme | No purple in any palette | Token review in the audit |
| 5 | Neon colours | Every value at or below 85 % HSL saturation (R-61) | Token check |
| 6 | Default pastel colours | Palette taken from drafting materials; pale cyan removed | Audit |
| 7 | Heavy drop shadows | No `box-shadow` or `text-shadow` (R-61) | CSS check |
| 8 | Liquid / frosted glass | No `backdrop-filter`, no translucent panels (R-61) | CSS check |
| 9 | Uniform soft corner radius | Radius 0 everywhere except the lens (R-61) | CSS check |
| 10 | Coloured left stripe on cards or callouts | No callout boxes; notes are red-pencil annotations | Audit |
| 11 | Inter / Geist / Space Grotesk | Archivo and IBM Plex Mono (R-49) | Font check |
| 12 | Em dashes scattered through copy | None in UI strings; at most one per 250 words of case-study prose (R-62) | Content validation |
| 13 | "It's not X, it's Y" construction | Not used in site copy (R-62) | Content validation |
| 14 | Emojis as UI or bullet markers | No emoji in site copy or UI (R-62) | Content validation |
| 15 | Fake testimonials | None (Non-goals) | Audit |
| 16 | Three feature cards in a row | Projects are a sheet index (R-15) | Audit |
| 17 | Three pricing tiers | No pricing (Non-goals) | Audit |
| 18 | Bento grids | None (Non-goals) | Audit |
| 19 | Dot-grid backgrounds | The blueprint grid is hairlines, never dots (R-61) | Audit |
| 20 | Radial orbs / glowing blobs | None; the lens ring is a hairline without glow or blur (R-61) | CSS check |
| 21 | Decorative fake terminal window | None (Non-goals) | Audit |
| 22 | Checkmark bullets | Lists use numbers or plain text | Audit |
| 23 | Sparkle icons | None (R-62, R-63) | Content validation, audit |
| 24 | Lucide icons unmodified as the icon system | No icon library (R-63) | Dependency check |
| 25 | Hover animations on every element | Hover states only on links, buttons and diagram nodes, with no movement (R-31) | CSS check |
| 26 | Animated arrows nudging rightward | Arrows never move (R-31) | CSS check |
| 27 | No skeleton loaders (content pops in raw) | Prerendered content and defined loading states (R-64) | Browser test |
| 28 | No real product demo | The blueprint shows real measured values; case studies show real designs with explorable diagrams | Audit |
| 29 | No terms of service | Terms-of-use section on `/privacy` (R-65) | Browser test |
| 30 | No privacy policy | Privacy section on `/privacy` (R-65) | Browser test |

The rules also set a baseline that applies without being announced on the page: a responsive
layout (R-39), visible keyboard focus (R-13, AC-04), respect for `prefers-reduced-motion` (R-32),
and real semantic HTML (R-66).

## Behaviour

`R-nn` items are the functional requirements. Each one states what can be observed when it is met.
Acceptance criteria says how each one is verified.

### Routes

| Path | Shows | Served from |
|---|---|---|
| `/` | Home: intro screen, projects, More, footer, as one scrolling page | prerendered `index.html` |
| `/projects/<slug>` | Case study for that project | prerendered file for that route |
| `/privacy` | Privacy & terms page (R-65) | prerendered file for that route |
| anything else | Not-found view | `404.html` (GitHub Pages serves it with HTTP 404) |

Home sections have in-page anchors: `#projects` and `#more`.

### Intro screen (G-1)

- **R-01** At 360×640 and at 1366×768 CSS px, the home route shows the following without scrolling: the owner's name at display size, the headline (OQ-1), the line `{n}+ years · <location>`, a LinkedIn link, a GitHub link and the blueprint hint (R-06).
- **R-02** `n` is the number of whole years between `site.careerStart` (2021-05) and the current date. The prerendered HTML carries the value as of the build date. After JS loads, the client recomputes it from the visitor's clock and never shows a value lower than the build value. Observable: with the system clock set to 2027-05-02, the intro reads "6+ years" after load.
- **R-03** The LinkedIn and GitHub links are icon links using each brand's own mark, in the ink colour where the brand's guidelines allow it (A-15). They don't come from an icon library (R-63). Their accessible names are "LinkedIn profile" and "GitHub profile", each target is at least 24×24 CSS px, and they open in the same tab.
- **R-04** When `site.featured` names a project slug, the intro shows a "Featured: <project title>" link to that case study. When it is `null` (the launch state), nothing renders and no empty space is reserved.
- **R-05** No email address, `mailto:` link, phone number or contact form appears on any route or in any built file (enforced by R-52).
- **R-06** **Blueprint hint.** A one-line hint sits on the intro screen, with text from `site.hint`:
  - On a device whose primary pointer is fine, it reads the pointer text (default: "Move the pointer to see how this page is built").
  - On a coarse pointer, it reads the touch text (default: "Press and hold to look inside").

  The hint disappears once the visitor uses the lens or the toggle, and stays hidden for the rest of the browser session. If session storage is unavailable, the hint stays visible and no error is raised. If the blueprint code fails to load, the hint is hidden.

### Blueprint layer (G-2)

- **R-07** **Two layers.** Every route has a finished layer (the design) and a blueprint layer. The blueprint layer shows the same content in the same layout, drawn as line-art in the blueprint palette (R-48):
  - element boxes outlined
  - display type hollow
  - baselines and the column grid in hairlines
  - annotations (R-11) and notes (R-12).

  The box of every element in the blueprint layer matches its finished-layer box within 1 CSS px, at every viewport width from 320 px up, and after any resize.
- **R-08** **Lens (fine pointer).** A circle of 200 CSS px diameter, centred on the pointer, shows the other layer: the blueprint in finished mode, and the finished design in blueprint mode.
  - It appears on the first pointer movement over the page.
  - It hides when the pointer leaves the document, and on any key press (D-45). It reappears on the next pointer movement.
  - The system cursor stays visible.
  - The lens never intercepts clicks, hovers, text selection or scrolling: everything under it behaves as if it weren't there.
  - It trails the pointer by at most 100 ms. Under reduced motion it doesn't trail.
- **R-09** **Toggle.** A header button labelled "Blueprint" switches the whole viewport between finished and blueprint mode, and exposes its pressed state.
  - The mode persists across client-side route changes and resets on a full page load.
  - The switch takes 300 ms or less; under reduced motion it is instant.
  - The toggle is the single-pointer, non-drag alternative to the lens (WCAG 2.2 SC 2.5.7).
  - The toggle is in the prerendered HTML and shows only when JS runs, so it never pops in.
  - Until the blueprint code is ready, the toggle shows a loading state, and an activation in that state takes effect as soon as the code is ready.
  - If the code fails to load, the toggle reads "Blueprint unavailable" and is disabled.
- **R-10** **Touch (coarse pointer).**
  - A press held for 300 ms without moving more than 10 CSS px shows the lens, centred 80 CSS px above the touch point.
  - Dragging moves the lens; lifting the finger hides it.
  - A touch that moves before 300 ms scrolls the page normally.
  - A press that starts on a link or button doesn't start the lens, so the platform's own long-press behaviour applies.
- **R-11** **Measured annotations.** In the blueprint layer, each annotated element (Data & contracts lists them) shows:
  - its box size, as `W × H`
  - for text: the first declared font family name, the font size and line height, and the letter spacing when it isn't zero
  - for layout containers: the column count, column width and gutter
  - the gap to the next annotated sibling.

  Every value equals what the browser computes for the finished layer, rounded to whole CSS px. Values update within 150 ms after a resize ends and after web fonts finish loading.
- **R-12** **Notes.** Up to three hand-written notes per page, each 80 characters or less and anchored to an annotated element, appear only in the blueprint layer. They come from `content/blueprint-notes.yaml`.
- **R-13** **Accessibility of the layer.** The blueprint layer is hidden from assistive technology and can't be focused. It adds no tab stops and no duplicate text to the accessibility tree. The focused element's focus indicator is visible in both layers.
- **R-14** **Cost.** The blueprint code loads after the route's content has painted and doesn't change the route's Largest Contentful Paint (LCP). With Chrome devtools CPU throttling at 4×, five seconds of continuous lens movement over the home page produces no task longer than 50 ms.

### Projects (G-3)

- **R-15** The Projects section is a **sheet index**, not a card grid (D-49). It has one row per project, sorted by `order` ascending and numbered "SHEET 01", "SHEET 02" and so on. A row shows:
  - sheet number
  - title
  - summary
  - status label
  - skill labels (plain text, not interactive)
  - a link to the case study
  - either a repo link (when `repo` is set) or the `repoNote` text (when it isn't).

  Rows are separated by hairline rules, and have no box, background or shadow.
- **R-16** Activating a row's title or its "Read case study" link navigates client-side to `/projects/<slug>`.

### Case study page (G-3)

- **R-17** `/projects/<slug>` is laid out as a drawing sheet. It shows, in this order:
  1. the sheet number ("SHEET 01 / 02")
  2. title (the page's only `h1`)
  3. summary
  4. a title block holding status, skill labels, and the repo link or `repoNote`
  5. the Markdown body
  6. "All projects" (to `/#projects`), plus previous and next sheet links in `order`, wrapping around.

  The body holds the sections Problem, Design, Decisions and Outcome. Outcome may describe the current state of unfinished work.
- **R-18** **Diagram nodes.** Every diagram node can be reached with the keyboard (Tab) and activated with click, Enter or Space. Activating a node:
  - highlights it and its connected edges
  - shows its label and description in a details panel next to the diagram (below it under 768 px).

  Activating the selected node again, or pressing Escape, clears the selection. At most one node is selected at a time.
- **R-19** **Variant groups.** A diagram with variant groups shows one segmented control per group. Choosing an option swaps the nodes of that variant into the diagram, using a layout precomputed at build. The previous selection clears if its node is no longer shown. Each group's default option comes from data. Launch content: chat has three groups:
  - persistence: JPA | in-memory
  - broadcast: Kafka | in-memory
  - transport: REST | WebSocket

  java-overload-recovery has one group: threads: platform | virtual.
- **R-20** Every diagram has a text alternative that lists every node with its description and every edge (from → to, label). It is in the prerendered HTML and programmatically associated with the diagram. It describes the default variant, plus each variant option's nodes.
- **R-21** In the blueprint layer, diagrams are drawn in the blueprint palette, and every node box carries its measured size (R-11).

### More section (G-4)

- **R-22** Home has a `#more` section after Projects, with three blocks in this order: Roles, Skills, Education.
- **R-23** **Roles** are listed newest first, one line each: company, title, date range ("Nov 2024 – Present") and computed duration ("2 yrs"). All are collapsed on load. Activating a role line expands it, showing each achievement as a headline of 90 characters or less, followed by the full bullet text. Activating it again collapses it. Several roles may be open at once. The line exposes its expanded state to assistive technology.
- **R-24** Without JS, role lines are visible. Achievements are in the HTML but hidden, and they can't be opened (D-22).
- **R-25** **Skills** show the technical groups as plain text lists, followed by a "Spoken languages" line.
- **R-26** **Education** shows one line per entry: institution, degree and field, and dates. A thesis title, when present, goes on its own line beneath.

### Header and footer

- **R-27** The header holds, in order:
  - the owner's name, linking to `/`
  - anchor links to Projects and More (to `/#<section>` on other routes)
  - the theme toggle: a text button labelled "Dark theme", whose pressed state is true while the dark theme is on
  - the Blueprint toggle (R-09), also text-labelled.

  "Skip to content" is the first focusable element on every route.
- **R-28** The footer holds:
  - the LinkedIn link
  - the GitHub profile link
  - a "Source" link to `github.com/gabrielsek/gabrielsek.github.io`
  - a "Privacy & terms" link to `/privacy`
  - the text "Anonymous page-view counts by GoatCounter. No cookies."

### Theme

- **R-29** On a first visit, the theme follows `prefers-color-scheme`. The theme toggle switches between light and dark. The choice is stored in `localStorage` and overrides the OS setting on later visits. The stored theme is applied before first paint, so the other theme never flashes. Diagrams use theme colours. If `localStorage` is unavailable or throws, the toggle still works for the current page lifetime and no error is raised. The blueprint palette is identical in both themes (D-39).

### Motion (G-5)

- **R-30** No preloader, intro sequence, scroll hijacking, smooth-scroll library or scroll-triggered reveal. No content is hidden waiting for an animation. After a route change, the new route's content is in the DOM and interactive before any transition finishes.
- **R-31** Allowed motion, and its limits:
  - UI state transitions (expand, collapse, theme switch, blueprint switch, diagram selection and variant swap): 300 ms or less
  - route transitions: 400 ms or less
  - lens trailing: 100 ms or less
  - hover states: 200 ms or less.

  Hover states exist only on links, buttons and diagram nodes. They change colour, underline or outline only: no movement, scaling or animation. Arrows never move.
- **R-32** With `prefers-reduced-motion: reduce`, every transition and animation has zero duration and the lens follows the pointer without trailing. Nothing else changes; the lens and the toggle still work.

### Navigation and focus

- **R-33** After a client-side route change, `document.title` updates and focus moves to the new page's `h1`.
- **R-34** Browser Back from a case study returns to the home page at the scroll position it was left at, with the blueprint mode unchanged.

### Deep links and not-found (G-5)

- **R-35** Opening or refreshing `/`, `/privacy`, or `/projects/<slug>` (with or without a trailing slash, for every slug) ends in HTTP 200, either directly or after one redirect to the canonical form (R-40). The route's content is present in the initial HTML (visible in view-source).
- **R-36** Any other path, `/projects/<unknown-slug>` included, returns HTTP 404 and shows the not-found view: the heading "Page not found", the requested path, and a link to `/`. The view carries `<meta name="robots" content="noindex">`.
- **R-37** Opening `/#projects` or `/#more` scrolls to that section. An unknown fragment leaves the page at the top and raises no error.

### Without JavaScript (G-5)

- **R-38** With JS disabled:
  - The text of the intro, projects, case studies, More section and footer is readable.
  - Role lines show, but achievements don't (R-24).
  - Diagrams show their default variant as static SVG, and their text alternative is available to assistive technology.
  - The theme follows the OS.
  - The theme toggle, the Blueprint toggle, the hint and the lens are absent.
  - Every link works as a full page load.

### Mobile layout (G-5)

- **R-39** No route scrolls horizontally from 320 CSS px wide upward. Under 768 px the layout is a single column. A diagram wider than its container scrolls horizontally inside its own container. Diagram labels and blueprint annotations render at 12 CSS px or larger.

### Page metadata and social preview (G-5)

- **R-40** Every route's prerendered HTML has:
  - a unique `<title>`
  - a meta description (the project summary on case studies)
  - `<html lang="en">`
  - a canonical URL `https://gabrielsek.github.io/<path>`, in the form GitHub Pages serves with HTTP 200 without a redirect (with a trailing slash for project routes, if A-1 holds). Every internal link uses the same form.
  - `og:title`, `og:description`, `og:type`
  - `og:url` equal to the canonical URL
  - `og:image`, as the absolute URL of the 1200×630 social image (OQ-6)
  - `twitter:card` = `summary_large_image`.
- **R-41** `/sitemap.xml` lists `/`, `/privacy` and every project route. `/robots.txt` allows all crawlers and points to the sitemap.
- **R-42** The home route includes JSON-LD of type `Person`, with `name`, `jobTitle`, `url`, and `sameAs` holding the LinkedIn and GitHub profile URLs.

### Analytics (G-8)

- **R-43** GoatCounter records one page view for the initial load and one for each client-side route change. Each records the route path without its query string.
- **R-44** GoatCounter records one event named `blueprint-used` per page view, the first time the visitor uses the lens or the toggle on that page.
- **R-45** With GoatCounter blocked or unreachable, every route renders and behaves identically, and the application throws no uncaught error.

### GitHub profile (G-7)

- **R-46** The public repository `gabrielsek/gabrielsek` has a `README.md`. `github.com/gabrielsek` renders it with: the site headline, a link to `https://gabrielsek.github.io`, and a list of the site's projects whose repos are public, each linked. The site's intro and footer link to `github.com/gabrielsek`.
- **R-47** The GitHub profile's website field is `https://gabrielsek.github.io` (an owner action; see Migration & rollback).

### Visual direction (G-2)

- **R-48** **Palettes.** Colours, spacing and type sizes are design tokens. The colour tokens are exactly the 14 named values in the Design plan's palette table (light, dark and blueprint); the blueprint palette is identical in both themes. In all three palettes:
  - contrast is at least 4.5:1 for body text and annotation text
  - contrast is at least 3:1 for large text, UI components, focus indicators and informative strokes
  - the hairline rules are decorative and exempt.

  The owner may change values at the AC-10 approval only if they still pass these thresholds and R-61.
- **R-49** **Type.** Two families and no others:
  - **Archivo**, variable on weight and width, for display and body text
  - **IBM Plex Mono** for annotations, labels, section numbers and code.

  Both are self-hosted WOFF2 subsets covering Latin and Latin Extended-A, at most three font files in total, served with `font-display: swap`. Fallback fonts carry metric overrides matched to each face so the swap doesn't shift layout (R-64). Both families are OFL. "Sęk" must render in the web font (the ę glyph is present in the subset). Body text is at least 16 CSS px. Case-study prose lines run 60 to 80 characters.
- **R-50** **Technical detail.** The finished layer carries:
  - section numbers in monospace ("01", "02")
  - hairline rules between sections
  - large, tight display type for the name, in graphite (or paper ink in the dark theme), never in the accent.

  The full column grid shows only in the blueprint layer.
- **R-51** **Originality.** No asset, layout composition, copy or identity element is taken from the reference sites (D-32); they inform direction only. The owner approves the visual design before the first deploy (AC-10).

### Deliberate design (G-9)

These carry the Design rules into requirements. Each automated check runs in CI as a small script
over the content files or the built output.

- **R-60** **Accent placement.** The three red-pencil values are used only by: the lens ring, the Blueprint toggle's pressed state, blueprint notes, and the focus indicator. A browser scan of computed colours, borders and outlines on every route, in both themes and in blueprint mode, finds them on no other element.
- **R-61** **Surfaces.** The built CSS contains:
  - no `linear-gradient`, `radial-gradient` or `conic-gradient`
  - no `box-shadow` or `text-shadow` other than `none`, and no `backdrop-filter`
  - no `border-radius` other than `0`, except the lens (`50%`)
  - no colour literal outside the token definitions, and no token equal to `#FFFFFF` or above 85 % HSL saturation.

  The blueprint grid is drawn with lines, never dots. The lens ring is a hairline, with no glow or blur. A script over the built CSS checks every item that can be checked from CSS.
- **R-62** **Copy.** These rules cover all site copy (`content/` and the text of the built HTML). Code blocks in case studies are exempt, and so is this spec.
  - **Emoji:** none (Unicode `Extended_Pictographic`), sparkle characters included.
  - **Em dashes:** none in UI strings, meaning `site.yaml`, project titles, summaries and `repoNote`, role headlines, blueprint notes, `privacy.md` headings and component labels. In case-study prose, at most one per 250 words.
  - **"Not X, it's Y":** no sentences built as "it's not X, it's Y", "not just X, but Y" or "isn't X, it's Y". A pattern list in the content validation enforces it, with a reviewed allowlist.

  Content validation fails the build on any breach (validation table).
- **R-63** **Icons.** `package.json` lists no icon library: no `lucide*`, `@fortawesome/*`, `heroicons`, `@material-icons/*`, `@ng-icons/*`, `@tabler/*` or similar. The only marks are LinkedIn's and GitHub's (R-03). Toggles are text. Arrows are type glyphs. A dependency check in CI enforces the list.
- **R-64** **Loading states.**
  - Every route's content is in its prerendered HTML, so no content is fetched after load and no skeleton is needed.
  - Web-font swap causes a Cumulative Layout Shift (CLS) of 0.05 or less on every route.
  - A client-side navigation whose content isn't rendered within 300 ms shows a 2 px graphite progress line at the top of the viewport until it is, with `aria-busy` set on `main`.
  - The Blueprint toggle has its loading and unavailable states (R-09).
- **R-65** **Privacy & terms page.** `/privacy` is prerendered, linked from the footer, and listed in the sitemap. Its content comes from `content/privacy.md`, in two sections.
  - **Privacy** states:
    - who runs the site (the owner, by name)
    - how to reach them (LinkedIn)
    - that GoatCounter counts page views and the `blueprint-used` event without cookies, linking GoatCounter's own privacy policy
    - that the browser stores only the theme choice (`localStorage`) and the hint flag (`sessionStorage`)
    - that nothing else is collected.
  - **Terms of use** states:
    - the copyright of the content
    - the licence of the code (OQ-16)
    - that external links lead to sites the owner doesn't control
    - that the site is provided as is.
- **R-66** **Semantic HTML.**
  - Each route has exactly one `header`, `nav`, `main` and `footer`.
  - Headings descend without skipping a level.
  - Lists are list elements.
  - Every control is a native `button` or `a`, diagram nodes included (or an element with an equivalent role and keyboard handling).
  - The sheet index is a list.

### Privacy and security (detail under Security & compliance)

- **R-52** CI fails the build if a scan finds an email address or a phone number in the built site's HTML text, in `content/`, or in any committed text file. The scan matches any `x@y.tld` pattern and any run of nine or more digits with optional `+`, spaces or dashes. False positives go in a reviewed allowlist file. That file starts with exactly two entries: the noreply commit address and the `x@y.tld` example, both of which appear in this spec.
- **R-53** At runtime, the site requests resources only from its own origin and from GoatCounter's script and count hosts. Observable in the browser's network panel on every route.
- **R-54** A Content-Security-Policy `<meta>` tag allows scripts only from self and GoatCounter's script host, plus hashes for any inline scripts the build emits. The console shows no CSP violation on any route.
- **R-55** No route sets a cookie from any origin. Observable in devtools storage after visiting every route.

### Deploy (G-5, G-6)

- **R-56** A push to `main` runs one workflow with these steps, in order:
  1. install from the lockfile
  2. content validation
  3. lint
  4. unit tests
  5. build and prerender
  6. privacy scan (R-52)
  7. internal link and anchor check
  8. Lighthouse on every route
  9. accessibility scan on every route.

  It deploys to Pages only if every step passes. After deploying, a smoke step requests `/`, one project route and one unknown path from the live URL. It fails the run unless the final status codes are 200, 200 and 404.
- **R-57** Pull requests run the same steps, without deploying.

### Edge cases

| Case | Expected behaviour |
|---|---|
| The pointer leaves the window while the lens shows | Lens hides; it reappears on the next pointer movement over the page (R-08). |
| Window resized while in blueprint mode | Layers stay aligned within 1 px; annotations update within 150 ms after the resize ends (R-07, R-11). |
| Web fonts load after the first measurement | Annotations re-measure once the fonts have loaded (R-11). |
| A keyboard user tabs while the lens shows | Lens hides on the key press; focus indicator is visible (R-08, R-13). |
| A long press on a link on touch | Platform long-press menu; the lens doesn't start (R-10). |
| Theme switched while in blueprint mode | Blueprint unchanged; finished layer switches theme (R-29). |
| The blueprint code fails to load | Site works as finished layer only; the toggle reads "Blueprint unavailable" and is disabled; the hint is hidden (R-06, R-09). |
| The Blueprint toggle is pressed before its code is ready | Loading state; the switch happens once the code is ready (R-09). |
| A client-side navigation is slow (lazy code on a slow network) | Progress line after 300 ms (R-64). |
| `sessionStorage` or `localStorage` throws | Hint stays visible; theme toggle works for the page lifetime; no error (R-06, R-29). |
| A project has no diagram | Allowed. The case study renders without the diagram block. |
| Markdown references an unknown diagram id | Build fails (validation table). |
| A blueprint note references an unknown page or anchor, or a page has more than three notes | Build fails (validation table). |
| `site.featured` names an unknown slug | Build fails. |
| `repo` is null and `repoNote` is missing | Build fails. |
| The JS bundle fails to load (network) | Same as no-JS (R-38). |
| The visitor's clock is earlier than the build date | Years line keeps the build value (R-02). |
| Very long title at 320 px | Wraps; no horizontal page scroll (R-39). |
| The trailing-slash form of a project route is opened | Same content, HTTP 200 (R-35). |
| GoatCounter blocked by an ad blocker | No visible effect (R-45). GoatCounter's author estimates about a third of page views are lost this way, so counts are a lower bound. |

## Data & contracts

### Content inventory

Content is kept apart from layout. All content lives under `content/` in the site repo. Status:
**ready** means the value is known (source given); **draft** means the builder writes it from the
named source and the owner reviews it; **owed** means it comes from the owner (Open questions).

| Section | Data it needs | Launch value / source | Status |
|---|---|---|---|
| Intro | name | "Gabriel Sęk" (CV) | ready |
| Intro | headline | OQ-1 | owed (default given) |
| Intro | location | "Warsaw, Poland" (CV) | ready (OQ-9) |
| Intro | careerStart | 2021-05 (CV: first role's start) | ready |
| Intro | blueprint hint texts | R-06 defaults | draft (OQ-15) |
| Intro / footer | LinkedIn URL | `https://www.linkedin.com/in/gabriel-sek` (CV) | ready |
| Intro / footer | GitHub URL | `https://github.com/gabrielsek` | ready |
| Intro | featured | `null` | ready |
| Projects | chat | title, summary, skills, repo `https://github.com/gabrielsek/chat`, status "In progress" | draft from the chat repo's `SPEC.md` |
| Projects | java-overload-recovery | title, summary, skills, `repo: null`, `repoNote: "Code private until the thesis is published"`, status "Thesis work" | draft from the repo README |
| Case studies | 2 Markdown files, 300–600 words each | OQ-2 | owed (default given) |
| Diagrams | chat architecture | chat-domain, chat-application, chat-starter-spring-boot, chat-app-example, and the six adapters in three variant groups (module names read from the repo on 2026-10-22) | draft (OQ-3) |
| Diagrams | overload testbed | load clients that retry, Spring Boot service, database with an injected slowdown; variant group: platform vs virtual threads (from the repo README) | draft (OQ-3) |
| Blueprint notes | up to 3 per page | OQ-12 | draft |
| More: roles | mBank S.A. | Software Engineer, Warsaw, 2024-11 to present, 5 achievements | draft (trimmed headlines) from the CV |
| More: roles | BEC Financial Technologies | Software Engineer / Team Architect, Warsaw, 2021-05 to 2024-10, 5 achievements | draft (trimmed headlines) from the CV |
| More: roles | role skill ids | Chosen only from the skills list, matching the technologies each role's bullets name; used for validation, not shown as an interface | draft |
| More: skills | 5 technical groups | The CV's groups and items, verbatim: Languages & Frameworks; Architecture & Messaging; Testing & Observability; Tools & Infrastructure; Data & Workflow | ready |
| More: skills | spoken languages | English (C1), Polish (native) | ready |
| More: education | WUT | Warsaw University of Technology, M.Sc. Computer Science, 2023-02 to present, thesis per OQ-4 | ready, apart from the thesis |
| More: education | PJATK | Polish-Japanese Academy of Information Technology, B.Sc. Computer Science, 2018-10 to 2022-07, thesis "Application of Deep Q-Learning (Reinforcement Learning) in Atari Environments" | ready |
| Design | palette, typefaces | Design plan (D-47, D-48) | ready; the owner approves at AC-10 |
| Privacy & terms | `content/privacy.md` | R-65; code licence per OQ-16 | draft |
| Metadata | social image, 1200×630 | OQ-6 | owed (default given) |
| Analytics | GoatCounter site code | OQ-5 | owed (default given) |
| Profile repo | README text | headline, site link, public project links | draft |

### Content files and schemas

YAML for data, because it is prose-friendly and parsed only at build. Markdown for case studies.
Ids are kebab-case. Dates are `YYYY-MM`.

`content/site.yaml`

| Field | Type | Req. | Rule |
|---|---|---|---|
| name | string | yes | |
| headline | string | yes | ≤ 120 chars |
| location | string | yes | |
| careerStart | YYYY-MM | yes | ≤ the earliest role start |
| hint.pointer | string | yes | ≤ 70 chars |
| hint.touch | string | yes | ≤ 70 chars |
| links.linkedin | URL | yes | https |
| links.github | URL | yes | https |
| featured | slug or null | yes | must match a project slug when set |
| siteUrl | URL | yes | `https://gabrielsek.github.io` |
| socialImage | path | yes | file exists, 1200×630 PNG |
| goatcounterCode | string | yes | |

`content/projects.yaml`: a list of:

| Field | Type | Req. | Rule |
|---|---|---|---|
| slug | id | yes | unique; `content/projects/<slug>.md` exists |
| title | string | yes | ≤ 60 chars |
| summary | string | yes | ≤ 160 chars; used on the card and as the meta description |
| status | enum | yes | `in-progress` \| `thesis` \| `complete`; labels "In progress", "Thesis work", "Complete" |
| skills | list of skill ids | yes | ≥ 1; each exists in skills |
| repo | URL or null | yes | https when set |
| repoNote | string | when repo is null | ≤ 80 chars |
| order | integer | yes | unique |

`content/projects/<slug>.md`: Markdown body. Required level-2 headings, in order: Problem, Design,
Decisions, Outcome. A diagram is placed with a fenced block whose info string is `diagram` and
whose content is a diagram id. No raw HTML.

`content/roles.yaml`: a list of:

| Field | Type | Req. | Rule |
|---|---|---|---|
| company | string | yes | |
| title | string | yes | |
| location | string | yes | |
| start | YYYY-MM | yes | |
| end | YYYY-MM or null | yes | null = present; ≥ start |
| skills | list of skill ids | yes | each exists |
| achievements | list of {headline, detail} | yes | ≥ 1; headline ≤ 90 chars |

`content/skills.yaml`: `groups`, a list of {name, items: list of {id, label}}; ids are unique across
groups. `spoken`, a list of {language, level}.

`content/education.yaml`: a list of {institution, degree, field, start, end or null, thesis or
null}.

`content/diagrams/<id>.yaml`

| Field | Type | Req. | Rule |
|---|---|---|---|
| title | string | yes | used as the diagram's accessible name |
| nodes | list of {id, label, kind, description, variant?} | yes | ids unique; `kind` is one of `domain`, `application`, `adapter`, `store`, `broker`, `client`, `service`; `variant` is `<group>.<option>` |
| edges | list of {from, to, label?} | yes | both ends exist |
| variantGroups | list of {id, label, options: list of {id, label}, default} | no | `default` is one of the options; every option has ≥ 1 node |

`content/blueprint-notes.yaml`: a list of:

| Field | Type | Req. | Rule |
|---|---|---|---|
| page | string | yes | `home`, `privacy`, or `projects/<slug>` with an existing slug |
| anchor | string | yes | one of the annotated element names for that page (below) |
| text | string | yes | ≤ 80 chars |

At most three notes per page.

**Annotated element names.** This is the fixed contract between the blueprint notes and the
components; the README repeats it.

| Page | Names |
|---|---|
| home | `name`, `headline`, `facts`, `links`, `hint`, `projects-grid`, `project-card-<slug>`, `more-roles`, `more-skills`, `more-education`, `footer` |
| projects/<slug> | `sheet-number`, `title`, `title-block`, `body`, `diagram-<id>`, `project-nav` |
| privacy | `title`, `body` |

`content/privacy.md`: Markdown with two required level-2 headings, in order: Privacy, Terms of use.
No raw HTML.

### Build-time validation

The content script fails the build on any of the following. The error message names the file and
the field.

| Rule | Example message |
|---|---|
| Required field missing or of the wrong type | `projects.yaml[1].summary: required` |
| Length or format limit exceeded | `projects.yaml[0].summary: 171 chars, max 160` |
| Duplicate slug, skill id, node id or `order` | `skills.yaml: duplicate id 'kafka'` |
| Unknown skill id referenced | `roles.yaml[0].skills: unknown 'kafkaa'` |
| Project without its Markdown file, or Markdown missing a required heading | `projects/chat.md: missing heading 'Decisions'` |
| Unknown diagram id in Markdown, or edge to an unknown node | `diagrams/chat-architecture.yaml edges[3].to: unknown node 'jpa'` |
| `repo` null without `repoNote` | `projects.yaml[1].repoNote: required when repo is null` |
| `end` before `start`; `careerStart` after the earliest start | `roles.yaml[1]: end before start` |
| `featured` set to an unknown slug | `site.yaml.featured: unknown project 'foo'` |
| Blueprint note on an unknown page or anchor | `blueprint-notes.yaml[2].anchor: unknown 'hero' for page 'home'` |
| More than three notes on a page | `blueprint-notes.yaml: page 'home' has 4 notes, max 3` |
| `privacy.md` missing or missing a required heading | `privacy.md: missing heading 'Terms of use'` |
| Emoji anywhere in site copy (R-62) | `projects.yaml[0].summary: emoji U+2728 not allowed` |
| Em dash in a UI string (R-62) | `site.yaml.headline: em dash not allowed in UI strings` |
| More than one em dash per 250 words of case-study prose (R-62) | `projects/chat.md: 4 em dashes in 610 words, max 2` |
| "Not X, it's Y" construction (R-62) | `projects/chat.md:14: construction "not just ... but" not allowed` |

### Generated output

The content script writes typed modules, the prerender route list and `sitemap.xml` input into a
git-ignored folder before `ng build`. Nothing generated is committed. The project data is the single
source for cards, routes, the sitemap and previous/next order.

### Versioning and compatibility

n/a. The content files have exactly one consumer, which lives in the same repository. A schema
change and its consumer change in the same commit.

### Adding a project (G-6)

- **R-58** Adding a project takes four steps:
  1. Add an entry to `content/projects.yaml`.
  2. Add `content/projects/<slug>.md`.
  3. Optionally add `content/diagrams/<id>.yaml` and up to three notes in `content/blueprint-notes.yaml`.
  4. Push.

  Without any file outside `content/` changing, the result is:
  - a card on home, in `order`
  - a prerendered `/projects/<slug>`
  - a sitemap entry
  - correct previous/next links
  - a blueprint layer for the new page, with measured annotations.
- **R-59** The repo README has an "Adding a project" section with these steps, a complete example entry, and the annotated element names.

## Changes to binding documents

None. The repository has no `docs/workflow.md` and no binding documents.

## Failure semantics

- Timeout / retry: there are no runtime calls apart from the GoatCounter beacon, which is async, fire-and-forget and never retried. The build has no network calls besides dependency install.
- Idempotency key: n/a at runtime. A deploy is repeatable: re-running the workflow on the same commit produces the same site, except the build-date value of the years line (R-02).
- Partial failure end state:
  - If any CI step fails, the deploy job does not run and the previously deployed site stays live.
  - If content validation fails, the Angular build never starts.
  - Assumed (A-1): a Pages deployment replaces the whole site at once.
- Poison / DLQ: n/a. No queues or messages.
- Client-side degradations:
  - JS fails to load: R-38.
  - The blueprint code fails to load or throws: the finished layer works on its own. The toggle reads "Blueprint unavailable" and is disabled, and the hint is hidden (R-06, R-09).
  - `localStorage` or `sessionStorage` fails: R-06, R-29.
  - GoatCounter fails: R-45.

## Security & compliance

- Authn / authz: none. A public static site. Write access to both repositories is the owner's only.
- Personal data in scope: the owner's name, city, LinkedIn URL, GitHub username, employment and education history. All of it is deliberately public. The site stores no visitor data. `localStorage` holds only the theme choice, and `sessionStorage` only the hint-dismissed flag. GoatCounter's own privacy policy governs what it collects. No visitor data reaches the repository.
- Must never appear (page, source, repo, spec, commit metadata): the owner's email address and phone number (R-05, R-52). Commits use the noreply identity `Gabriel <49335488+gabrielsek@users.noreply.github.com>`.
- Retention / audit: GoatCounter retention per its terms. Git history is the change audit.
- Secrets: none. The Pages deploy uses the workflow's built-in token with Pages permissions only. The GoatCounter site code is public by design.
- Supply chain: dependency versions pinned and the lockfile committed. Third-party Actions pinned to full commit SHAs. The graph-layout and Markdown libraries are build-only dev dependencies.
- Licences: Archivo and IBM Plex Mono are OFL (R-49). The code licence is OQ-16. No asset from the reference sites (R-51). Brand marks follow each brand's guidelines (A-15).
- Privacy notice: `/privacy` (R-65). Whether it meets GDPR transparency duties for a personal site with LinkedIn as the only contact channel is unverified (A-14).
- CSP: R-54. Cookies: none (R-55).

## Observability

- Logs: build and check logs in GitHub Actions. The production app writes nothing to the console.
- Metrics:
  - GoatCounter page views per path, and referrers.
  - The `blueprint-used` event (R-44). Its ratio to home page views shows whether visitors find the idea, which is the bet this revision makes.
  - All counts are a lower bound (see Edge cases).
- Alerts: none at runtime; a personal site has no on-call. A failed workflow shows as a red run, and GitHub notifies about failed runs (A-10).
- How you know it works in production:
  - the post-deploy smoke step (R-56)
  - GoatCounter shows page views and `blueprint-used` events.

## Migration & rollback

- Data migration / backfill: n/a, greenfield.
- Feature flag: none. The only switchable element is the featured-project slot, which is data (`site.featured`).
- Rollback: `git revert` the bad commit and push; the workflow redeploys. Alternatively, re-run the deploy workflow for the last good commit.
- Owner actions at launch. These are account actions that the builder cannot perform:
  1. Create the public repos `gabrielsek/gabrielsek.github.io` and `gabrielsek/gabrielsek`.
  2. Set Pages to deploy from GitHub Actions.
  3. Create the GoatCounter account (OQ-5).
  4. Set the profile website field (R-47).
  5. Pin the public project repos on the profile.
  6. Supply or waive the anti-slop rules (OQ-11).
  7. Approve copy, diagrams, notes and visuals (AC-10).
- Adding a custom domain later: OQ-8.

## Acceptance criteria

### Quality bars

- [ ] **AC-01** (G-5) Lighthouse runs in CI on `/`, each project route and the not-found view, at its default mobile emulation and throttling, against the built output. Every route meets: Performance ≥ 90, Accessibility = 100, Best Practices ≥ 95, SEO = 100, and Cumulative Layout Shift ≤ 0.05. The not-found view is exempt from SEO because it is `noindex`. A miss fails the build.
- [ ] **AC-02** (G-5) The first load of any route transfers ≤ 500 KB compressed, counting every request until network idle (the blueprint code included) and excluding lazy-loaded images below the fold. A miss fails the build.
- [ ] **AC-03** (G-5) An automated accessibility scan finds zero violations of WCAG 2.0, 2.1 and 2.2 A and AA rules. It runs on every route, in light and dark themes, in four states: default, a role expanded, a diagram node selected, and blueprint mode on.
- [ ] **AC-04** (G-5) A manual WCAG 2.2 AA pass, recorded as a checklist in the repo, covers:
  - every interaction completed by keyboard alone
  - a focus indicator that is visible in both layers and never hidden by sticky elements or the lens
  - the Blueprint toggle as the non-drag alternative to the lens (SC 2.5.7)
  - no single-key shortcuts (SC 2.1.4)
  - reflow at 320 CSS px with no loss of content
  - 200 % text zoom
  - contrast measured in all three palettes, annotations and diagram labels included
  - target size of at least 24×24 CSS px
  - R-32 (reduced motion)
  - a screen-reader smoke test (NVDA on Windows, and VoiceOver on iOS Safari): the intro, the Blueprint toggle's state, role expand, a diagram's text alternative, and confirmation that blueprint text is never read out.
- [ ] **AC-05** (G-5) Every R-requirement holds on the latest two major versions of Chrome, Edge, Firefox and Safari (macOS and iOS).

### Verification of the functional requirements

- [ ] **AC-06** (G-1…G-8) Each R-requirement passes the check named for it below. A test counts only if it went red against a deliberately broken implementation first.

| Requirements | Verified by |
|---|---|
| R-01 to R-04, R-06, R-15 to R-19, R-23, R-29, R-31, R-33, R-34, R-37, R-43, R-44, R-45 | Automated browser tests in CI. Viewports 360×640 and 1366×768. R-02 uses a mocked clock. R-45 blocks GoatCounter's hosts. R-43 and R-44 intercept the count requests and assert path and event name. |
| R-07, R-11, R-12, R-21 | Browser tests at 320, 360, 768 and 1366 px, before and after a resize. For every annotated element: the blueprint box matches the finished box within 1 px, and the displayed values equal the computed ones, rounded. Each note from a fixture appears next to its anchor in the blueprint layer and nowhere in the finished layer. |
| R-08, R-09, R-10, R-13 | Browser tests: the lens follows a synthetic pointer and hides on leave and on keydown. A click under the lens reaches the element. The toggle exposes its pressed state and survives a route change. A touch hold of 300 ms shows the lens while a swipe scrolls. The accessibility tree contains no blueprint text. |
| R-14 | Performance trace in CI with 4× CPU throttling and five seconds of scripted lens movement: no task over 50 ms. LCP compared with and without the blueprint code. |
| R-05, R-52 | Privacy scan in CI, plus a test that plants an email in a fixture and expects the build to fail. |
| R-20, R-24, R-38 | Browser tests with JS disabled, plus an HTML assertion on the prerendered files. |
| R-22, R-25, R-26, R-27, R-28, R-40, R-41, R-42 | Assertions on the prerendered HTML and `sitemap.xml` / `robots.txt`. |
| R-30, R-32 | Browser test: no element starts hidden pending an animation. With reduced motion emulated, computed transition durations are 0 and the lens has no trailing. |
| R-35, R-36 | The workflow's post-deploy smoke step (status codes), plus a browser test of refreshes on the live URL. |
| R-39 | Browser test at 320 px: the document is no wider than the viewport. Diagram label and annotation font sizes are ≥ 12 px. |
| R-46, R-47 | Manual: open `github.com/gabrielsek` signed out. |
| R-48, R-49, R-50, R-51 | Contrast checks in AC-03 and AC-04. A font check: the computed font of the intro name is the web font. Owner review (AC-10) for R-50 and R-51. |
| R-53, R-54, R-55 | Browser test recording network origins, console CSP errors and cookies on every route. |
| R-56, R-57 | A pull request with a deliberately failing check shows a red run. A push with a failing check to a test branch configured like `main` produces no deployment. |
| R-58, R-59 | A fixture project added under `content/` in a test build yields its card, route, sitemap entry and blueprint layer, with `git diff --stat` showing only `content/` changed. |
| R-60 | Browser scan of computed colours, borders and outlines on every route, in both themes and in blueprint mode, against the allowed element list. |
| R-61 | CSS-check script over the built CSS, plus a test that plants a `box-shadow` in a fixture and expects the build to fail. |
| R-62 | Content-validation rules (validation table), one unit test each. |
| R-63 | Dependency-check script over `package.json`, plus a test that adds `lucide` to a fixture manifest and expects failure. |
| R-64 | Lighthouse CLS (AC-01). A browser test with network throttling and a delayed route chunk: the progress line appears after 300 ms and clears once content renders. A browser test that blocks the blueprint code: the toggle shows loading, then unavailable. |
| R-65 | Assertions on the prerendered `/privacy` HTML: both headings present, a footer link on every route, a sitemap entry. |
| R-66 | Assertions on the prerendered HTML of every route: landmark counts, heading order, list markup, and no click handler on a non-interactive element. |
| Validation table | One unit test per rule, asserting the exact error message. |

### Definition of done for the build

- [ ] **AC-07** (G-5) The lint step passes with zero errors.
- [ ] **AC-08** (G-5) The unit tests pass. These cover the content script: every validation rule, the Markdown conversion and the diagram layout output. They also cover the years computation, annotation value formatting, the touch-hold threshold logic and theme persistence.
- [ ] **AC-09** (G-7) `https://gabrielsek.github.io` serves the site, and `github.com/gabrielsek` shows the profile README linking to it.
- [ ] **AC-10** (G-1, G-2, G-3) Before the first deploy to `main`, the owner has approved: the headline, the hint texts, both case-study texts, both diagrams, the blueprint notes, the trimmed achievement headlines, the palette and typefaces, the visual design in all three palettes, the Privacy & terms text, and the social image.
- [ ] **AC-11** (G-9) Before the first deploy to `main`, a design audit of the built site scores every row of the Design rules table, with one line of evidence each, and is committed as `docs/design-audit.md`. The target is 0 boxes checked. The first deploy is blocked if more than 2 are checked, and every checked box needs a written justification that the owner accepts. The audit is repeated whenever a change touches the palette, type, layout or motion.

### Traceability

| Goal | Requirements and criteria |
|---|---|
| G-1 | R-01 to R-06, AC-10 |
| G-2 | R-06 to R-14, R-21, R-48 to R-51, AC-06, AC-10 |
| G-3 | R-15 to R-21, AC-10 |
| G-4 | R-22 to R-26 |
| G-5 | R-27 to R-42, R-48, R-49, R-52 to R-57, AC-01 to AC-08 |
| G-6 | R-56 to R-59 |
| G-7 | R-46, R-47, AC-09 |
| G-8 | R-43 to R-45, R-55 |
| G-9 | R-15, R-31, R-60 to R-66, AC-11 |

## Assumptions

Each of these was inferred or is unverified. The breakdown must re-check them.

- **A-1** GitHub Pages:
  - serves `404.html` with HTTP 404 for unknown paths
  - serves `<dir>/index.html` for `/<dir>/` and redirects `/<dir>` to it
  - swaps a deployment atomically.

  Unverified this session.
- **A-2** `@angular/ssr` 22.2.3 exists (read from npm on 2026-10-22). Unverified for 22.x: that it can prerender every route, including parameterised ones whose params come from the content script, to static files without a running server. Also unverified: how Angular 22 emits inline scripts, which matters for R-54.
- **A-3** GoatCounter's script loads from `gc.zgo.at` (verified in its FAQ snippet), and its docs list an "Events" page (heading seen, page not read). Unverified:
  - the count endpoint is `https://<code>.goatcounter.com/count`
  - its SPA guide covers counting route changes as R-43 requires
  - its events support covers R-44.
- **A-4** GoatCounter's FAQ says a GDPR consent notice is "probably" not needed. Whether that is legally sufficient for this site is unverified.
- **A-5** java-overload-recovery is the M.Sc. thesis project. Inferred from its README ("Go/no-go demo for a thesis on overload recovery").
- **A-6** The chat diagram draft uses the module names read from the repo's top level on 2026-10-22. Their actual dependencies (the edges) have not been read.
- **A-7** `@dagrejs/dagre` 3.1.1 and `elkjs` 0.12.0 are candidates for build-time layout (sizes read from npm). Neither was checked for whether it runs under Node in the content script without a DOM.
- **A-8** "Whole years" in R-02 means full calendar years since the 1st of the `careerStart` month.
- **A-9** The free tier of a public user-site repo can deploy Pages from GitHub Actions.
- **A-10** GitHub notifies the commit author when a workflow run fails, under default notification settings. Unverified.
- **A-11** CSS masking with a moving radial mask (or an equivalent `clip-path`) works unprefixed on the latest two versions of every browser in AC-05. Unverified; Safari has historically needed `-webkit-` mask properties.
- **A-12** Measuring every annotated element after a resize (layout reads, no writes in between) fits inside 150 ms on a mid-range phone. Unverified until profiled; R-14 and R-11 are the checks.
- **A-13** Laying the blueprint layer exactly over the finished layer, by rendering the same components twice with different styles, keeps the boxes within 1 px. Hollow display type (outlined text) can change glyph metrics slightly. Unverified until R-07's test runs.
- **A-14** A privacy notice that names the owner and gives LinkedIn as the only contact channel meets GDPR transparency duties for a personal, non-commercial site. Unverified; this is not legal advice.
- **A-15** LinkedIn's and GitHub's brand guidelines allow their marks in a single ink colour at icon size. Unverified.
- **A-16** Google's `latin-ext` subset of Archivo and IBM Plex Mono includes ę (U+0119). Inferred from the subset name; R-49's font check confirms it.

## Open questions

Each one has a default. The build proceeds with the default unless the owner answers otherwise.

- **OQ-1 Headline copy.** Default: "Backend engineer building event-driven systems for banking. Java, Spring Boot, Kafka." It is condensed from the CV summary and contains no em dash (R-62). The owner edits `site.headline`.
- **OQ-2 Case-study text** for chat and java-overload-recovery (Problem, Design, Decisions, Outcome; 300–600 words each). Default: the builder drafts both, from chat's `SPEC.md` and java-overload-recovery's README and testbed spec. The owner approves before the first deploy (AC-10). Launch is blocked until approval.
- **OQ-3 Diagram content.** Default: the builder drafts both diagrams from the repos, as the Content inventory describes. The owner checks nodes, edges and descriptions before the first deploy.
- **OQ-4 M.Sc. thesis line in Education.** Default: no thesis line until the topic is final.
- **OQ-5 GoatCounter site code.** Default: `gabrielsek`. The owner creates the account. GoatCounter requires a verified email, which stays in the GoatCounter account and never reaches the site.
- **OQ-6 Social preview image.** Default: a 1200×630 PNG of the intro screen split down the middle, with the finished design on the left and the blueprint on the right. It shows the name, headline and site URL, and is committed to the repo.
- **OQ-7 Status labels.** Default: chat "In progress", java-overload-recovery "Thesis work".
- **OQ-8 Custom domain.** Default: none at launch; the site is served at `https://gabrielsek.github.io`. Revisit if the owner buys a domain. Then add the domain in Pages settings and DNS, and update `site.siteUrl`, the canonical URLs and the sitemap. Unverified: that GitHub redirects the `github.io` address to the custom domain.
- **OQ-9 Location on the intro screen.** Default: shown ("Warsaw, Poland"), because recruiters filter by location and the CV already states it.
- **OQ-10 Profile README wording.** Default: the builder drafts it (headline, site link, public project links); the owner approves.
- **OQ-11 Anti-slop rules.** Resolved in revision 3: see Design plan, Design rules, D-46 to D-54, R-60 to R-66 and AC-11.
- **OQ-12 Blueprint notes.** Default: the builder drafts up to three notes per page about real design decisions on that page (for example, why there is one accent). The owner approves (AC-10).
- **OQ-13 Accent colour.** Default: red pencil (`#B8321A` light, `#E0603F` dark, `#F49A7E` blueprint), from the Design plan. The owner approves at AC-10.
- **OQ-14 Typefaces.** Default: Archivo and IBM Plex Mono, from the Design plan (D-48). The owner approves at AC-10.
- **OQ-15 Hint texts.** Default: "Move the pointer to see how this page is built" and "Press and hold to look inside".
- **OQ-16 Licence.** Default: code under the MIT licence (a `LICENSE` file in the repo); content (text, diagrams, notes) © the owner, all rights reserved, as `/privacy` states.
- **OQ-17 Privacy & terms wording.** Default: the builder drafts `content/privacy.md` from R-65; the owner approves at AC-10. Its legal sufficiency is A-14.

## Provenance

- Interviewed on 2026-10-22. Mode: greenfield; neither repository existed on GitHub on 2026-10-22. Branch: n/a.
- **Revision 1:** 8 rounds, a content-led portfolio.
- **Revision 2:** same day, after the owner's feedback on the first draft. The emphasis moved to design and one idea (D-32 to D-45). The blueprint layer was added (R-06 to R-14, R-21). The skill filter was removed. Tone and motion were replaced (D-13, D-15). The requirements were renumbered.
- **Revision 3:** 2026-10-24, after the owner supplied the anti-slop rules. Added the Design plan, the Design rules table, D-46 to D-54, R-60 to R-66, AC-11 and G-9. Changed R-03, R-06, R-09, R-15 to R-17, R-27, R-28, R-31, R-35, R-41 and R-48 to R-50. Requirement numbers from revision 2 are unchanged.
  - Palette contrast and saturation were computed by a script implementing the WCAG 2 relative-luminance formula and HSL saturation; all ten pairs in the Design plan passed.
  - Font licences, axes and subsets were read from `ofl/archivo/METADATA.pb` and `ofl/ibmplexmono/METADATA.pb` in the google/fonts repository during revision 3.
  - The icon-package names in R-63 were confirmed on npm.
- Grounded in:
  - the owner's CV (PDF, 2026 version) for roles, dates, skills and education
  - npm registry versions read on 2026-10-22: `@angular/core`, `@angular/ssr` and `@angular/cli` 22.2.3; `astro` 7.4.2; `@sveltejs/kit` 3.1.0; `@sveltejs/adapter-static` 4.0.0; `svelte` 5.57.2; `vite` 8.3.5; `@analogjs/platform` and `@analogjs/content` 2.9.1 (peer range includes Angular 22); `@dagrejs/dagre` 3.1.1; `elkjs` 0.12.0
  - the GitHub REST API (5 public repos; empty profile bio and website)
  - the local READMEs and top-level listings of chat, java-overload-recovery and durable-agent-runtime
  - goatcounter.com's home page and FAQ, read on 2026-10-22.
- Reference sites viewed on 2026-10-22 in the built-in browser: the ten listed in D-32. The browser pane was hidden, so the WebGL-heavy sites (geex-arts, laxspace, madeinuxstudio) were judged from page text rather than rendering. Examples gathered for the concept choice:
  - [Speckyboy's X-ray effect roundup](https://speckyboy.com/looking-through-the-use-of-x-ray-effects/) (opened)
  - [Marga Navarro's mask reveal on Awwwards](https://www.awwwards.com/inspiration/hover-trails-and-mask-reveal-on-cursor-move-marga-navarro) (opened)
  - [Oatly Infinite Canvas](https://www.awwwards.com/sites/oatly-infinite-canvas) (title loaded)
  - [paodao.fr](https://paodao.fr) (via its webgpu.com write-up)
  - [bruno-simon.com](https://bruno-simon.com/) (opened).
- The spec template is the bundled `sek` 0.9.0 template. The repository has no `docs/workflow.md`.
