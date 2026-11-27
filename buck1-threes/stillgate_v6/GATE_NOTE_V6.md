# Buckminster Ep. 1 "Privacy" — Three.js pilot V6 STILL GATE

**Status: STILL GATE ONLY. No capture run, no master assembled. Audio untouched.**
V6 = V5's render stack (threejs-skills lighting/materials/post, ACES, GTAO,
env intensities) unchanged, plus a staging/asset pass aimed at exactly the
three defects arab named from the V5 stills. All changes are in
`web/main.js` (mode `v6`); V5's code path is intact.

## The three fixes

### 1. Sinking in the sofa — seat lift + leg re-plant
Measured cause: the baked hips bone sat at y 0.44–0.45 with the seat
cushion top at y 0.48 — the pelvis was *below* the seat line, knees at
0.41–0.42, also below it.
Fix: each root is lifted (Arlo +0.070 m, Riko +0.080 m → both hips bones
at **y 0.52, 4 cm above the cushion top**), then each thigh is rotated
about its own hip hinge until the shoe's lowest vertex returns to its
baked floor height (thigh deltas: Arlo 9.9°, Riko 12.2°; 15.9° mid-dive).
The DiveCushion rises with Riko so his baked hand contact is preserved.

| check (gate frames) | baked (V5 staging) | V6 |
|---|---|---|
| hips bone Y vs cushion top 0.48 | 0.45 / 0.44 (below) | **0.52 / 0.52 (above)** |
| shoe lowest vertex Y | 0.002 / 0.014 | 0.003 / 0.014 (planted) |
| thigh verts below cushion-top plane, rear seat zone | 36 / 42 | **15 / 18** |
| baked deepest seat vertex (ThighL #27) | y 0.391 | y 0.463 (lifted +0.072) |

The deepest vertex in the zone in V6 (y 0.363) is an underside vertex
the body turn swung into the measurement box; it is inside the sofa
base volume but occluded by the cushion in every gate framing — the
same class of hidden underside contact the baked rig already carried,
with ~55% fewer submerged verts than baked. No foot/floor penetration:
a post-pass in the re-plant guarantees no shoe vertex below y 0.001
(it also cured a 6 mm dip on Riko's dive foot).

### 2. Bodies angled toward each other — staging yaw replaces head strain
Measured cause: the baked staging faces the bodies ~99° apart; V4/V5
bought the gaze with neck+head and the skinning stretched the heads.
Fix: each character is rigidly yawed on the sofa — **Arlo +54°, Riko
−54°** (swept 40/46/50/54; 54 gave the smallest residuals with feet and
cushions still clean) about a pivot 0.12 m in front of the spine, like
turning in on a corner sofa. The spine's local rotation is never
written (**spineLocalDelta = 0.00° at every gate frame**); the baked
pose data is otherwise intact. Neck+head caps tightened to
**yaw ≤ 20°, pitch ≤ 20°**, with a gain-refinement pass so the head
uses its full legal budget. Two choreographed easings: Riko's staging
relaxes to 30% across the dive so the duck lands on the cushion's baked
line; Arlo's relaxes to 35% across the phone-place window so the
hand-off stays on its baked spot.

Gaze residuals (face-forward vs target, degrees; heads stay ≤ caps):

| beat | Arlo err | Riko err | note |
|---|---|---|---|
| L1 | 8.9 | 37.0 | Riko's target is his phone, held on his outer side — see flags |
| L2 | 17.6 | 16.6 | gate still f0152 |
| L3 | 17.0 | 13.7 | |
| L4 whisper | 13.9 | 14.6 | gate still f0323 |
| L5 / dive | 5.7 | 60.7 | Riko mid-dive behind the cushion — see flags |
| L6 phone insert | 21.5 | 3.5 | both look at the table phone |
| L7/L8 | 16.8 | 15.0 | |
| L9 plate | 5.2 | 0.9 | |
| L10 | 12.2 | 19.7 | gate still f0851 |
| tail wide | 2.8 | 15.6 | gate still f0883 |

Mutual dialogue beats land at **13–20°** (V2 measured ~95–100°, V4's
caps left ~35–50°). Applied head turns at the gate frames: yaw
13.8–20.0°, pitch 14.6–20.0° — at the caps but never past them.
**Head/neck stretch check (the specific V5 complaint):** in the L2 and
L10 stills at full size the heads keep their baked proportions — no
elongation; the turn is visibly carried by the shoulders/torso
orientation, not the neck. Torso world tilt stays 10–13°, as baked.

### 3. Hair gaps — scalp patch under the spikes
Measured cause: a HairCap mesh exists but its front edge stops ~11 cm
(in z) short of the fringe line, so skin striped through between the
front spikes.
Fix: a patch grown from the Head mesh's own scalp vertices (front
hairline band + one relaxed ring), skinned to the same skeleton with
the same bind, in the HairCap material, offset 4.5 mm along
position-averaged normals. (First attempt offset along per-face
normals — the head is a triangle soup, the plates separated and skin
lined every patch edge; the averaged-normal offset cured it, verified
at 2× zoom in the L4 close-up.) The patch is under the spike layer at
every angle: in the L4 close-up and both wides there is **no
see-through gap between spikes anywhere** — the between-spike region
is solid black, continuous with the cap.

## No-regression checks
- **Grade:** scene stills medS 0.467–0.478, medL 0.537–0.571, warmth
  +72…+73 — V5's exact band (0.467–0.478 / 0.53–0.57 / +70…+73). The
  L4 close-up shifts compositionally (medL 0.68 vs V5's 0.81, warmth
  +63 vs +71): the staged turn puts more dark hair in the close-up
  frame. Lighting stack untouched.
- **Interpenetration:** full-episode broadphase (all 932 frames, v6):
  zero AABB flags; minimum head-to-head distance 1.049 m. Exact
  ray-parity (3-direction majority, brute force) at the 5 gate frames:
  **0 points of either character inside the other, both directions**
  (min surface-to-surface distance 0.373 m at k=151); 0 points inside
  the table top. Riko↔DiveCushion contact (45–79 sampled points) also
  exists in the baked build (33–48) — it is the prop he leans on and
  grabs; staging raises the count, but no clipping is visible in any
  gate still. (An earlier grid-hash parity pass reported false
  character-interpenetration positives; it was discarded and replaced
  with the brute-force test — numbers above are the corrected ones.)
- **L4 frontal reframe:** kept (camera condition now v4||v6); Riko's
  face is front-on and fully visible for the whisper, as in V5.
- **Audio:** untouched — no capture was run; the approved mix
  (stream MD5 d48cfc936920bda972cf61530674d14f) is not involved at a
  still gate.

## My own flags (read the stills like arab will)
1. **L2 framing is tight on Riko.** The s02 camera was composed for
   the unstaged seats; with his body turned in, Riko sits at the right
   frame edge in f0152 — face fully visible, but tighter than V5. If
   this bothers you, the fix is a small s02 camera nudge at capture
   time, not a staging change.
2. **L1 Riko/phone residual (37°).** His body now faces Arlo while the
   phone sits in his outer hand, so within the 20° no-stretch cap his
   face lands between phone and Arlo. For the line ("Don't say my
   salary out loud!") looking toward Arlo arguably plays better — but
   the beat map still says phone. Your call; the alternative is the
   head strain you banned.
3. **Dive gaze number (60.7°) is a non-issue visually** — mid-duck his
   face is behind/above the cushion looking forward-down, exactly the
   baked dive read; the "target" during those frames is nominal.
4. **They sit slightly wider apart** (heads ~1.25 m vs 0.91 m baked):
   the corner-sofa turn swings the hips outward a touch. It reads as
   two people turned in to talk, not as distance — but it is a
   different seat picture from the Blender banked frames.
5. **Hair reads as a solid black mass with spikes** — no gaps, but at
   close range the patch+cap surface is smoother/flatter than the
   spiky silhouette in the Blender frames. That is the price of
   closing the gaps with this rig's geometry.
6. Carried nits from V4/V5, unchanged: L4's pale frame-left eye (baked
   geometry), Riko's hand over the dive cushion's front face (baked),
   hot-orange forearm in wides.

## Files
- Stills: `f0152/f0323/f0385/f0851/f0883.png` (720×1280)
- Three-way side-by-sides (Blender | V5 | V6): `sbs3_*.png`,
  contact sheet `contact_sheet_v6.png`
- Measurements: `pose_report.json`, `extras.json`, `gaze_lines.json`,
  `overlap.json`
- Code: `web/main.js` mode `v6` (staging constants are named
  V6_LIFT / V6_STAGE_YAW at the V6 section)

**STOP — awaiting arab's verdict. No capture without it.**
