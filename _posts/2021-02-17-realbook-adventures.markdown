---
layout: post
title:  "Real book adventures"
date:   2021-02-17
categories: geek-adventure music
---
My hobby projects often combine two or more of my interests, and music has been a recurring theme for me. This post is about music theory, coding and data modelling.

First though, what is a real book? Well you could say it’s the bible of jazz musicians. It’s a book of jazz songs or standards, each standard has title, composer, and a musical score where you can find the melody and chord chart. In short it’s a kind of sheet music.

A chord chart represents harmony. It’s the list of chords, in order, that an accompanist needs to play while the singer sings the words or soloist plays the tune. It’s the harmonic structure that underpins the melody of a song.

So one day I got to wondering what defines a musical style. Now there are lots of different elements to music including rhythm, tempo, orchestration, etc. but one of the key components is harmony. For example many blues tunes need just three chords, whereas some forms of jazz fusion harmonize each note of the melody with a different chord.

It would be kind of neat to be able to model harmonic structure. It might be possible to create a chord chart generator for example based on existing songs. And the first step to building a model is gathering together some data. That is the subject of this post. So let’s make a start by considering some options.

Manual data entry is one, but that would be fastidious, error-prone, and we software developers are lazy by nature. Far too much like hard work for a hobby project.

Scraping online song charts is another possibility, and I had a quick look at parsing sites like this one with Beautiful Soup and a simple script like the one below. There may be some mileage in this line of attack.

```python
soup = BeautifulSoup(html)
spans = soup.find_all("span", attrs = {"data-name": re.compile(".*")})
chords = [span["data-name"] for span in spans]
```

And then I considered using the iReal Pro forums. iReal Pro is a mobile app that plays songs as backing tracks. It’s a great tool for music practice and some nice MIDI audio (apart from being a fan, I’m not in any way connected to the app, but it is really awesome). But the good news here is that there are songbooks available for download, and the data is sufficiently well structured for the app to play them so parsing it should also be possible.

And there is more good news: the format is documented, although closer inspection reveals that the documentation is incomplete and there is a totally undocumented obfuscation thing going on. Now I should mention I am not the first to tread this path, there are already a few repositories on github that decode the format, in particular accompaniser has a nearly complete description of the grammar.

Let’s now take a deeper look at the iReal Pro data format. Each song chart has some meta data and then a description of the score. The example given in the documentation page looks like this:

```
irealbook://Song Title=LastName FirstName=Style=Ab=n=T44*A{C^7 |A-7 |D-9 |G7#5 }
```

However the actual href elements do not follow this URL scheme. For example, the start of the first URL on the forum page after URL decoding looks like this:

<pre><code>
irealb://26-2=Coltrane John==Medium Up Swing=F==1r34LbKcu7ZL7bD4F^7 ZL7F 7-CZL7C...
</code></pre>

At first glance, this looks to follow the same format, but there are a few surprises. First thing to note is that songs are concatenated into a song book, although you can’t see that in the example above.

Next up, there’s something weird going on at the start of the chord progression. The first sequence of characters is not documented and makes no sense at all. The answer is in accompaniser on this line, each song is separated by this sequence of characters:

<pre><code>
irealb://26-2=Coltrane John==Medium Up Swing=F==<strong>1r34LbKcu7</strong>ZL7bD4F^7 ZL7F 7-CZL7C...
</code></pre>

But that’s not the end of the surprises. When you split the URL into songs using the separator sequence, the resulting chord charts are garbage. There is a further undocumented mechanism that scrambles the chord progression. That is also solved by accompaniser here.

And there’s still more. Once you split into songs and unscramble the chord progression, some of the symbols in the chord progression are not documented at all.

Ignoring the scrambling for the moment, each url contains a bunch of song charts, a bit like this:

- URL
  - Song
    - Title
    - Composer (Last name, first name)
    - Style
    - Key Signature
    - Chord Progression
  - Song
  - ...

To decode each URL, I first broke it down into separate songs and split the meta data. You can find some C# code to do this on github. I then ran pandas profiling over the meta-data to see what I could find.

```python
import pandas as pd
from pandas_profiling import ProfileReport

style = 'Catalog'

df = pd.read_csv(
  style + '.csv',
  sep=';'
  usecols=[
    'Style',
    'StyleId',
    'KeySignature',
    'KeySignatureId',
    'TimeSignature',
    'TimeSignatureId'])

profile = ProfileReport(
  df,
  title=style,
  correlations={
    "pearson": {"calculate": True},
    "spearman": {"calculate": True},
    "kendall": {"calculate": True},
    "phi_k": {"calculate": False},
    "cramers": {"calculate": False},
  })

profile.to_file(style + '.html')
```

So to whet your appetite, here is a quick visualization of key signatures filtered by styles:

<iframe src="/realbook" width="100%" height="1050" frameBorder="0">
</iframe>

Key signatures when I filter for styles including the word ‘Pop’ are dominated by keys easy to play on guitar and keyboard (C, D and E). And when I filter for ‘Swing’ styles, I get a different set of key signatures preferred by jazz solo instruments (saxophone Eb, Bb, trumpet Bb).

That’s all for now, but in a future post I will describe how to linearize the chord progression, in other words how to play it like a musician would to produce a single linear sequence of chords. Once that is done, it should be possible to build some kind of model, maybe based on Markov chains, that can generate completely new chord progressions.