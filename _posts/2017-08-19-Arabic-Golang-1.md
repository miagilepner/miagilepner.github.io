---
layout: post
title: 'Arabic Transliteration in Golang - Part 1'
---

I majored in both Arabic and Computer Science in college, and regret that I didn't spend more of my time experimenting with the overlap between those two fields. I'd love to learn more about how computer science is taught in Arabic speaking countries. I'd also like to do technical work with Arabic, such as writing an Arabic language compiler. The project [ﻗﻠﺐ](http://nas.sr/%D9%82%D9%84%D8%A8/) (pronounced 'Qalb') is a Scheme-like interpreted language entirely in Arabic. It's a really interesting project, particularly because it incorporates Arabic calligraphy in the language, combining the technical process of writing code with artistic output.

I'm not quite ready to make an Arabic compiler (though after taking Berkeley's compilers class I might be a little bit closer to ready). Instead, I decided to make an Arabic transliterator. When I had to type papers in Arabic, I would change my keyboard to be Arabic, then hunt and peck for the right keys. Using a transliteration program would be a lot faster. I also decided to implement my program in Golang.

To explain how the transliterator works, I need to explain a little about how Arabic works. There are 28 letters in Arabic. Some of those letters are consonants, and some of them are long vowels. "Long" just means that when spoken, the vowels are stressed. In addition to those letters, there are short vowels. Short vowels float on top or below of the main 28 letters.<sup>[1](#short-vowels)</sup>

So, there's Unicode for the 28 different letters, as well as Unicode for the short vowels. Arabic is written right-to-left, but Unicode is interpreted left-to-right. That means that the first letter in an Arabic word (aka the rightmost character) should be the *leftmost* Unicode element.<sup>[2](#rtl)</sup> So, the Unicode corresponding to ع ر ب is
```
(U+0639) (U+0631) (U+0628)
```
where `(U+0639)` is the ع in that word. That word is written without the short vowels. So even though it's pronounced "araba", nothing about the way it's written tells you about the pronounciation. If you wanted to distinguish "araba" from "ariba," you would need to add in the short vowels. The vowels go on top of each letter as [combining characters](https://en.wikipedia.org/wiki/Combining_character). Because of the right-to-left nature of Arabic, the combining character must be to the left of the character that it's on top of. So, to provide the proper vowels we would use the Unicode
```
(U+064E) (U+0639) (U+064E) (U+0631) (U+064E) (U+0628)
```
and get َ ع َ ر َ ب. Simple enough, right?

Except there's one thing I haven't mentioned yet. Arabic words have each letter connected, forming a cursive-like script. Except, unlike in English, there's no such thing as non-cursive writing. The words I listed above are actually meaningless, because the letters aren't connected. Oftentimes, word processors recognize that the letters should be connected and switch the letter to be the proper, connected form. But sometimes, it doesn't happen.

![id]({{site.baseurl}}/public/arabic-tattoo.jpg)
*A very unfortunate tattoo <sup>[3](#tattoo)</sup>*	

Each letter in Arabic has 5 forms. We've been using the general Unicode so far. But to insure that the text renders correctly, we should be using the extended forms. Letters can be in the front of a word, in the middle of the word, or at the end of a word, and there are 3 forms per letter corresponding to those locations. There is also an isolated form, which is equivalent to the standalone general form. 

The proper way to write araba is ََﻋَﺮَﺏ and uses the Unicode 
```
(U+064E) (U+FECB) (U+064E) (U+FEAE) (U+064E) (U+FE8F)
```

Now that we understand how Arabic letters work in Unicode, we need to start thinking about how they can be represented in English. There are two issues that make transliteration challenging:
1. Arabic has sounds that don't exist in English. For example, the letter \`ayn is a deeply throaty sound, often very hard for English speakers to reproduce. To get an idea of what I mean, try emphasizing the "a" in the word "father." Make the "a" as deep as it can possibly go. That's about what an \`ayn sounds like. 
2. Arabic has letters that are combinations of two english letters. For example, Arabic uses the letter shin (pronounced like "sheen") to denote the "sh" sound in English.

This makes the mapping from English to Arabic more difficult. Thankfully, I didn't have to figure out a scheme myself. I used the [Qalam transliteration system](http://langs.eserver.org/qalam.txt), which is known for its simplicity. It's fairly intuitive for me to think of an Arabic word, then find the proper English representation in this scheme. 

| Arabic letter | English representation | Arabic letter | English representation | Arabic letter | English representation |
|----------------|------------------------|----------------|------------------------|-----------------|------------------------|
| 'alef | aa | zayn | z | qaaf | q |
| baa' | b | syn | s | kaaf | k |
| taa' | t | shyn | sh | laam | l |
| thaa' | th | Saad | S | mym | m |
| jym | j | Daad | D | nuwn | n |
| Haa' | H | Taa' | T | haa' | h |
| khaa' | kh | Zaa' | Z | waaw | w |
| daal | d | &#96;ayn | &#96; | yaa' | y |
| dhaal | dh | ghayn | gh | raa' | r |
| faa' | f |  |  |  |  |
| taa' marbuwTah | t or h | haa' marbuwTah | h | 'alef maqSuwrah | ae |
| hamza | ' | hamzat alwaSl | e |  |  |

Here are the short vowels and other diacritics:

| Arabic diacritic | English representation |
|------------------|------------------------|
| fatHah           | a                      |
| kasrah           | i                      |
| Dammah           | u                      |
| shaddah          | double previous letter |
| maddah           | ~aa                    |
| sukuwn           | -                      |
| tanwyn           | N                      |
 

Using the transliteration scheme, we can write ََﻋَﺮَﺏ as \`araba. Not too different from what we started with!

Alright, this provided some intro and background on the problem I was hoping to solve. Look for part 2 where I delve into Golang and the code I used to make my transliterator.

---
<a name="short-vowels">1</a>: Even on long vowels!<sup>[a](#disclaimer)</sup>

<a name="rtl">2</a>: I'm not going to get into punctuation and Unicode bidirectionality.

<a name="tattoo">3</a>: [Here's](http://www.gettysburgcollegeitt.org/LRCblog/2014/12/when-arabic-tattoos-go-bad-or-why-you-should-talk-to-a-translator-first/) some discussion of that image.

<a name="disclaimer">a</a>: Sort of. But for this post, let's assume that's true.
