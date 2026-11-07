# CRISPR Guide Designer

A single-file, browser-based tool for designing CRISPR guide RNAs. Paste a target sequence, choose a nuclease, and get ranked guides with cut sites, in-sequence off-target checks, cloning oligos and paired-guide deletion designs.

No install, no server, no dependencies beyond two Google Fonts. Your sequence never leaves your browser.

## Quick start

1. Open `guide-designer.html` in any modern browser (or host it anywhere that serves static files).
2. Paste a DNA sequence or FASTA record into the text box, or select **Load example**.
3. Choose a nuclease and adjust the options below.
4. Select **Find guides**.

## Features

- **Three nucleases:** SpCas9 (NGG, 20 nt), SaCas9 (NNGRRT, 21 nt) and Cas12a (TTTV, 23 nt, PAM upstream of the guide).
- **Both strands scanned:** positions are always reported on the strand you pasted, numbered from 1.
- **Cut sites:** SpCas9 and SaCas9 cut 3 bp from the PAM. For Cas12a the reported cut is on the non-target strand, which is staggered.
- **Region filter:** optional start and end limit results to guides overlapping a region of interest, such as an exon.
- **Near-match check:** each guide is compared with every other site in your sequence. Sites within 3 mismatches are counted and penalized, and exact duplicates are penalized heavily.
- **Cloning oligos:** top and bottom oligos for SpCas9 and SaCas9 with CACC and AAAC overhangs, for BbsI (for example pX330) or BsmBI (for example lentiGuide) U6 vectors. A 5′ G is added when the guide doesn't start with one.
- **Paired guides:** the best pairs of guides scoring 70 or higher that cut 20 to 1000 bp apart, for deleting the segment between them.
- **Interactive map and table:** bars on the map are color-coded by score and linked to table rows. Every column that has a sort key can be sorted.
- **Export:** copy a single guide, an oligo, or the whole results table as CSV.

## Options

| Option | Default | Effect |
| --- | --- | --- |
| Nuclease | SpCas9 | Sets the PAM, guide length and PAM position. |
| Min GC % / Max GC % | 40 / 70 | Guides outside this range lose 12 points. Outside 30 to 80 they lose 30. |
| Region start / end | whole sequence | Keeps only guides overlapping this range. |
| Avoid TTTT | on | Penalizes TTTT runs, which terminate U6 transcription, by 35 points. When off, the penalty is 5. |

## How scoring works

Every guide starts at 100 and the score is never below 0.

| Rule | Adjustment |
| --- | --- |
| GC content outside your range | −12 |
| GC content below 30% or above 80% | −30 |
| TTTT run | −35 (U6 on) or −5 (U6 off) |
| Run of 5 or more A, C or G | −10 |
| GGGG | −8 |
| Possible hairpin (a 4-mer whose reverse complement appears later in the guide) | −6 |
| Base next to the PAM is G (3′-PAM nucleases only) | +4 |
| Base next to the PAM is C (3′-PAM nucleases only) | −5 |
| Exact duplicate elsewhere in the sequence | −40 |
| Near matches (1 to 3 mismatches) | −8 each, capped at −25 |

A missing 5′ G is flagged in the notes but not penalized, since the oligo design adds one.

Colors on the map: green is 80 or higher, amber is 60 to 79, red is below 60.

## Limitations

- **Not a genome-wide off-target search.** The near-match check only looks inside the sequence you paste. Before ordering oligos, check your shortlist against your organism's genome with a tool such as [CRISPOR](https://crispor.gi.ucsc.edu) or [CHOPCHOP](https://chopchop.cbu.uib.no).
- **Heuristic scoring.** The score is rule-based, not a trained on-target efficiency model such as Rule Set 2 or DeepSpCas9. Treat it as a first-pass ranking.
- **Near-match check cap.** It is skipped when the input has more than 2,500 candidate sites, and the table then shows "n/a".
- **Oligo overhangs.** They are standard for BbsI and BsmBI U6 vectors, but confirm them against your vector's documentation. Cas12a cloning varies by vector, so no oligos are generated for it.
- **Input handling.** FASTA header lines are ignored, U is read as T, and any character other than A, C, G or T is dropped, so sequences containing N are joined across the gap. Strip or split such sequences before pasting.
- **Paired guides.** Pairs are chosen on cut-site distance and score only. Orientation is not considered, so they aren't suited to nickase (PAM-out) designs.

## Output columns

`id`, `strand`, `start`, `end`, `cut_after`, `guide`, `pam`, `gc_percent`, `near_matches`, `score`, `notes`. The CSV button copies these to your clipboard as comma-separated text.

## Customizing

Everything lives in one HTML file with no build step.

- **Add a nuclease:** add an entry to the `NUC` object with its PAM pattern (IUPAC letters supported by the `IU` table), guide length and PAM side (`3` or `5`). Add a matching `<option>` to the nuclease select. Check the cut-site arithmetic in `scan()` for the new enzyme.
- **Change scoring:** edit the penalty lines inside `scan()`.
- **Change colors:** edit the CSS variables at the top of the stylesheet. A dark theme is defined for `prefers-color-scheme: dark`.

## License

Add your preferred license here.
