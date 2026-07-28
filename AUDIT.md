# SURGeLLM — consistency audit

Compiled while building the project page. Every item was checked against the
tables in the manuscript itself, so each one is either an internal contradiction
or a claim the tables do not support. Nothing here is a guess about your
experiments — where I could not verify something from the PDF, I say so.

Grouped by severity. Numbers in parentheses are what the tables actually give.

---

## A. Substantive — a reviewer will raise these

**A1. `ρ_k` is not scale-invariant, so the pre/post-IWN comparison is confounded.**
Definition 1 makes `ρ_k` an expected *absolute inner product*, `E|⟨s′, ∇_ĥ log p⟩|`.
Its magnitude scales with `‖s′‖`. IWN changes the standardization of `s`, which
changes `‖s′‖`. So the headline `ρ₄ : 0.61 → 2.13` (Table 15) partly measures a
change in feature scale rather than a change in alignment. This matters because
the theory-to-empirics link in §8 and Appendix A rests entirely on that number.
*Fix:* report a normalized alignment alongside it — e.g. cosine
`E[|⟨s′, ∇⟩| / (‖s′‖‖∇‖)]` — and show the ordering survives. If it does, the
argument gets much stronger; if it doesn't, better to know now.

**A2. IWN is not only a recentering — it also rescales, and the "free on balanced
corpora" claim overstates the equivalence.** §3.5 says that when `π_c = 1/n_c`,
`σ^bal_k = σ_k` "up to the difference between weighted and unweighted variance
estimators." That understates it. By the law of total variance,

```
σ_k²  =  mean_c(σ_{c,k}²)  +  var_c(s̄_{c,k})
```

so whenever the class means differ at all, the pooled SD strictly exceeds the
mean of the per-class SDs — even at perfect balance. `σ^bal_k < σ_k` is the
normal case, not an estimator technicality. IWN therefore inflates feature
magnitudes by roughly `σ_k / σ^bal_k`, which interacts with `W_s` and with A1
above. This is not a problem for the method; it is a problem for the claim that
IWN "costs nothing in the balanced regime." Either soften the claim or quantify
the ratio on D₁–D₃ (all near-balanced) and show it is ≈ 1.

**A3. D₃ has a *larger* raw skew than D₄ and shows none of the pathology.**
§4 gives LLM-7 an ≈ 11.8:1 human skew; D₄ (HumLLM) is 9.3:1. The mechanism story
says raw skew contaminates the training statistics and depresses alignment — yet
`ρ₃ = 1.83` pre-IWN against `ρ₄ = 0.61`, and IWN buys only +0.018 on D₃ against
+0.181 on D₄. A reviewer will read this as evidence that skew is *not* the
operative variable. You almost certainly have the answer (different capping,
different feature moments, longer texts), but the paper never addresses it. One
paragraph in §6.3 would close the hole.

**A4. Cross-backbone significance test.** Table 4 reports
"SURGeLLM-FULL-ALBERT vs. Base-RoBERTa, D₂, p = 0.011". That comparison changes
the backbone *and* the components at once, so it cannot isolate either. No
ALBERT baseline appears anywhere in Tables 3, 7, or 12 — which is also why the
ALBERT row in Table 7 carries "—" for ∆. Either train Base-ALBERT or drop the
comparison.

**A5. Overlapping vocabulary groups undercut the leave-one-out reading.**
`sst_neg` and `human_emo` both contain *terrible* and *awful* (Appendix D).
Leave-one-out is interpreted in §7.2.4 as though the groups are separable, but
removing one leaves its shared members firing via the other. Report group
overlap (a 10×10 Jaccard matrix would take three lines) or note the caveat.

**A6. `prompt_cot` contains bare high-frequency tokens.** *first*, *then*, and
*next* are exact matches with no prefix marker, so this group will fire on
ordinary prose across every task, not just chain-of-thought text. It is
plausibly why `prompt_cot` shows the largest D₃ leave-one-out drop; it is also
why a skeptical reader will ask whether the group is measuring reasoning
structure or just discourse-marker density.

---

## B. Internal numerical contradictions

