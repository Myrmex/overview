---
layout: page
title: Chiron
subtitle: "Metrical Analysis Tool"
---

Chiron is a framework for the analysis of metrics and observation of linguistic and metrical phenomena, with an extensive linguistics basis. You can read more about it in these papers:

- 2021 D. Fusi, _Introducing Chiron, a Full-Stack Framework for Metrical Analysis: Part 1 - Data Collection_, «RCCM» 63, 1 (2021) 53-85.
- 2021 D. Fusi, _Introducing Chiron, a Full-Stack Framework for Metrical Analysis: Part 2 - Data Interpretation_, «RCCM» 63, 2 (2021) 407-431.

As a framework, it's not limited to a single language or metre; currently, I've implemented support for ancient Greek, Latin, Italian, and Modern Greek.

Also, you can find some real-world analysis samples and an overview of further Chiron extensions to prose rhythm in D. Fusi, _I due volti di Giano - Saggio di filologia classica e tardoantica allo specchio delle Digital Humanities_, Parma 2022 (ISBN 978-88-32158-49-6).

## Overview Video - Prosodies

This video presents a very short overview of the default Chiron web-based UI, even though being a full-stack, layered and modular system any other UI (or even none) can be used for it.

<iframe width="560" height="315" src="https://www.youtube.com/embed/biGX7zrm3po" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

The video starts with a demo page showing you only the phonological analysis of a Latin line. Please note that phonological analysis is just the first step; full appositives detection and metrical scan follow.

As you can see, once you enter some text, the system provides you with a full prosodical analysis, up to the subphonematic level. Of course, everything here is an approximated model, but it provides the basis for metrical scansion and phenomena observations.

The prosodical model is based on segments, roughly corresponding to phonemes, and visually represented in a number of rows just below the analyzed text:

- the blue row represents phonological values.
- the mini-charts row represents the opening level of each segment, which is at the core of the generic syllabification algorithm.
- the row below the charts summarizes syllabic weights.

After metrical scansion, further rows are shown, but that's not the case of the demo page.

This view is fully interactive: you can explore each segment, getting a human-friendly, synthetic description at the top, and a list of all the segment's traits. Though inspired by the linguistic notion of trait, here traits model any metadata attached to a segment. Consequently, there are several categories for them.

Also, some traits refer to a single segment, while others have a wider scope, ranging from syllables to tokens and even to the whole line. All these details are shown in the table.

You can also apply a quick visual hint by typing a trait's name and eventually value, and see which segments feature it. In this video I type the trait `syl-w` = syllabic weight, and look at some of its different values.

Syllabification is based on the universal syllabic model ultimately dating to Saussure, with adjustments provided by each language-specific module. So, you can see the opening curve hinting at syllabic boundaries at a glance in the chart displayed under `Syllabification`.

Back to the text, I then switch to Greek and type a Greek line. As you can see, the generic framework stays unchanged, but we are now effectively exploring the results of analysis of a different language.

This kind of detailed and interactive view is available for every single analyzed line in a full corpus, and provides the closest reading experience. Next to it, the UI also allows you to jump to the other extreme point, by looking at aggregated data about any observed phenomenon, in the context of a distant reading experience.

Prosodical analysis and metrical scansion in fact are only a part of the system; once they are completed, all the details about their results get stored in a database, which in turn can be subject to inspection for specific phenomena of any kind, either known or hypothetic. Observations produce other data in the database, for each observed phenomenon. Every observation is produced by an independent, pluggable software module, named _observer_, which rely on a multi-dimensional model also ready for third-party, ML-based analyses.

For instance, in this video I jump directly to an interactive graph of data nodes, built from the hierarchy underlying the model of hiatus as observed by its corresponding software module. This model distributes the relevant hiatus features in a hierarchy, visualized as a tree in the left part of the page. The first two branches of this tree are `n` for non prosodical hiatus, and `p` for prosodical hiatus. When the branches are expanded, you get the detailed frequencies for each feature.

You can drill down the tree and see the chart get in synch with your selection, thus exploring all the features in the model. Here for instance you can see that both these branches have children representing false (`f`) and true (`t`) wordends.

In turn, each of these wordends can be the place of a punctuation sign, which often is a good clue for syntactic pauses. If we drill down any of the wordend nodes, we can see that it has a number of children representing different punctuation strengths (from `p0` to `p4`).

Again, each of these punctuation nodes has two children representing the presence (`e1`) or absence (`e0`) of elision. So, here you can explore the features of the hiatus model, at any level of detail.

This is an interactive, high-level view of this phenomenon; yet, at any time we can jump down to the lower level, back to close reading. Here for instance I click on a single node, like absence of elision (`e0`) in the cases of hiatus with a light punctuation (`p1`), true wordend (`t`), and affecting syllabic weight (`p`), and I'm immediately taken back to the detailed list of all the verses presenting right this type of hiatus (highlighted in yellow).

For instance, if you look at the first verse, you can see right this type of phenomenon in `μετανίσσεται, ἀλλὰ`, where the final diphthong `-ται` is shortened by hiatus, and stands before a comma, in a true wordend (orthotonic + prepositive), without elision. We can even drill down to examine the details of each single segment and trait, just like before, in a fully interactive view. On passage, you should now notice that there are more rows for each line, because these are metrically scanned verses.

This concludes our 2-minutes overview video, showing you a constant movement between close and distant reading approaches, for a sample chosen phenomenon like hiatus. Now, the real corpus analyzed here counts about 90,000 verses for 1300 years from Homer to Nonnos, averaging 20 traits per segment, and 40 segments per verse, for a total of 70 millions of raw data. Also, once we add phenomena observations, we get other 7 millions of high-level data about every linguistic and metrical phenomenon we desire. All of this is available in a standard RDBMS, totally independent from the Chiron software, so that both data and software (via API) can be consumed by any third party.
