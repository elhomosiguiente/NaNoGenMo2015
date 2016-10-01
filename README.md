# NaNoGenMo2015
"Choice of Someone Else's Adventure", my NaNoGenMo project for 2015.

NaNoGenMo2015: Choice of Someone Else’s Novel
  BY CAROLYN VANESELTINE DECEMBER 1, 2015 BLOG, CRAFT & TECHNIQUE
NaNoGenMo, aka National Novel Generation Month, is Darius Kazemi’s brainchild where people create procedural generation systems to make novels, starting November 1 and ending November 31.

I got really excited about doing NaNoGenMo this year. My goal was to create a system in C# that would read from a sample text to produce interactive novels in ChoiceScript.

And I did! Here’s the final project at GitHub: Choice of Someone Else’s Novel. And, for those who want to cut to the chase, here’s a game it made: “Saving?”

But… November included my wedding, my honeymoon, Thanksgiving, and a family emergency. I’ve brought some projects in with bumpy landings, but this was one of the bumpiest. In a “if I hadn’t talked about this already, I might not admit to it” way. I was very tempted to bury it away and pretend this never happened until a later, more polished version – except that I have other stuff I need to do, and I won’t get back to it.

So I’m just going to show it off (while keeping some good advice about game jams firmly in mind.)


(Yeah, I use this quote a lot. But it keeps being applicable to my life.)

The game itself
To play an example – go here to see “Saving?”, produced last night by CoSEN. Just to set expectations, it has poetic prose at best, and the chapters are ludicrously short due to working around a bug.

The Markov chain source text for “Saving?” is Caelyn Sandel’s short story “Mara and the Bottleman”. The original needs a for violence and gore, but the same doesn’t apply to “Saving?” (though there’s a distinctly creepy tone).

Source files for “Saving?”
startup.txt
choicescript_stats.txt
chapter1.txt
chapter2.txt
chapter3.txt
(continues to chapter10.txt)

How CoSEN works
There are four major parts:

Breaking down the source text
Generating the outline
Writing prose based on the source text
Converting the outline into ChoiceScript
If you’d like to see the C# source, it’s available here: https://github.com/cvaneseltine/NaNoGenMo2015  There’s also a copy of the latest CS build and the Moby Part-Of-Speech dictionary in bin/Debug, which are necessary for the system to run.

1) Breaking down the source text
Prosebreaker.cs reads the file in and breaks it down, first sentence by sentence and then word by word. Each word gets checked against a dictionary (the Moby Part-of-Speech) to determine what part of speech it is. As the sentence structure is uncovered, the system builds a grammar map that includes the grammar for each sentence (with notes about what punctuation can reasonably fit there). Every word is also filed in a dictionary so that they can be retrieved based on their parts of speech. Additionally, it peels out some adjectives to become ChoiceScript stats later on, and attempts to recognize proper nouns for the same purpose.

The end result (holding all the vocabulary, parts of speech, etc) is called a “novel” in the source code, which is a fairly awful name for a class that means “deconstructed novel” as opposed to “constructed novel”.

The prosebreaker could use hardening. If you use a source text from Project Gutenberg, it t throws a “STOP STOP STOP STOP” message and then crashes. (I was originally using this as a debug marker.) It will likely have trouble with a variety of other perfectly reasonable texts, which has something to do with improperly processing whitespace.

2) Generating the outline
This was far and away the most complicated part of this system, and I really should have diagrammed first, programmed it second. But I didn’t. (Twice.)

My initial architecture compiled and worked marvelously and was totally wrong. It assumed that every page ended with a choice, and while it built beautiful recursive choice maps (branching and rejoining and everything!) – it had no way to actually check stat values and redirect the player based on stat values. And it wasn’t flexible enough for me to add that ability.

I ripped it all out and started again.

My second architecture round was more flexible, but also more complicated. It looked something like this:

The outline has a number of chapters.
Each chapter has a number of sections.
A section has a label and a section end.
A section end can be a choice, a stat check (aka if statement), or an actual ending.
Each section end has a list of options.
Each option leads to a different label.
Rather than having all these pieces build themselves, I created “architects” to do it in an organized fashion. There’s probably a name for this design pattern, but I thought of them as game-style AI.

A chapter creates an architect.
The architect creates a section, a section end, and options.
The original architect follows one option itself, and spawns clones to follow the other options.
All the existing architects built new sections, until they reach an appropriate section depth.
Each architect rinses and repeats until it has built/visited a predetermined number of sections.
As each architect finishes, it passes its list of sections over to the chapter.
Ideally, the number of sections per chapter should be 50+. In practice, it’s 3, because everything breaks above that number. I didn’t become aware of this bug until about 10 PM last night, since I had everything dialed down to produce small games for testing purposes. I tried to raise the number and – crash. If I were going to fix anything here, it would be this.

There’s also a spackled-over bug that has something to do with architects not handing in their sections correctly at the end. It’s been spackled over by passing sections to the chapter as they’re created, rather than waiting until the end. But the original code is still all present and functioning.

On a design front, the frequency of section end types swings from “more choices” to “more if statements” as you go deeper in chapters. But the swing is much too sharp – some later chapters have no choices at all! With more sections per chapter, this would be less pronounced, but as is… yeah. Not ideal.

3) Writing prose based on the source text
Basic idea:

Choose a random word that was used as the first word of a sentence in the original source text.
Take a look at that word’s part-of-speech.
Move down the grammar map to a part-of-speech that can follow the original word.
Choose a random word that has that part of speech.
Check to see if punctuation follows that part-of-speech, and add it in if so.
Continue until reaching the end of a sentence.
There are a few overarching classes: paragraphs are made of several sentences, and pages are made of several paragraphs.

The result is sometimes coherent: “Just understand, somehow.”

and sometimes less coherent: “Mara blank rum two later, such trigger over any throw, its torture got distraction, flash whatever shoes a glowing spray. Whining that love?”

After this Markov chain base, I got fancy. I set up systems to alter tense and person and detect pluralizations and swap in characters and pronouns. I spent far too much time working on this, and it made everything much, much worse. You won’t see most of it in the source because I killed it with fire. (If I ever need it again, that’s what version control is for.)

I wish the quotation marks weren’t so horrible. I kept starting to smooth them out, but there was always something else to do.

4) Converting the outline into ChoiceScript
The startup file is generated from the chapter list.
The choicescript_stats file is generated from the stats list.
…and then it’s just a matter of filling in the outline with associated text.
As long as the outline is happy, everything works. (Which isn’t necessarily true. Sadly. But that’s the theory!)

BOOKMARK THE PERMALINK.