| # | Location | Says | Tables give |
|---|---|---|---|
| B1 | Abstract | "+0.036 over the strongest non-IWN baseline" | +0.034 over the strongest non-IWN **variant** (G-RoBERTa, 0.906); +0.036 over **Baseline-RoBERTa** (0.904). §6.1 states both correctly — the abstract conflates them. |
| B2 | RQ1 (App. A.4) | "SURGeLLM-IWN-RoBERTa exceeds SURGeLLM-G-RoBERTa by +.005 avg. F1" | 0.940 − 0.906 = **+0.034**. (The companion figure, "+.132 on D₄", is correct.) |
| B3 | RQ3 (App. A.4) | extended training without IWN costs "−0.054 vs. Baseline" on D₄ | 0.711 − 0.762 = **−0.051**. Figure 3 says −0.051; App. L.1 says "worst case −0.052". Three different values for one quantity. |
| B4 | §7.4 prose | "Three models are Pareto-efficient" (names DistilBERT, BERT, IWN-RoBERTa) | Table 12 stars **five**, and five is correct on the stated data: Baseline-RoBERTa (233 s, 0.904) and G-RoBERTa (327 s, 0.906) are both non-dominated. The prose is wrong, not the table. |
| B5 | Abstract, §1, §7.3, RQ4, App. E | auto-extraction "recovers 99.5%" | 0.903 / 0.906 = **99.7%**. The companion phrasing "a 0.3% relative gap" is right; 99.5% is not. |
| B6 | Table 5 caption | "∆ is IWN vs. SURGeLLM-FULL on the same backbone" | There is no SURGeLLM-FULL-**BERT** anywhere in the paper. The ∆(BERT) row (+.118 on D₄) is 0.866 − 0.748, i.e. against **SURGeLLM-S-BERT**. Caption needs correcting. |
| B7 | §7.3 vs App. J | "auto-extracted vocabularies yield F1 within 0.02 English-curated baselines" | App. J makes the precise claim: gains of +0.009 in both languages, within **0.001** of the +0.008 English gain on D₁. The §7.3 phrasing is both looser and grammatically broken. |
| B8 | Table 12 caption | `Eff = F1 × 10³ / log₁₀ P` | Only reproduces the tabulated values if **P is in millions** (DistilBERT: 886 / log₁₀66 = 487.0 ✓; with P = 66×10⁶ you get 113.3). State the unit in the caption. |
| B9 | Table 3 vs Table 12 | `⋆` = early stopping in Table 3, `⋆` = Pareto-efficient in Table 12 | Same glyph, two meanings, two pages apart. Use `†` for one of them. |

---

## C. Citations and sources

**C1. The Chang et al. (2018) citation is the wrong paper.** §6.4 cites it for
"for a fixed parameter budget, classification-specific encoders match or beat
seq2seq models on classification tasks." The reference list resolves it to
*"Neuropathic-Like Ocular Pain and Nonocular Comorbidities Correlate With Dry
Eye Symptoms," Eye & Contact Lens*. This is the single most damaging item in the
audit — it is the kind of thing that gets screenshotted. Find the intended
reference or drop the sentence.

**C2. HotPotQA split size is mislabeled.** §4.2 says "We use the validation
split (90,564 questions—context pairs)." HotpotQA's dev set is 7,405 examples;
**90,564 is the training split**. Either you sampled from train and called it
validation, or the count is wrong. Both readings are fixable, but they imply
different things about test contamination, so this needs to be exactly right.

**C3. Two dataset references are still marked "Author-check required."**
`LLM-7 Dataset Contributors (2024)` and `Grinberg (2024)` both carry the note
"please update with canonical URL/DOI before camera-ready." These are the
sources for D₃ and D₄ — the task that carries your headline result. Resolve
before anything goes public.

**C4. `GermanSentiment` has no reference.** App. J names the corpus but the
bibliography has no entry for it (AlloCiné → Blard 2020 is present).

**C5. "Code: released at https://surgellm-iwn.github.io (Project Webpage)."**
A project page is not a code release. The reproducibility checklist should point
to a repository with a commit hash or a DOI'd archive (Zenodo).

---

## D. Wording

- §3.5: "yielding +0.130 an absolute F1" → "an absolute +0.130 F1"
- §6.4: "T5-base reaches 0.897 an avg. F1" → same construction
- §3.3: "Let V = {v₁,…,v₁₀} the ten indicator groups of the surgical vocabulary
  be" → word order
- §7.3: "yield F1 within 0.02 English-curated baselines" → missing preposition
- §2: "Class imbalance in loss-side(Lin et al., 2017)" → missing space

Some of these may be PDF-extraction artifacts rather than source errors — worth
a pass over the `.tex` to confirm.

---

## E. Page placeholders — replace before publishing

The HTML marks each of these with a visible `add file` / `add URL` chip so none
can ship silently. There is also a comment block at the bottom of `index.html`.

1. `href="surgellm.pdf"` — drop the PDF next to `index.html`
2. Code button — repository URL
3. Slides button — the Beamer deck
4. Poster button — the A0 / NAACL-style poster
5. Author name links — personal or Scholar pages
6. `og:image` and a favicon
7. Dataset links for D₃ and D₄, once C3 is resolved

---

## F. What I deliberately did not change

The page reproduces every table value exactly as printed in the manuscript,
including the ones flagged above, **except** the abstract's "+0.036 over the
strongest non-IWN baseline" (B1), which I rendered as "+0.034 over the strongest
non-IWN variant; +0.036 over the RoBERTa baseline" because the page states both
numbers next to the table that contains them, and printing the conflated version
next to its own contradiction would have been worse. The B4 Pareto count is
rendered from the table's `⋆` marks (five), not the prose (three). Everything
else — including B5's "99.5%", which appears in the abstract — is left as
written so the page and the PDF do not diverge. Fix them in the source and the
page can follow.

The `π` slider in the IWN section is a schematic of Eq. 9 with class profiles
placed at ∓1 and unit within-class scale. It is labeled on the page as
illustrative and not measured, and no number from it appears anywhere else.
