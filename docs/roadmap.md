---
layout: default
title: xdoubledot Roadmap Slide — DRAFT
---

# xdoubledot Roadmap Slide — DRAFT

> **DRAFT — NEEDS MAX & TIZ REVIEW FOR DATES**

---

## 1. Plain-Text Content

### Slide Title
**Roadmap** — From Lab to Orbit

### Subtitle / framing line
> "From algorithm to thruster to orbit — four phases, four years."

---

### Phase 0 — Foundation  *(NOW · Q1–Q2 2026)*
**Status: CURRENT / COMPLETE**

- Aegis RL control algorithm validated in simulation
- Pitch deck live; investor conversations initiated
- ESA BIC Estonia application submitted
- Company incorporated: xdoubledot OÜ

**Key Milestone:** ESA BIC application submitted
**Deliverable:** Aegis RL v1.0 validated, company fully operational

---

### Phase 1 — Incubation Year 1  *(Q3 2026 – Q2 2027)*
**Status: PLANNED**

- ESA BIC incubation begins (assumed Q3 2026 — **CONFIRM DATE**)
- Thruster magnetic circuit design finalised
- Coil procurement and winding
- Bench-test rig assembled (vacuum-free benchtop)
- HallThruster.jl simulation validation against published data
- First Krypton gas discharge achieved (benchtop)

**Key Milestone:** First Krypton discharge
**Deliverable:** Validated simulation model + benchtop discharge data

---

### Phase 2 — Incubation Year 2  *(Q3 2027 – Q2 2028)*
**Status: PLANNED**

- Full prototype thruster assembly
- Vacuum chamber testing (facility access — **CONFIRM PARTNER/FACILITY**)
- TRL progression: 4 → 5
- Aegis RL closed-loop control demonstrated on hardware
- First customer Letter of Intent (LOI) signed

**Key Milestone:** TRL 5 + first customer LOI
**Deliverable:** Prototype test report, LOI, seed funding close target

---

### Phase 3 — Qualification & Commercialisation  *(2029+)*
**Status: TARGET**

- TRL 6 qualification campaign
- Environmental testing (vibration, thermal, EMC)
- First commercial contract signed
- In-orbit demonstration target (hosted payload or small sat)
- Series A fundraise

**Key Milestone:** First commercial contract + in-orbit demo
**Deliverable:** Flight-qualified thruster unit, revenue

---

## 2. HTML Snippet

Paste this `<section>` block into `xdoubledot_pitch_deck.html` at the desired position.
Add a corresponding `<div class="nav-dot">` entry to the `.footer-nav` and update the `XX / YY` slide-number counters throughout.

