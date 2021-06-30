# About techbits.io

techbits.io was started 8 years ago, and has been through many revisions. As a pandemic-project, the site has been rebuilt from scratch with a focus on privacy and performance.

## Privacy

techbits.io doesn't use any cookies. Not tracking cookies, not session cookies, nothing. Building a site from the ground up with this as an aim forces you to make subconscious decisions as you progress, as opposed to trying to rip cookie-reliance out of an existing system. For example, a cookie could be used to track light-mode/dark-mode preferences, but as that simply isn't an option, a pure CSS approach was taken with the preference being set by the visitor's operating system.

## Performance

The average web page is around 2MB, and while this doesn't take long to load on fast(er) home internet, when using spotty 3G mobile networks, 2MB can take **forever** to load. Every element of this site has been considered in terms of performance, and because it's a static site which is periodically rebuilt, resources can be controlled in a way that's not possible on a dynamic website/CMS (for example, the CSS stylesheets are generated on each build and only include the selectors which the site uses).

## Comments and Edits

A commenting system and a cookie-less, static site don't really go hand-in-hand. Equally, enabling anyone to submit corrections/expansions of points would usually require some kind of CMS. 

By using the Github API, PRs and issue text can be used when generating pages allowing the site to remain static, but appear dynamic.

## So what is this repo?
This is the content repo for https://techbits.io. The files here are used to generate the website. PRs will drive change/corrections on the website, and issues represent comment threads on posts.

## Is there a license?
There's no license. Like most websites, all of the content in this repo is copyrighted unless explicitly stated, and all rights are reserved by techbitsio. Unaltered quotes can be reproduced under fair usage/fair dealing laws, but please link back to original content. If you're not sure, please DM https://twitter.com/techbitsio

Before submitting content to this repo, please ensure you're happy with these copyright rules, and that what you're adding is your own work which you're happy to relinquish the copyright for. Contributing authors will be credited (merged PRs will show on the affected posts/pages, with a link back to your PR).

For the sake of clarity, forking the repo is permitted for the exclusive purpose of submitting a PR.

If you're not sure about copyright terms, or anything else, please ask!