---
title: Better reading
date: 2023-11-29 23:04:52
tags:
---

Reading is important. And for me having a good setup to do so is equally important. On the first post of this blog I wrote about {% post_link Looking-for-the-perfect-reading-setup "my journey to find the perfect reading setup"%}. The tl;dr; is:

- I like taking notes and scribbling while I read. That's why I like consuming content in my iPad
- But the Apple Books app for iOS sucks big time. It crashes[^1] frequently and its search and organization capabilities are useless for me
- I switched to [Reader](https://readwise.io/read) and have been using it thus far

[Reader](https://readwise.io/read) is great. The team behind it has done a great job building a tool that covers most of the needs I have. But not the most important.

Over time I realized that most of the benefit I was getting from Reader was from its organization capabilities rather than anything else. There are many of the functionalities that I simply don't use. I don't usually take notes on the documents (with the keyboard), don't use ghostreader, sync notes to a knowledge management app or use [Readwise](https://readwise.io/) to revisit/learn from what I highlight. I tried all of these but none stuck with me. And what's worse, because sometimes the imported content is missing images or other parts of the original, I have developed the tendency of reading the content in their original location, giving away the focus and distraction free environment that Reader provides.

So at this point I'm _almost_ back at square one. Looking for a better reading setup.

## Building my own

This wasn't the first time I was toying with the idea of building my own solution but every time I pushed the idea back because of all the effort required. Or so I thought.

What I truly need is to convert html to _something_ that I can write onto on my iPad and nothing else. Of course, my mind is buzzing with ideas and features that I could add but those are not what I _need_. At least for the first iteration.

So, what's my solution to "convert html to something that I can write onto my ipad"

- "Convert html to something". For this I'll use [Puppeteer](https://pptr.dev/) and [mozilla/readability](https://github.com/mozilla/readability). With Puppeteer I can progamatically generate PDFs from websites and with Readability I can get the "reader" (or decluttered) version of the pages I want to read
- "that I can write onto on my iPad". I'll use iCloud drive to get files into my iPad and [pdfexpert](https://pdfexpert.com/)[^2] to annotate and scribble by hand

## In use

To convert the documents I use either a command line utility or a chrome extension

![chrome-extension](chrome-extension.png)

In the true spirit of "progress over perfection" I took the [example extension](https://developer.chrome.com/docs/extensions/mv3/getstarted/development-basics/) that google uses in their tutorial and simply added a button to it to trigger the page conversion. It works and that's enough for now.

![folder-layout](folder-layout.png)

To help a bit with the organization I store the PDFs in folders per domain.

![pdf-annotations](pdf-annotations.png)

And finally I can go back to reading articles on my iPad, taking notes with the pencil. And I can open the same PDFs from my mac or iPhone and see the same annotations.

[^1]: [1](https://discussions.apple.com/thread/254695725), [2](https://discussions.apple.com/thread/254524185) and [3](https://discussions.apple.com/thread/253766874) examples of what I mean
[^2]: I tried [Goodnotes](https://www.goodnotes.com/) and [Notability](https://notability.com/) too. Even though PDFExpert is slightly more expensive it seemed to me the more polished of the pack.
