---
id: 01J630QT1HYQB03JMPA7P0718N
modified: 2024-08-28T10:53:18-04:00
title: Chapter 1 - Introduction
description: Chapter 1 of Software design by Example Python
tags:
  - programming
  - python
  - software-design
  - sdbe
---
# Chapter 1 - Introduction
- ## Section 1.1 - The Audience
- _Maya has a master’s degree in genomics. She knows enough Python to analyze data from her experiments, but struggles to write code other people can use. These lessons will show her how to design, build, and test large programs in less time and with less pain._
	- This is who this book is for
- Like Maya, you should be able to:
	- Write Python programs using lists, loops, conditionals, dictionaries, and functions.
	- Puzzle your way through Python programs that use classes and exceptions.
	- Run basic Unix shell commands like `ls` and `mkdir`.
	- Read and write a little bit of HTML.
- Use [Git](https://git-scm.com/) to save and share files. (It’s OK not to know [the more obscure commands](https://git-man-page-generator.lokaltog.net/).)
- These chapters ([Figure 1.1](https://third-bit.com/sdxpy/intro/#intro-syllabus)) are also designed to help another persona:
> _Yim teaches two college courses on software development. They are frustrated that so many books talk about details but not about design and use examples that their students can’t relate to. This book will give them material they can use in class and starting points for course projects._

![[Pasted image 20240824162358.png]]
Our approach to design is based on three big ideas. First, as the number of components in a system grows, the complexity of the system increases rapidly ([Figure 1.2](https://third-bit.com/sdxpy/intro/#intro-complexity)). However, the number of things we can hold in working memory at any time is fixed and fairly small [[Hermans2021](https://third-bit.com/sdxpy/bib/#Hermans2021)]. If we want to build large programs that we can understand, we therefore need to construct them out of pieces that interact in a small number of ways. Figuring out what those pieces and interactions should be is the core of what we call “design”.

![Complexity and size](https://third-bit.com/sdxpy/intro/complexity.svg)

Figure 1.2: How complexity grows with size.

Second, “making sense” depends on who we are. When we use a low-level language, we incur the [cognitive load](https://third-bit.com/sdxpy/glossary/#gl:cognitive_load "The mental effort required to solve a problem.") of assembling micro-steps into something more meaningful. When we use a high-level language, on the other hand, we incur a similar load translating functions of functions into actual operations on actual data.

More experienced programmers are more capable at both ends of the curve, but that’s not the only thing that changes. If a novice’s comprehension curve looks like the lower one in [Figure 1.3](https://third-bit.com/sdxpy/intro/#intro-comprehension), then an expert’s looks like the upper one. Experts don’t just understand more at all levels of abstraction; their _preferred_ level has also shifted so they find �2+�2 easier to read than the Medieval equivalent “the side of the square whose area is the sum of the areas of the two squares whose sides are given by the first part and the second part”. This curve means that for any given task, the code that is quickest for a novice to comprehend will almost certainly be different from the code that an expert can understand most quickly.

![Comprehension curves](https://third-bit.com/sdxpy/intro/comprehension.svg)

Figure 1.3: Novice and expert comprehension curves.

Our third big idea is that programs are just another kind of data. Source code is just text, which we can process like other text files. Likewise, a program in memory is just a data structure that we can inspect and modify like any other. Treating code like data enables us to solve hard problems in elegant ways, but at the cost of increasing the level of abstraction in our programs. Once again, finding the balance is what we mean by “design”.
## Section 1.3: Formatting

](https://github.com/gvwilson/sdxpy/issues/new?title=/intro%20-%20Section%201.3:%20Formatting%20(2024-04-28))

We display Python source code like this:
```python
for ch in "example":
	print(ch)
```

[python_sample.py](https://third-bit.com/sdxpy/intro/python_sample.py)

and Unix shell commands like this:

```bash
for filename in *.dat 
do     
	cut -d , -f 10 $filename 
done
```


[shell_sample.sh](https://third-bit.com/sdxpy/intro/shell_sample.sh)

Data files and program output are shown like this:
```yml
-name: read
	params:
	- sample_data.csv
```

[data_sample.yml](https://third-bit.com/sdxpy/intro/data_sample.yml)

```output
alpha
beta
gamma
delta
```
[output_sample.out](https://third-bit.com/sdxpy/intro/output_sample.out)

We use `...` to show where lines have been omitted, and occasionally break lines in unnatural ways to make them fit on the page. Where we do this, we end all but the last line with a single backslash `\`. Finally, we show glossary entries in **bold text** and write functions as `function_name` rather than `function_name()`. The latter is more common, but the empty parentheses makes it hard to tell whether we’re talking about the function itself or a call to the function with no parameters.

[

## Section 1.4: Usage

](https://github.com/gvwilson/sdxpy/issues/new?title=/intro%20-%20Section%201.4:%20Usage%20(2024-04-28))

The source for this book is available in [our Git repository](https://github.com/gvwilson/sdxpy/) and all of it can be read on [our website](https://third-bit.com/sdxpy/). All of the written material in this book is licensed under the [Creative Commons - Attribution - NonCommercial 4.0 International license](https://creativecommons.org/licenses/by-nc/4.0/) (CC-BY-NC-4.0), while the software is covered by the [Hippocratic License](https://firstdonoharm.dev/). The first license allows you to use and remix this material for noncommercial purposes, as-is or in adapted form, provided you cite its original source; if you want to sell copies or make money from this material in any other way, you must [contact us](mailto:gvwilson@third-bit.com) and obtain permission first. The second license allows you to use and remix the software on this site provided you do not violate international agreements governing human rights; please see [Appendix D](https://third-bit.com/sdxpy/license/) for details.

If you would like to improve what we have, add new material, or ask questions, please file an issue in [our GitHub repository](https://github.com/gvwilson/sdxpy/) or [send an email](mailto:gvwilson@third-bit.com). All contributors are required to abide by our Code of Conduct ([Appendix E](https://third-bit.com/sdxpy/conduct/)).

[

## Section 1.5: What People Are Saying

](https://github.com/gvwilson/sdxpy/issues/new?title=/intro%20-%20Section%201.5:%20What%20People%20Are%20Saying%20(2024-04-28))

Here’s what people said about the JavaScript version of this book [[Wilson2022a](https://third-bit.com/sdxpy/bib/#Wilson2022a)]:

- [Jessica Kerr](https://jessitron.com/2023/02/20/book-review-software-design-by-example/): “_Software Design by Example_ is the book I’ll recommend to every new dev… It is nice to you. It wants you to succeed… It’s a bridge from ‘learn to program’ to working programmer.”
    
- [Jenn Schiffer](https://mastodon.social/@jenn@pixel.kitchen/109985276835264400): “I am v much enjoying gvwilson’s book _Software Design by Example_. It makes me miss teaching, it would be such a fun text to use!”
    
- [Emily Gorcenski](https://emilygorcenski.com/post/book-report-software-design-by-example-by-greg-wilson/): “There’s a lot of books on programming but fewer books that couple software development with effective and practical use of tools, presenting a language not as a main course but as a part of an engineering ecosystem. Greg Wilson’s book hits all the right notes in bringing together theory, pragmatism, and best practices.”
    
- [Danielle Navarro](https://blog.djnavarro.net/posts/2023-05-31_software-design-by-example/): “The book is really bloody lovely.”
    

[

## Section 1.6: Acknowledgments

](https://github.com/gvwilson/sdxpy/issues/new?title=/intro%20-%20Section%201.6:%20Acknowledgments%20(2024-04-28))

Like [[Wilson2022a](https://third-bit.com/sdxpy/bib/#Wilson2022a)], this book was inspired by [[Kamin1990](https://third-bit.com/sdxpy/bib/#Kamin1990), [Kernighan1979](https://third-bit.com/sdxpy/bib/#Kernighan1979), [Kernighan1981](https://third-bit.com/sdxpy/bib/#Kernighan1981), [Kernighan1983](https://third-bit.com/sdxpy/bib/#Kernighan1983), [Kernighan1988](https://third-bit.com/sdxpy/bib/#Kernighan1988), [Oram2007](https://third-bit.com/sdxpy/bib/#Oram2007), [Wirth1976](https://third-bit.com/sdxpy/bib/#Wirth1976)] and by:

- [_The Architecture of Open Source Applications_](https://aosabook.org/) series [[Brown2011](https://third-bit.com/sdxpy/bib/#Brown2011), [Brown2012](https://third-bit.com/sdxpy/bib/#Brown2012), [Armstrong2013](https://third-bit.com/sdxpy/bib/#Armstrong2013), [Brown2016](https://third-bit.com/sdxpy/bib/#Brown2016)];
- [Mary Rose Cook’s](https://maryrosecook.com/) [Gitlet](http://gitlet.maryrosecook.com/);
- [Matt Brubeck’s](https://limpet.net/mbrubeck/) [browser engine tutorial](https://limpet.net/mbrubeck/2014/08/08/toy-layout-engine-1.html);
- [_Web Browser Engineering_](https://browser.engineering/) by [Pavel Panchekha](https://pavpanchekha.com/) and [Chris Harrelson](https://twitter.com/chrishtr);
- [Connor Stack’s](https://connorstack.com/) [database tutorial](https://cstack.github.io/db_tutorial/);
- [Maël Nison’s](https://arcanis.github.io/) [package manager tutorial](https://classic.yarnpkg.com/blog/2017/07/11/lets-dev-a-package-manager/);
- [Paige Ruten’s](https://viewsourcecode.org/) [kilo text editor](https://viewsourcecode.org/snaptoken/kilo/index.html) and [Wasim Lorgat’s](https://wasimlorgat.com/) [editor tutorial](https://github.com/seem/editor);
- [Bob Nystrom’s](http://journal.stuffwithstuff.com/) [_Crafting Interpreters_](https://craftinginterpreters.com/) [[Nystrom2021](https://third-bit.com/sdxpy/bib/#Nystrom2021)]; and
- the posts and [zines](https://wizardzines.com/) created by [Julia Evans](https://jvns.ca/).

I am grateful to Miras Adilov, Alvee Akand, Rohan Alexander, Alexey Alexapolsky, Lina Andrén, Alberto Bacchelli, Yanina Bellini Saibene, Grigoriy Beziuk, Matthew Bluteau, Adrienne Canino, Marc Chéhab, Stephen Childs, Hector Correa, Socorro Dominguez, Christian Drumm, Christian Epple, Julia Evans, Davide Fucci, Thomas Fritz, Francisco Gabriel, Florian Gaudin-Delrieu, Craig Gross, Jonathan Guyer, McKenzie Hagen, Han Qi, Fraser Hay, Alexandru Hurjui, Finnur Agust Ingimundarson, Bahman Karimi, Carolyn Kim, Kitsios Konstantinos, Jenna Landy, Peter Lin, Zihan Liu, Becca Love, Dan McCloy, Ramiro Mejia, Michael Miller, Firas Moosvi, Joe Nash, Sheena Ng, Reiko Okamoto, Joshua Ossai, Juanan Pereira, Mahmoodur Rahman, Arpan Sarkar, Silvan Schlegel, Dave W. Smith, Stephen M. Sturdevant, Diyar Taskiran, Ece Turnator, and Yao Yundong for feedback on early drafts of this material.

I am also grateful to Shashi Kumar for help with LaTeX, to [Odin Beuchat](https://www.drafolin.ch/) for help with JavaScript, and to the creators of [Black](https://black.readthedocs.io/), [flake8](https://flake8.pycqa.org/), [Glosario](https://glosario.carpentries.org/), [GNU Make](https://www.gnu.org/software/make/), [isort](https://pycqa.github.io/isort/), [ark](https://www.dmulholl.com/docs/ark/main/), [LaTeX](https://www.latex-project.org/), [pip](https://pip.pypa.io/), [Python](https://www.python.org/), [Remark](https://remarkjs.com/), [WAVE](https://wave.webaim.org/), and many other open source tools: if we all give a little, we all get a lot.

All royalties from this book will go to the [Red Door Family Shelter](https://www.reddoorshelter.ca/) in Toronto.