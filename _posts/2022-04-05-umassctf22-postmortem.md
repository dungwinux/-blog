---
layout: post
title: "UMass CTF 2022 Postmortem"
date: 2022-04-05 04:00:00 +0000
categories: security
tags: ctf
---

_Inspired by the [previous year postmortem](https://jakob.space/blog/umass-ctf-2021-postmortem.html) by Jakob._

This is the second year of our capture-the-flag event, [UMassCTF 2022](https://ctftime.org/event/1561).
The time frame of this year's event is _Fri, 01 April 2022, 22:00 UTC — Sun, 03 April 2022, 22:00 UTC_.
Thanks to the timing, it overlaps with four other CTFs, and so many teams either couldn't join or had to spread thin.
Despite that, **314** teams solved at least one challenge, and **23/26** flags were captured successfully.
I hope everyone enjoyed the CTF.

This post reflects on my perspective of the event, from making the challenges to supporting during the event.

Thanks to everyone in **UMass Cybersecurity Club** for assisting me in completing these challenges and resolving tickets.
Thank you to my parents, who always supported me, my pursuit of this event, and my dream.

## Challenge design

Last year I was a participant; this year I became a challenge contributor. The event gave me lots of experience with challenge design and insights into how people solve challenges. Below are discussions on them, sorted in the order of first blood time.

### tino

> I'm trying to write the program to print out the flag,
> with only one bug left. However, before I could fix it,
> my future me stole it and obfuscated it
> ~~, preventing the route to dystopia~~.
> Can you help me get back the source code?
> And promise you won't look at the flag?

#### Comment

_Yes, I love time leap stories._
I came up with this challenge one day before CTF started. The response to this challenge is better than I expected. I hope people enjoy solving this and learning about the C++ language, even just a little bit.

#### Links

Challenge static file:
[UMassCybersecurity:UMassCTF-2022-challenges/misc/tino/static](https://github.com/UMassCybersecurity/UMassCTF-2022-challenges/tree/main/misc/tino/static)

Generator source code: https://github.com/dungwinux/tino

<!-- Add write-ups here -->

#### Design

The given challenge file is a C++ source code _obfuscated_ with C/C++'s usual `#define`.
The original source code is a flag printer written in C++20.
There is one bug in the lambda expression passed to `std::views::filter()`,
which should return true on odd instead of even number.

The obfuscation idea is based on the fact that C++ preprocessor directive [`#define`](https://en.cppreference.com/w/cpp/preprocessor/replace) allows replacing every keywords, constants, whole strings, ... in normal source code.
Aligned with that, `_` (underscore) is the only non-alphanumeric ASCII character that can be used as identifier in `#define`.
For strings obfuscation, the challenge used a less-known C/C++ [feature](https://en.cppreference.com/w/c/language/string_literal#Explanation): the prepared source code have strings separated each character into their own `""` bracket
(i.e: `"Hello"` must be rewritten as `"H" "e" "l" "l" "o"`.)

#### Solution discussion

_Solve count: 35_

There are many ways to solve the challenge, so I will only highlight some appealing takes.

An issue came up among some flag capturers: the flag they got has a lot of `,` (commas).
After looking into it, I realized a twist that was ignored when replacing by hand: `#define` does not replace characters in strings.
Therefore, you need to craft a decent regex or perform eye-straining manual checking to successfully deobfuscate the source code.
It was unintended when I wrote the challenge, but it definitely avoided simple _copy-paste_ solution.

For the bug fixing part, it sounds like you need a working compiler and C++20 knowledge, but no.
The `std::views` that appeared in the source is fundamentally High-Order Function + Pipe-forward.
A solution of rewriting the code into another language is possible. Some teams even modify and debug right on that source code, which is a creative solution outside of my expectation.

### mere

> Who loves games?
> Has anyone tried to open up a game directory and see what's inside?
> I do. I'm pretty sure some others do too.
> This challenge is _mere_-ly based on that. Get it?

Hint:

> I was supposed to give out this document from ■■■ game engine along with the file.
> However, the physical version is damaged, and I don't have a digital backup.
> This should look clear enough, right? [mere_hint.png](/assets/images/mere_hint.png)

#### Comment

_I think everyone get it; it's a lame joke._
This challenge is inspired by many game engines' archives (Ever heard of UniExtract, GARbro?).
I don't know whose idea to make each engine has its way to store assets, but it's certainly not easy to work on each for thousands of them.
I hope people who attempted to solve this can get one or two insights into reversing binary structure.

#### Links

Challenge static file:
[UMassCybersecurity:UMassCTF-2022-challenges/misc/mere/static](https://github.com/UMassCybersecurity/UMassCTF-2022-challenges/tree/main/misc/mere/static)

Generator source code: https://github.com/dungwinux/myr_archive

Official writeup: _To be added_

<!-- Add write-ups here -->

#### Design

It took me at least a month to have a decent proof of concept of the challenge. Other than the inspiration, the challenge archive was made from scratch. It contains and encrypts files with the weak XOR encryption (For once, I wanted to put rotation like some formats, but then decided not to do so since it would take too much time guessing).

Most of the time was spent figuring out how and where to hide the key.
In the final design, the key for the encryption is not transparent. You have to get it through the given file: The image prefixed in the archive is the _way_ to get the key.
The pattern is the key repeated and reset after every 4096 bytes due to buffered processing: the code copies and processes every 4096 bytes rather than doing it byte-by-byte on files.

#### Solution discussion

_Solve count: 1_

The original solution is to either use Kaitai Struct or Python's `struct` library to describe the structure and extract the content.
Not many people know Kaitai Struct, so I expect people to approach this challenge using Python.

Without the file structure given to the player, it seems too demanding to guess what it looks like. Although brute-force sounds applicable, you must at least know what you are looking for, or it would just be `O(infinity)`. Therefore, after Day 1 + 1h25m, I published a partial structure specification as a hint. I'm glad that one team captured this flag because every solution and writeup are signals for me to know what to improve.
(Note to future self: I should look into the amount of guessing a player has to go through.)

## Ticket

If you participated in the event and opened a ticket at around
_04:00 UTC — 13:00 UTC_,
then you might have received a response from me.
Many queries are of challenges that the authors were not active (_it was nighttime_),
so I could only reply to those tickets with something along the line of
_The author of the challenge is offline. He will reach you when he is back online_.
Sorry for the inconvenience, but I hope that keeps the CTF responsive.

I have never figured out before how helpful the ticket system can be.
There were two quizzes in this year's CTF, and most of the tickets asked about `JeopardyV2`. The issues were commonly question misunderstanding or the answer was so close but unexpected.
If we are going to do this again next year, we should add _answer_ coverage measure.
Several flag capturers have also used the ticket system to report issues with solutions and infrastructure.
Thanks to that, we could fix the issues early and had the event run smoothly throughout the two days.

## Other lessons learned

-   Challenge Update Transparency: This was at least improved compared to the last year, although the problem is still there. Notification in CTFd was used but did not sync with the _#announcement_ channel in Discord. Perhaps we should prepare a bot to sync them for next year's.
-   Better timing: We did not put the time on the calendar early, so the overlapping was inevitable and the number of teams is incomparable to last year. For next year event, time and date will be added early.
