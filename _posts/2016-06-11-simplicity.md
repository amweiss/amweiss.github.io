---
title: Simplicity
excerpt: Start small, improve, make progress.
categories: reflection
tags: newbie simplicity
header:
  overlay_image: /assets/images/headers/simple-desk.jpg
  teaser: /assets/images/headers/simple-desk.jpg
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
---

Every project starts from an idea, a simple thought:

> What if...

Shortly after that, things get complicated.
A simple requirement turns into the most epic of epics, the innocent
question of _Could it also..._ leads to scope creep.

Everyone has good intentions, well, almost everyone, but in the end things get more and more complicated.
Regardless of [agile](https://en.wikipedia.org/wiki/Agile_software_development), [waterfall](https://en.wikipedia.org/wiki/Waterfall_model), [incremental](https://en.wikipedia.org/wiki/Incremental_build_model), [lean](https://en.wikipedia.org/wiki/Lean_software_development), or [whatever else](http://scottberkun.com/2007/asshole-driven-development/), simplicity should always be the goal.

I've found that through my career keeping the goal of simplicity is a crucial part of success.
Simple solutions lead to less maintenance, easier build/test/deploy cycles, and faster reaction to the need for change.
Sometimes it's as simple as a choice of language. If you're on a linux system and want to do a quick find and replace in a set of files, go ahead, use perl.

```sh
perl -pi -e 's/Copyright 2015/Copyright 2016/g' file
```

On Windows writing some packaging scripts? Powershell to the rescue!

```powershell
Write-Host "Syncing releases"
& $syncReleases -releaseDir $release_dir -url "https://github.com/amweiss/vigilant-cupcake" -token $env:GitHubToken
Write-Host "Releasifying"
& $squirrel -releasify "$build_dir\vigilantcupcake.$trimmedVersion.nupkg" -releaseDir $release_dir -setupIcon "$src_dir\VC2-nobg-whitecake.ico" -n "/a /f $src_dir\vigilant.pfx /p $env:SigningPass"
```

Sometimes you may have to learn a bit of new syntax to save yourself a lot of headache later.
Unfortunately, this is the easy part. You've spent your whole professional career learning new
tools, showing off slick new tricks to coworkers, and always being the life of the party when you
bring up a new bit shifting trick you learned to save yourself `10ms` of processing on your computation.

Simplicity within organizations and large projects is another beast entirely. You may or may not be familiar with
[Conway's Law](https://en.wikipedia.org/wiki/Conway%27s_law):

>organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations

You'll find that there are then two possible ways to simplify this situation.  Either you accept the fact that you're doing it and work within those constraints, or you restructure you organization to align with your products. Now the latter may seem like a viable option but you can't do it every time your software changes.

I hope that over time I can share some experiences here that will help others or at least be interesting stories. Sometimes I'll just share fun things like here's a picture from the [Architects of Air](http://outerharborbuffalo.com/events/event/architects-of-air-4/)

![Architects of Air](/assets/images/photos/architects-of-air.jpg)

I appreciate [Kaizen](https://en.wikipedia.org/wiki/Kaizen) so this blog too shall change and improve over time, but for now, I'll stop rambling and invite you to check back soon.