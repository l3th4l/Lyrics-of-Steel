
# Lyric of Steel – Musical Combat System Design

## 1. Overview
Players fight by performing short **note sequences** on a magical lyre.  
Every unique sequence produces a **deterministic, unique spell effect**, and sequences that sound musically pleasant are rewarded with **greater power**.

## 2. Core Concepts
### 2.1 Spell Sequence
* A “spell” is a series of **3-8 notes** (letters A–G plus optional #/b).
* Example input: `A A B D#`.

### 2.2 Deterministic Mapping
Each valid sequence always yields the **same effect**, independent of who plays it.  
No collisions: two different sequences never produce the same *complete* effect card.

## 3. Musical Layers → Gameplay Effects

| Musical Layer | Game Meaning |
|---------------|-------------|
| **Element** | Each pitch maps to an elemental school. |
| **Interval Contour** | Defines projectile behaviour & targeting. |
| **Chord Quality** | Governs role (damage, heal, control, buff). |
| **Cadence & Rhythm Flags** | Adds rider effects: charge, echo, instability. |
| **Harmony Score** | Adjusts overall potency based on musical consonance. |

### 3.1 Element Mapping
Every note’s pitch-class determines an **elemental colour**:

| Pitch | Element |
|-------|--------|
| A | Fire |
| A#/Bb | Blood / Life |
| B | Lightning |
| C | Ice |
| C#/Db | Metal / Armor |
| D | Wind |
| D#/Eb | Shadow |
| E | Water |
| F | Earth |
| F#/Gb | Arcane |
| G | Nature |
| G#/Ab | Radiance |

*Primary Element*: the note appearing most often.  
*Secondary Elements*: all others modify or add status riders.

### 3.2 Interval Contour
Intervals (in semitones, mod 12) between adjacent notes:

* **±1–2** – Precise projectile / single target, higher crit chance.  
* **Thirds (±3–4)** – Cleave arcs or cone AoE.  
* **Fourth/Fifth (±5/7)** – Multi-ricochet or chaining bolts.  
* **Tritone (±6)** – Debuff or curse rider.  
* **Octave (0 or ±12)** – Magnitude amplifier.

### 3.3 Chord Quality (Last 3–4 Notes)
* **Major triad** – Buffs & healing.  
* **Minor** – Pure damage or life-drain.  
* **Diminished/Half-dim** – Stun, immobilize, fear.  
* **Augmented** – Armor-pierce, barrier break.  
* **Suspended (sus2/4)** – Interrupts, silence.  
* **7 / m7 / maj7** – AoE duration or radius bonus.

### 3.4 Cadence & Rhythm Flags
Pattern recognition over the raw text:

| Pattern | Effect |
|---------|-------|
| Repeated opening note (`AA…`) | Charged shot: +power, slower cast. |
| Ends on sharp/flat | Unstable: chance of extra bleed/chain. |
| Palindrome (`ABBA`) | Echo/return projectile. |
| Overall ascending | Long-range travel. |
| Overall descending | Close burst. |

## 4. Harmony Score (Good-Sound Bonus)
To reward pleasant sequences:

**Metric** = weighted average of  
*Interval consonance*, *Chord consonance*, *Key cohesion*.

`harmony_score ∈ [0,1]`

**Power Scaling**

| Score Range | Bonus |
|-------------|------|
| 0.0–0.3 | no bonus |
| 0.3–0.6 | +10 % |
| 0.6–0.8 | +20 % |
| 0.8–1.0 | +30 % & cosmetic “resonance” VFX |

Low-harmony sequences remain valid but may favour chaotic riders (random bounces, confusion).

## 5. Effect Generation Algorithm (Pseudo-Code)

```
def generate_spell(notes):
    pcs = [to_pitch_class(n) for n in notes]            # 0–11
    elements = [ELEMENT_MAP[p] for p in pcs]
    primary = most_common(elements)

    intervals = [(pcs[i+1]-pcs[i]) % 12 for i in range(len(pcs)-1)]
    delivery = contour_to_delivery(intervals)

    chord_q = infer_chord_quality(pcs[-3:])             # major/minor/etc
    cadence = detect_cadence(notes, intervals)

    harmony = compute_harmony_score(pcs)

    magnitude = base_power(len(notes))
    magnitude *= 1 + 0.3 * harmony                      # reward good sound

    riders = riders_from(chord_q, cadence)

    return SpellEffect(
        element=primary,
        secondary=set(elements)-{primary},
        delivery=delivery,
        role=role_from_chord(chord_q),
        magnitude=magnitude,
        riders=riders,
        unique_id=hash(notes + intervals + chord_q + cadence)
    )
```

## 6. Player Experience
* **Pre-Battle Setup** – Players pre-select favourite sequences into quick-slots.  
* **During Combat** – Enter or tap a sequence → cast spell.  
* **Progression** – Skill tree unlocks new rider effects and higher harmony multipliers.

## 7. Balancing Levers
* **Base Power** scales with length of sequence.  
* **Cost** rises with accidentals and dissonant intervals.  
* **Cooldown** set by control strength.

## 8. Example Spell Cards

| Sequence | Effect Summary |
|----------|----------------|
| **A A B D#** | Ember Lance – Fire bolt with shield-pierce, small cleave, 20 % chain. |
| **A B A D#** | Cinder Rebound – Bouncing fire-shadow bolt with stun. |
| **D F# A** | Gale Hymn – Wind/Arcane haste pulse with minor healing. |
| **C C# D** | Discordant Clutch – Chaotic shadow burst that confuses enemies. |

## 9. Optional Extensions
* **Dynamic Key** per dungeon for extra resonance rewards.
* **Performance Gestures** for live play bonuses.
* **Mode Cards** to bias effect weights.
