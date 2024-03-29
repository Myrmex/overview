---
layout: page
title: Proteus
subtitle: "Text Conversion Tool"
---

👀 [Proteus Demo](http://proteus.fusi-soft.com/)

## Proteus

Nowadays, still many useful resources are text-based and legacy-encoded, in both academic and business worlds. Often, not only their digital format, but also their character encoding are either proprietary or legacy, so that such digital resources are doomed to early obsolescence.

Especially in humanities, a lot of invaluable resources are still out of the reach of a modern digital world, sealed in the coffer of their venerable paper-based edition. Yet, even more business-oriented resources may require considerable efforts to be recovered from their legacy digital formats.

I often happened to design software solutions for a number of such scenarios, often involving dictionaries and textual corpora. The more I worked on these solutions, the more work patterns emerged, leading to a progressive generalization of my original software design and tools. The current outcome of this process is a set of text-conversion software modules I collectively named Proteus.

A number of relevant real-world usages came out from its application, and I'm sketching some of them here not only as a quick overview for my customers, but also as a proposal for more academic oriented use cases.

Apart from big legacy resources like dictionaries, a more specialized field of application was converting academic texts: often books or papers, but even legacy digital corpora, or paper critical editions. For instance, this has been the case for [MapAeg](https://mapaeg.websoupcloud.it), recovering data from Word documents; or from a set of four distinct [documental archives from Piero Calamandrei](https://archiviocalamandrei.it), refactored and merged into one using this system.

Other scenarios might even involve converting the text encoding itself.

For instance, many readers versed in classics might probably recall the outgrowth of polytonic Greek fonts in the pre-Unicode era. In this case, a presentational device like a font was misused as a true, proprietary character encoding, allowing word-processor users to type on their Latin-based keyboards and magically get what seemed a Greek text. Of course, simply switching to another font, or to a machine without that specific font resource, caused the loss of this appearance, resulting in a sort of “garbage text”. Further, for any machine-based treatment this was just Latin text, whatever its appearance.

Once a lot of digital texts were produced with these fonts, each with its own, undocumented and incompatible encoding, they were effectively siloed in a sort of legacy format, hard to be recovered and integrated in a modern environment. You can refer to the Proteus demo for most of these Greek fonts.

A less episodic scenario can be represented by huge corpora or digital works encoded with standard, yet legacy formats like Beta code. This is both a character encoding and a digital rich-text format, born in the ASCII-era to represent mainly Greek and Latin texts in their critical editions. A lot of valuable resources, especially from the Perseus Digital Library, still use this legacy format, but could effectively be converted into Unicode by leveraging this system.

Following the same path, even more rich texts might be converted into modern and semantic-oriented encodings and formats, starting from just a word processor document, or even from paper via OCR: critical editions, corpora, encyclopedic works, dictionaries, etc.

A more specialized application of the same system also concerns documental archives, which are often affected by the early obsolescence of their digital formats or software tools, or may even be still on paper. A project based on Proteus (together with other software systems) for recovering or acquiring such data is under development, and was partially funded by institutions and companies.

Finally, a more unexpected application of the conversion system is found in content creation processes, ranging from simple typing aids (see the Theuth demo) to full-fledged, XML-oriented user friendly editing experiences. In fact, this is essentially a real-time, bidirectional, and modular converter between any XML dialect and a user-defined rich text, used to provide a visual, word-processor like editing experience for simple TEI-based corpora or specialized XML dialects (e.g. in lexicography).

▶️ Next:

- [Text Encoding](./proteus/proteus-encoding.md)
- [Format Encoding](./proteus/proteus-format.md)

📁 Downloads:

- [Theuth]({{ site.url }}/downloads/theuth-demo.zip)