```html
<!-- Roadmap Slide — DRAFT, needs Max & Tiz date review -->
<section class="slide" id="slide-roadmap">
    <span class="slide-number">XX / YY</span>

    <h3>Roadmap</h3>
    <h2>From Algorithm to Orbit</h2>

    <!-- 4-column phase grid -->
    <div style="
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: 20px;
        margin-top: 40px;
        position: relative;
        z-index: 1;
    ">

        <!-- Connecting line behind cards -->
        <!-- (purely decorative — drawn via the card borders and accent colours) -->

        <!-- Phase 0 — CURRENT -->
        <div style="
            background: linear-gradient(135deg, rgba(0, 212, 255, 0.18) 0%, rgba(0, 212, 255, 0.05) 100%);
            border: 2px solid var(--accent);
            border-radius: 16px;
            padding: 28px 24px;
            position: relative;
            transition: all 0.3s ease;
        ">
            <!-- "NOW" badge -->
            <div style="
                position: absolute;
                top: -12px;
                left: 50%;
                transform: translateX(-50%);
                background: var(--accent);
                color: #000;
                font-family: 'Space Grotesk', sans-serif;
                font-size: 0.7rem;
                font-weight: 700;
                letter-spacing: 2px;
                padding: 3px 12px;
                border-radius: 20px;
                text-transform: uppercase;
                white-space: nowrap;
            ">NOW</div>

            <div style="
                font-family: 'Space Grotesk', sans-serif;
                font-size: 0.75rem;
                font-weight: 600;
                color: var(--accent);
                text-transform: uppercase;
                letter-spacing: 3px;
                margin-bottom: 6px;
            ">Phase 0</div>

            <div style="
                font-family: 'Space Grotesk', sans-serif;
                font-size: 1.1rem;
                font-weight: 600;
                color: var(--text);
                margin-bottom: 4px;
            ">Foundation</div>

            <div style="
                font-size: 0.8rem;
                color: var(--accent);
                font-weight: 500;
                margin-bottom: 16px;
            ">Q1–Q2 2026</div>

            <ul style="
                list-style: none;
                padding: 0;
                margin: 0 0 20px 0;
            ">
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">✓</span>
                    Aegis RL validated
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">✓</span>
                    Pitch deck live
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">✓</span>
                    ESA BIC applied
                </li>
            </ul>

            <div style="
                background: rgba(0, 212, 255, 0.12);
                border-left: 3px solid var(--accent);
                padding: 10px 12px;
                border-radius: 0 8px 8px 0;
            ">
                <div style="font-size: 0.7rem; color: var(--accent); font-weight: 600; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 2px;">Key Milestone</div>
                <div style="font-size: 0.82rem; color: var(--text); font-weight: 500;">ESA BIC application submitted</div>
            </div>
        </div>

        <!-- Phase 1 — Incubation Year 1 -->
        <div style="
            background: var(--surface);
            border: 1px solid rgba(255, 255, 255, 0.10);
            border-radius: 16px;
            padding: 28px 24px;
            transition: all 0.3s ease;
        ">
            <div style="
                font-family: 'Space Grotesk', sans-serif;
                font-size: 0.75rem;
                font-weight: 600;
                color: var(--accent);
                text-transform: uppercase;
                letter-spacing: 3px;
                margin-bottom: 6px;
            ">Phase 1</div>

            <div style="
                font-family: 'Space Grotesk', sans-serif;
                font-size: 1.1rem;
                font-weight: 600;
                color: var(--text);
                margin-bottom: 4px;
            ">Incubation — Year 1</div>

            <div style="
                font-size: 0.8rem;
                color: var(--text-muted);
                font-weight: 500;
                margin-bottom: 16px;
            ">Q3 2026 – Q2 2027</div>

            <ul style="
                list-style: none;
                padding: 0;
                margin: 0 0 20px 0;
            ">
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    Thruster design finalised
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    Coil procurement &amp; winding
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    HallThruster.jl validated
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    Bench-test rig assembled
                </li>
            </ul>

            <div style="
                background: rgba(255, 255, 255, 0.04);
                border-left: 3px solid rgba(0, 212, 255, 0.4);
                padding: 10px 12px;
                border-radius: 0 8px 8px 0;
            ">
                <div style="font-size: 0.7rem; color: var(--accent); font-weight: 600; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 2px;">Key Milestone</div>
                <div style="font-size: 0.82rem; color: var(--text); font-weight: 500;">First Krypton discharge</div>
            </div>
        </div>

        <!-- Phase 2 — Incubation Year 2 -->
        <div style="
            background: var(--surface);
            border: 1px solid rgba(255, 255, 255, 0.10);
            border-radius: 16px;
            padding: 28px 24px;
            transition: all 0.3s ease;
        ">
            <div style="
                font-family: 'Space Grotesk', sans-serif;
                font-size: 0.75rem;
                font-weight: 600;
                color: var(--accent);
                text-transform: uppercase;
                letter-spacing: 3px;
                margin-bottom: 6px;
            ">Phase 2</div>

            <div style="
                font-family: 'Space Grotesk', sans-serif;
                font-size: 1.1rem;
                font-weight: 600;
                color: var(--text);
                margin-bottom: 4px;
            ">Incubation — Year 2</div>

            <div style="
                font-size: 0.8rem;
                color: var(--text-muted);
                font-weight: 500;
                margin-bottom: 16px;
            ">Q3 2027 – Q2 2028</div>

            <ul style="
                list-style: none;
                padding: 0;
                margin: 0 0 20px 0;
            ">
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    Prototype thruster built
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    Vacuum chamber testing
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    TRL 4 → 5
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    Aegis RL on hardware
                </li>
            </ul>

            <div style="
                background: rgba(255, 255, 255, 0.04);
                border-left: 3px solid rgba(0, 212, 255, 0.4);
                padding: 10px 12px;
                border-radius: 0 8px 8px 0;
            ">
                <div style="font-size: 0.7rem; color: var(--accent); font-weight: 600; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 2px;">Key Milestone</div>
                <div style="font-size: 0.82rem; color: var(--text); font-weight: 500;">TRL 5 + first customer LOI</div>
            </div>
        </div>

        <!-- Phase 3 — Commercialisation -->
        <div style="
            background: var(--surface);
            border: 1px solid rgba(255, 255, 255, 0.10);
            border-radius: 16px;
            padding: 28px 24px;
            transition: all 0.3s ease;
        ">
            <div style="
                font-family: 'Space Grotesk', sans-serif;
                font-size: 0.75rem;
                font-weight: 600;
                color: var(--accent);
                text-transform: uppercase;
                letter-spacing: 3px;
                margin-bottom: 6px;
            ">Phase 3</div>

            <div style="
                font-family: 'Space Grotesk', sans-serif;
                font-size: 1.1rem;
                font-weight: 600;
                color: var(--text);
                margin-bottom: 4px;
            ">Qualification &amp; Commercialisation</div>

            <div style="
                font-size: 0.8rem;
                color: var(--text-muted);
                font-weight: 500;
                margin-bottom: 16px;
            ">2029+</div>

            <ul style="
                list-style: none;
                padding: 0;
                margin: 0 0 20px 0;
            ">
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    TRL 6 qualification
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    Enviro. testing (vib/thermal)
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    Series A fundraise
                </li>
                <li style="font-size: 0.85rem; color: var(--text-muted); padding: 5px 0; padding-left: 16px; position: relative;">
                    <span style="position: absolute; left: 0; color: var(--accent);">›</span>
                    In-orbit demo target
                </li>
            </ul>

            <div style="
                background: rgba(255, 255, 255, 0.04);
                border-left: 3px solid rgba(0, 212, 255, 0.4);
                padding: 10px 12px;
                border-radius: 0 8px 8px 0;
            ">
                <div style="font-size: 0.7rem; color: var(--accent); font-weight: 600; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 2px;">Key Milestone</div>
                <div style="font-size: 0.82rem; color: var(--text); font-weight: 500;">First commercial contract + in-orbit demo</div>
            </div>
        </div>

    </div><!-- end grid -->

    <!-- Bottom progress bar / timeline strip -->
    <div style="
        margin-top: 30px;
        display: flex;
        align-items: center;
        gap: 0;
        position: relative;
        z-index: 1;
    ">
        <!-- Phase 0 segment — filled/active -->
        <div style="flex: 1; height: 4px; background: var(--accent); border-radius: 2px 0 0 2px;"></div>
        <div style="width: 10px; height: 10px; background: var(--accent); border-radius: 50%; flex-shrink: 0;"></div>
        <!-- Phase 1 segment -->
        <div style="flex: 1; height: 4px; background: rgba(0, 212, 255, 0.25);"></div>
        <div style="width: 10px; height: 10px; background: rgba(0, 212, 255, 0.25); border-radius: 50%; flex-shrink: 0;"></div>
        <!-- Phase 2 segment -->
        <div style="flex: 1; height: 4px; background: rgba(0, 212, 255, 0.15);"></div>
        <div style="width: 10px; height: 10px; background: rgba(0, 212, 255, 0.15); border-radius: 50%; flex-shrink: 0;"></div>
        <!-- Phase 3 segment -->
        <div style="flex: 1; height: 4px; background: rgba(0, 212, 255, 0.08); border-radius: 0 2px 2px 0;"></div>
        <div style="width: 10px; height: 10px; background: rgba(0, 212, 255, 0.08); border-radius: 50%; flex-shrink: 0;"></div>
    </div>

</section>
<!-- end Roadmap Slide -->
```

