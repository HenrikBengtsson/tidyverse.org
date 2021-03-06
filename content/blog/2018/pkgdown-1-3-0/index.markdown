---
title: 'pkgdown 1.3.0'
author: Mara Averick
date: '2018-12-10'
slug: pkgdown-1-3-0
description: > 
  pkgdown 1.3.0 is now on CRAN!
categories:
  - package
tags:
  - pkgdown
  - r-lib
photo:
  url: https://pixabay.com/en/decoration-packages-gifts-ribbons-3229259/
  author: jackmac34
---



We're happy to announce that [pkgdown](https://pkgdown.r-lib.org/) 1.3.0 is now
available on CRAN. pkgdown is designed to make it quick and easy to build a
website for your package. Here, we'll highlight some of the new features, and
improvements in this version. Note, this blog post describes pkgdown 1.2.0
and 1.3.0, because we accidentally released 1.3.0 instead of 1.2.1. For a full
list of changes, please see the
[NEWS](https://github.com/r-lib/pkgdown/blob/master/NEWS.md#pkgdown-120).

## New features

The new
[`deploy_site_github()`](https://pkgdown.r-lib.org/reference/deploy_site_github.html)
function can be used to automatically deploy your package website to GitHub
Pages with continuous integration systems (like
[travis](https://travis-ci.org/)). Setup details can be found
[here](https://pkgdown.r-lib.org/reference/deploy_site_github.html#setup). We
are gradually moving all tidyverse sites to use this process so that they're
always up-to-date.

[`build_favicon()`](https://pkgdown.r-lib.org/reference/build_favicon.html)
auto-detects the location of your package logo, and runs it through the
<https://realfavicongenerator.net> API to build a complete set of favicons with
different sizes.

Lastly, users with limited internet connectivity can now expressly disable
pkgdown's internet usage by setting `options(pkgdown.internet = FALSE)`.

## Front-end improvements

All third-party resources are now fetched from a single CDN and are given an SRI
hash. The package version displayed in the navbar now has `class="version"`,
which should make it easier to customize its appearance. The default footer now
displays the version of pkgdown used to build the site. You'll need to run this
once and check in the generated files.

## Rd translation

  - [`rd2html()`](https://pkgdown.r-lib.org/reference/rd2html.html) is now
    exported to facilitate creation of translation reprexes.
  - Invalid tags now generate more informative errors.
  - `\usage{}` now supports qualified functions, eliminating `Unknown call: ::`
    errors.

Again, these are just some of the updates, so please be sure to see the [change
log](https://pkgdown.r-lib.org/news/index.html#pkgdown-1-2-0) for a more
exhaustive inventory.

## Acknowledgements

A big thank you goes out to the 59 people who contributed to this release:
[&#x0040;alexpghayes](https://github.com/alexpghayes), [&#x0040;aqualogy](https://github.com/aqualogy), [&#x0040;aravind-j](https://github.com/aravind-j), [&#x0040;arilamstein](https://github.com/arilamstein), [&#x0040;ArtemSokolov](https://github.com/ArtemSokolov), [&#x0040;BarkleyBG](https://github.com/BarkleyBG), [&#x0040;bastistician](https://github.com/bastistician), [&#x0040;batpigandme](https://github.com/batpigandme), [&#x0040;Bisaloo](https://github.com/Bisaloo), [&#x0040;BruceZhaoR](https://github.com/BruceZhaoR), [&#x0040;cderv](https://github.com/cderv), [&#x0040;daviddoret](https://github.com/daviddoret), [&#x0040;DavisVaughan](https://github.com/DavisVaughan), [&#x0040;dongzhuoer](https://github.com/dongzhuoer), [&#x0040;Dripdrop12](https://github.com/Dripdrop12), [&#x0040;Fazendaaa](https://github.com/Fazendaaa), [&#x0040;GeoBosh](https://github.com/GeoBosh), [&#x0040;goldingn](https://github.com/goldingn), [&#x0040;GregorDeCillia](https://github.com/GregorDeCillia), [&#x0040;hadley](https://github.com/hadley), [&#x0040;HenrikBengtsson](https://github.com/HenrikBengtsson), [&#x0040;HughParsonage](https://github.com/HughParsonage), [&#x0040;IndrajeetPatil](https://github.com/IndrajeetPatil), [&#x0040;jameslamb](https://github.com/jameslamb), [&#x0040;jayhesselberth](https://github.com/jayhesselberth), [&#x0040;jennybc](https://github.com/jennybc), [&#x0040;JiaxiangBU](https://github.com/JiaxiangBU), [&#x0040;jimhester](https://github.com/jimhester), [&#x0040;jmgirard](https://github.com/jmgirard), [&#x0040;JohnMount](https://github.com/JohnMount), [&#x0040;jpzhangvincent](https://github.com/jpzhangvincent), [&#x0040;KasperSkytte](https://github.com/KasperSkytte), [&#x0040;kenahoo](https://github.com/kenahoo), [&#x0040;klmr](https://github.com/klmr), [&#x0040;koheiw](https://github.com/koheiw), [&#x0040;kopperud](https://github.com/kopperud), [&#x0040;krlmlr](https://github.com/krlmlr), [&#x0040;liao961120](https://github.com/liao961120), [&#x0040;lionel-](https://github.com/lionel-), [&#x0040;lrutter](https://github.com/lrutter), [&#x0040;maelle](https://github.com/maelle), [&#x0040;maurolepore](https://github.com/maurolepore), [&#x0040;maxheld83](https://github.com/maxheld83), [&#x0040;md0u80c9](https://github.com/md0u80c9), [&#x0040;mllg](https://github.com/mllg), [&#x0040;mrchypark](https://github.com/mrchypark), [&#x0040;mvinaixa](https://github.com/mvinaixa), [&#x0040;nbenn](https://github.com/nbenn), [&#x0040;pat-s](https://github.com/pat-s), [&#x0040;pbreheny](https://github.com/pbreheny), [&#x0040;peterdesmet](https://github.com/peterdesmet), [&#x0040;petermeissner](https://github.com/petermeissner), [&#x0040;Robinlovelace](https://github.com/Robinlovelace), [&#x0040;strengejacke](https://github.com/strengejacke), [&#x0040;thiloklein](https://github.com/thiloklein), [&#x0040;venelin](https://github.com/venelin), [&#x0040;wenjie2wang](https://github.com/wenjie2wang), [&#x0040;yihui](https://github.com/yihui), and [&#x0040;yonicd](https://github.com/yonicd).
