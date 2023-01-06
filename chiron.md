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