---

## 3. NEEDS REVIEW

> **Max and Tiz must confirm the following before this slide goes into the live deck.**

### Dates

- [ ] **ESA BIC incubation start date** — assumed Q3 2026. Confirm the actual acceptance/start date once notified.
- [ ] **Phase 1 end / Phase 2 start boundary** — assumed clean split at Q2/Q3 2027. Does the ESA BIC programme run for exactly 24 months, or is there a different cadence?
- [ ] **Phase 3 anchor year** — "2029+" is intentionally vague. Do you want a more specific target (e.g. "2029–2030") or keep it open-ended for investor conversations?
- [ ] **Company incorporation date** — currently implied as already complete. Confirm xdoubledot OÜ is formally registered before the ESA BIC application deadline.

### Milestone Accuracy

- [ ] **Aegis RL "validated"** — what is the accepted definition of validation here? Simulation convergence only, or comparison against published HET data? Clarify for due-diligence audiences.
- [ ] **"First Krypton discharge"** — is this a benchtop atmospheric discharge or does it require a vacuum chamber? If vacuum is needed for even the first discharge, Phase 1 timing may need adjustment and a facility partner should be named.
- [ ] **Vacuum chamber access in Phase 2** — no facility partner is named. Do you have a university/institute relationship (e.g. TUT, Aalto, ESA ESTEC, DLR) lined up? Investors will ask.
- [ ] **TRL 4 → 5 in Phase 2** — TRL 4 is "component validated in lab environment"; TRL 5 is "component validated in relevant environment." Confirm the ESA/NATO TRL definition you are using and that vacuum chamber testing satisfies TRL 5 criteria for your thruster class.
- [ ] **"First customer LOI" in Phase 2** — do you have any warm prospect conversations that make this realistic by mid-2028? If not, this is a risk flag investors will probe.
- [ ] **In-orbit demonstration target** — no launch vehicle, rideshare partner, or satellite host identified. Add a note or placeholder once known; leaving it unspecified is fine at pitch stage but flag it in the notes.
- [ ] **HallThruster.jl** — confirm this is the open-source Julia package by the JPL/academic community and that your use of it for validation is documented.

### Presentation / Deck Integration

- [ ] **Slide number** — replace `XX / YY` with the correct position in the deck (currently 15 slides, so this would be slide 16 of 16, or insert it before the closing/CTA slide).
- [ ] **Nav dot** — add one `<div class="nav-dot" onclick="goToSlide(N)"></div>` entry to the `.footer-nav` block.
- [ ] **Overflow check** — the 4-column grid with the bullet lists may be tight on 1080p screens. Preview at 100% zoom and adjust `padding` or `font-size` values as needed before presenting.
- [ ] **Mobile/narrow breakpoint** — the existing `@media (max-width: 1200px)` rule collapses `.grid-4` to a single column, but this slide uses inline `grid-template-columns`. Either extend the media query in the `<style>` block or accept that mobile layout will stack vertically.

---

*Draft prepared: 2026-03-29. All dates and milestones are provisional until confirmed by Max Simmonds and Tiziano Fiore.*
