---
layout: page
title: Proteus - Encoding Conversion
subtitle: "Text Conversion Tool"
---

👀 [Proteus Demo](http://proteus.fusi-soft.com/)

## Text Encoding

A first problem in modernizing text-based resources arises when the source text has a proprietary or legacy encoding. In this case, each numeric code has an arbitrarily defined value, or may even have more than one single value according to its context. Further, often older encodings also define some specific codes sequences (escapes) as having a specific textual or meta-textual meaning. Legacy digital documents may have several different encodings, changing either from document to document or even inside the same document: for instance, a dictionary with proprietary encoding may have a specific encoding for pronunciations, which require IPA characters, and another for the rest of the text.

A generic, reusable encoding converter must then take into account all these issues. As this configures a complex scenario, we must also maximize the effort made in describing each encoding we want to enter the conversion process. This is extremely useful in scenarios where you are dealing with several different encodings and you want to convert them in different directions and combinations. This implies that any encoding might be used as either the input or the output of the conversion process: an encoding A might be converted into an encoding B, but we might also want to convert B to A. In both cases, the description of A and B are the same. Should we introduce a third encoding C, the possible directions might be 6: A to B, A to C, B to A, B to C, C to A, C to B; yet, the encoding descriptions remain 3: one for A, one for B, one for C. We might even want to make conversions like A to B + C, when B and C represent two complementary encodings. Thus, the combinations that each encoding may enter in a conversion process are many, and every single encoding should be considered independently from them.

In other terms, the description of an encoding should be independent from direction of its conversion, i.e. either it represents the source or the target of the process. This is possible because the conversion systems relies on encoding descriptions, rather than directly mapping the source to the target encoding: i.e. we fully describe A, and we fully describe B, one independently from the other; and then we can view the conversion process as the intersection between these two descriptions. The conversion is not a process based on predefined mappings, but a heuristic process, where the converter tries to find the best match between two intersecting encodings.

This makes it possible to introduce any new encoding in the conversion process, whatever its role (source or target) and combination (e.g. targets B+C from A), by simply adding its description. The converter will pick it up, and use it to make possible any conversion involving it. Thus, the data model for encodings is the same, which allows for a modular and efficient software design. As a bonus, we get also the possibility to check every intermediate step in the conversion, as we can compare two descriptions, based on the same format, rather than two different encodings.

### Text Encoding – Encoding Description

A description is just a way of telling which is the meaning of any code in a specific encoding: e.g. in ASCII we might say that code 65 is an uppercase A. This is a very natural way of describing an encoding, as it just resembles the way we would describe it to another person. If then another encoding has code 123 for this same letter, then we will say that in that encoding code 123 is an uppercase A. If we are going to convert ASCII into this encoding, the encoder will first decode ASCII 65 into “uppercase A”, and then look for an uppercase A in the other encoding. This happens to be code 123, so that ultimately the conversion will map ASCII 65 to this encoding 123. Of course, inverting the direction of the conversion is only a matter of exchanging the source with the target description: in this case, 123 will be decoded as “uppercase A”, which will be found to be the same description as ASCII 65: thus, we convert 65 to 123.

Of course this is a very simple example, but the converter works with descriptions, which do not necessarily correspond to single characters. For instance, an accented letter like á would be described with two different entries: one for the “lowercase a”, and another for its “acute accent”. It might well be the case that another encoding (say it's named B) has two different characters for these entries; in this case, the converter will be smart enough to find all the entries in B which match those required by the description of á, and output two target codes against the single source code.

On this way, many other conversion paths become possible: for instance, it may happen that only a part of the entries in encoding A are matched against encoding B; in this case, the output will be the best possible match for that encoding, discarding the entries which cannot be represented in it. This is the best possible result, which of course is better than outputting nothing at all. For instance, an encoding A might have diacritics for macron and acute accent, while an encoding B might have only the accent; in this case, a letter “a” with macron and acute accent would turn into a letter “a” with just the acute accent, as there is no way for B to represent the macron. Yet, that's the best possible result, and it is obtained without any predefined mapping, by just matching the descriptions.

Of course, if the converter has to match two descriptions it is necessary that these description use the same “words”: if a set of descriptions described ASCII 65 as “A”, and another set as “uppercase a”, no matching would be possible between different sets of descriptions. To this end, we must ensure that each description shares the same “vocabulary”. This is accomplished by drawing from the Unicode standard, which is the obvious reference where we can find almost all the characters we might need to describe.

Thus, a character “description” in Unicode terms is just a description where each entry is represented by one or more Unicode codes. In the converter, these descriptions are found in encoding description tables, where each line represents a described code, which is followed by a single code for the corresponding “segmental” entry, separated with a slash from zero or more codes for the corresponding “suprasegmental” entries, if any. Each code is represented with hexadecimal digits. For instance:

```txt
0041=0041
```

This means that ASCII code 65 (hexadecimal 41) is described as Unicode U+0041, i.e. it's an uppercase letter A. We might add a comment in the table to make things clearer (comments are lines beginning with a semicolon):

```txt
; 0041 LATIN CAPITAL LETTER A
0041=0041
```

Another example referred to a proprietary encoding for ancient (polytonic) Greek characters shows some diacritics:

```txt
; 03B9 GREEK SMALL LETTER IOTA
; +0313 COMBINING COMMA ABOVE
; +0342 COMBINING GREEK PERISPOMENI
008D=03B9/0313,0342
```

Here we are saying that code 141 (hexadecimal 8B) in this encoding is described as “Greek small letter iota with comma above (=smooth breathing) and perispomeni (=circumflex) accent”. It might also be the case that the segmental part is missing, and we have a non-spacing character (a diacritic which gets superposed to the previous character). In this case, the conventional syntax is setting the segmental value to 0, like in:

```txt
; #003B
; +0300 COMBINING GRAVE ACCENT
003B=0000/0300
```

where we have a grave accent which combines with the previous character(s).

It should be stressed that _this is not a code-to-code mapping_, but only a description of a source encoding code in terms of Unicode codes, which is more convenient (shorter and more compact) than using e.g. their standard Unicode name. This often appears as a comment in these tables just for the purpose of making the table easier to be read. Anyway, it is not a direct code-to-code mapping, as the converter finds the best match by looking for the maximum intersection between two sets of descriptions, and this might even lead to more than a single result.

For instance, let's take the case of a proprietary encoding for ancient Greek, having the following codes (hexadecimal):

- 28 = rough breathing (diacritic)
- 2F = acute accent (diacritic)
- 41 = lowercase Greek alpha (letter)
- 7C = subscript iota (diacritic)

Their descriptions would be:

```txt
0028=0000/0314
002F=0000/0301
0041=03B1
007C=0000/0345
```

Now, say the converter is going to convert the sequence of codes 0041,0028,002F,007C (or whatever order the diacritics after 03B1 might have): it will first decode this sequence as alpha (0041), rough breathing (0028), acute accent (002F), subscript iota (007C); then, it would have to re-encode these entries into the target encoding. Say this is Unicode: there, the available descriptions would involve 5 different characters:

```txt
; 03B1 GREEK SMALL LETTER ALPHA
03B1=03B1
; 037A GREEK YPOGEGRAMMENI
037A=037A
; 00B4 ACUTE ACCENT
1FFD=00B4
; 1FFE GREEK DASIA
1FFE=1FFE
; 03B1 GREEK SMALL LETTER ALPHA
; +0314 COMBINING REVERSED COMMA ABOVE
; +0301 COMBINING ACUTE ACCENT
; +0345 COMBINING GREEK YPOGEGRAMMENI
1F85=03B1/0314,0301,0345
```

As you can see, Unicode has two ways of representing this text: it can combine the letter with each diacritic separately, or just use a single precombined character (1FFE; such things in Unicode happen for compatibility reasons). Here we are describing all these characters: it is up to the converter to match them.

Given the described entries, there would be 2 different results: 03B1 037A 1FFD 1FFE, or just 1F85. In such cases, the converter picks the more compact result, and thus chooses the latter. It might well have happened that a different target encoding would have required a different grouping of the entries into codes: maybe this encoding had a single code for alpha + rough breathing and acute accent, and another code for the combining subscript iota. In this case, the encoder would have combined both in its result.

Thus, the converter has no predefined conversion path; it just matches two sets of descriptions in the best possible way. Should we have a converter which directly maps any possible combination of source codes to the corresponding target codes, in this case we would have to enter 27 different combinations for the source codes. Instead, we just have to describe each encoding separately, and then let the converter find the match. That's much easier and efficient, and we can just throw a new description in the system to have it handle conversions, in any direction or combination of targets. In fact, describing the meaning of each code is the only task one should accomplish when having to talk about it to a person interested in using the text for his own purposes: something like “this code means that, this other code means that; now take a text encoded with the system, and convert it to something else”.

### Case Use: Font-based Encodings

A typical usage scenario for this converter is represented by what we might call “font-based encodings”. These arose before the Unicode era, even if sometimes they are still used today. In my scenario, this happened especially for ancient Greek texts: when no standard encoding like Unicode was available, and even later due to the survival of legacy technologies, scholars or editors faced the problem of digitally encoding texts which included ancient Greek characters (e.g. a philological edition, a scholarly paper or book, or a Greek dictionary). In this case, apart from older and trickier solutions like Beta code (see below), the practical solution to type Greek text in a word processor was designing an ad-hoc font, where the codes for the Latin-based characters were abused to represent Greek characters. For instance, such a font might place the glyph for the Greek letter alpha at code 97, which in standard Latin encodings from ASCII onwards is the code representing the Latin letter a. This way, you could just type “a” and get an alpha on your screen, assumed that you first installed this font in your system, and selected it before typing. Of course this was a hack: the resulting digital text represented a proprietary encoding, where someone had decided that code 97 stood for alpha (rather than for a). This is an obvious correspondance, but there are hundreths of characters to be encoded for Greek, often involving combinations of several diacritics, and there the choice of codes was purely a matter of personal taste or convenience; and even simple letters might be placed in different ways (e.g. a final sigma: “s” being reserved to sigma, the final form might be found in characters missing from Classical Greek like “j” or “q”).

At any rate, this practice caused a flood of texts encoded with several different Greek fonts, arbitrarily named by their creators, so that it might even be the case that two different fonts had the same name by accident (especially when a simple name like “Greek” was chosen for it). Unless you did not transfer the font with the text itself, you had no way of correctly reading it in another machine. This practice became so popular that even nowadays some journals still recommend the usage of one of these fonts of their choice when one wants to submit a paper including Greek text. Of course, given that so many different fonts were around, a typical issue in collecting papers or other contributions including Greek text was right the painful and error-prone requirement of converting all the Greek text into some other encoding, either standard or proprietary. It was right this situation that led to the first implementation of this system. For instance, I have some 30 font encodings descriptions for the encoder, with more or less exhotic names like Greek, SuperGreek, Sokrates, Aristarchos, Athenian, Graeca, Paulina Greek, MilanGreek, etc.; each with its own specific encoding to represent a slightly different set of characters. Given that these were fonts dating to the 8-bits era (and thus limited to 256 characters), none of them could represent all the characters effectively required for a scholarly Greek text, so that there are several different selections of them according to the purposes of their creator.

A simple converter demo shows how the converter works in this case: you can pick any of these font-based encodings, or just the Unicode encoding, and use it as the source or the target of a text encoding conversion. Of course, you won't see true Greek text unless your system has the corresponding font installed, but it is easy to see it at work. For instance, just type some Unicode letters like `μηνιν` and get their corresponding letters in font Greek: `mhnin`.

>Please note that the font description tables you will find in the demo are there just for demonstration purposes, and probably not all the codes in each font were described or just correctly interpreted by the table creators. At any rate, just changing a bit of text in the faulting table will adjust any conversion.

### Case Use: Meta-Encodings like Beta Code and SAMPA

Another scenario for the usage of this converter is provided by converting [Beta code](https://en.wikipedia.org/wiki/Beta_Code), a legacy rich text encoding format created in the 1970s to digitally represent ancient Greek, Latin, and Hebrew texts, with all the philological annotations typical of a scholarly edition.

This is not just a text encoding, but also a way of encoding rich text; anyway, the part of my Beta converter dealing with text encoding uses this converter to do part of its job. In fact, most part of the character encoding of Beta code might well be described in a way similar to the one about fonts above: for instance, in Beta code, ASCII letters are used to represent Greek letters in specific contexts.

Consider a word like `μῆνιν`: this gets encoded like `$MH=NIN`, where the dollar sign means that what follows must be interpreted as Greek, and each ASCII letter stands for a corresponding Greek letter. Further, the equal sign represents the circumflex accent, applied to the preceding character. Thus, it would be easy to describe such Beta characters with table entries like:

```txt
; Latin H is eta
0048=0397
; Latin I is iota
0049=0399
; Latin M is my
004D=039C
; Latin N in ny
004E=039D
; Equals sign is combining circumflex accent 
003D=0000/0342
...
```

This would be true only for those parts of the Beta text marked as Greek (using some escapes like `$`), but this just means that we should use two tables: one for Latin, and another for Greek. This is not different than converting two font-based encodings in a source text.

Of course, a full-fledged Beta code conversion has much more aspects to be taken into account, but these enter in the realm of a larger text format (and not only text encoding) conversion system, which you can see at work in the demo app. What matters here is that the encoding conversion is an independent subsystem, modular and generic enough to be applied to several different scenarios.

A similar case for its usage is in fact represented by [SAMPA](https://en.wikipedia.org/wiki/Speech_Assessment_Methods_Phonetic_Alphabet), which too uses ASCII characters, to represent the international phonetic alphabet. As above, the converter you find in the demo uses a description table at its core, integrated into a larger system.

### Case Use: Transliteration

Another case use you can find in the demo is transliteration. The demo refers to the Greek language, so you can enter a Unicode polytonic (or monotonic) Greek text, and have it transliterated into Latin characters, with all the required diacritics.

Again, this is another case where the encoding subsystem is integrated in a larger converter: in a sense, it's the inverse of Beta code. There, we use a table to describe Latin characters as Greek characters and diacritics; here, we do the opposite. To better show the flexibility of the system, the demo offers some options for the transliteration; among them, you can even choose if you want to convert to a full Unicode text, or just limit the conversion to the characters available in a legacy 8-bit encoding like [ISO8859-1](https://en.wikipedia.org/wiki/ISO/IEC_8859-1). This is one of the best examples for showing the heuristic nature of the conversion process, which is carried out even when the target encoding is less capable of the source one. In this case, the conversion will sacrifice all these aspects which are incompatible with the target encoding, doing its best to preserve as many aspects as possible. For instance, combinations like macron + accent will be lost, producing only accent, which is the best possible result in that encoding. Just switch the target encoding with Unicode, and you will gain a lossless conversion.

The encoding conversion system is thus totally open and modular: just throw a new encoding in, and you will automatically get the capability of converting from it to any other encoding, or vice-versa.

### Case Use: Typing Greek

A more creative case use I can quote is represented by the system I love to use when typing polytonic Greek texts. Of course, there are many options to ease this task, starting from the standard ones, e.g. using dead-keys combinations with a properly selected keyboard layout. In this case, you just type a key on your keyboard, which apparently does nothing, and then when typing the next one the software combines both inputs into a single character.

For instance, should I want to use the International English keyboard layout to type an accented letter like `á`, I'd just have to press the `'` (apostrophe) key, followed by the a key. This would produce the expected `á` character (should I need the apostrophe itself, I'd have to press the spacebar after it). This is a matter of personal habits; but this method may feel somewhat unnatural, as you first have to type the diacritics, and then the letter; and polytonic Greek has a lot of diacritics combinations, which are hard to remember.

Thus, I used the my conversion system to aid in typing characters with diacritics in an alternative way, similar to the traditional handwriting experience, where you draw a letter and then place the diacritics on it. In this system (I'm naming it Theuth for short), I just type the letter and then press a combination of keys (or click a button on a toolbar) to automatically combine them with the previous character. For instance, I can type `α`, then add a circumflex accent to it (`ᾶ`), then again add a subscript iota (`ᾷ`); the software automatically combines all these diacritics into the shortest sequence of characters available in the Unicode standard. Further, I can later go back to that character, and maybe replace the circumflex with an acute accent by just pressing its keys combination or clicking on its button in the toolbar: the software knows that this accent is not compatible with the circumflex one, so it removes the older accent and adds the new one. The same holds for breathings (rough excludes smooth, and vice-versa), and so forth.

All this can be accomplished by using the conversion system, as fundamentally this is right what is required here: we want e.g. to convert an unaccented alpha letter into an accented one, or the like. Any combination of diacritics can be seen as the conversion from a character with any number of diacritics into that character with any number of other diacritics. We thus need a conversion system which, given e.g. an alpha with an acute accent, knows how to analyze the corresponding Unicode character representing it, decomposing the character from its diacritics; then, the decomposed description of this character is modified by adding or removing diacritics as requested from the user, and this updated description is finally used to recompose a new character.

This is right the decode-encode process performed by the converter sketeched above, and this is the mechanism used to ease the experience of typing Greek (or other languages) texts. You can play with it by downloading and installing the free Theuth demo. Ultimately, this is just a real-time conversion where a set of entries decoded from an input text gets modified by the user, and then recomposed into an output text.

▶️ Next:

- [Format Encoding](./proteus/proteus-format.md)

◀️ Previous:

- [Text Encoding](./proteus/proteus-encoding.md)
