---
title: Jekylls and CDNs and GitHub Pages, oh my!
tags: [security, cdn, tls, privacy, github, jekyll, hacker-news]
---
Some months ago, I finally got around to redesigning my blog with [responsiveness](https://en.wikipedia.org/wiki/Responsive_web_design) in mind, along with switching it to run on a new blogging app and hosting platform. Previous to this, I had been using the excellent [Enki blogging app](https://github.com/xaviershay/enki/) (which I still remain quite fond of, assisting with maintaining the project on occasion) and [Heroku](https://www.heroku.com/). This combination resulted in a free hosting situation [with some limitations](https://www.heroku.com/pricing) which I was generally quite happy with. But while Enki is a great little Rails app, especially if you’re the kind of person that likes to be able to blog from any place where you have a web browser and an Internet connection, I felt like I needed a simpler platform. Something where I can worry less about config and maintenance, something that just lets me write my content and stays out of the way. And while Heroku is one of the most developer friendly hosting platforms I have ever used, I was hoping to find something at about the same ongoing price point (say, $0) without the caveats of Heroku’s free plan.<!--more-->

### Jekyll and GitHub Pages

I have been a GitHub user for some time. I do pay for some private repositories that I host there[^1], so I guess one could argue that my use of [GitHub Pages](https://pages.github.com/) as my new hosting platform for [Jekyll](https://jekyllrb.com/) isn’t really free. But I was paying GitHub before I started using GitHub Pages, so I see GitHub Pages as effectively free (and payment is not a requirement for GitHub Pages sites associated with public GitHub projects).

So I worked up a new responsive design based on the Twitter Bootstrap blog template (thanks Bootstrap guys!) and managed to move my content over from Enki and [Textile](https://en.wikipedia.org/wiki/Textile_(markup_language))[^2] to Jekyll and [Markdown](https://en.wikipedia.org/wiki/Markdown). Converting my posts from Textile to Markdown was a little more time consuming than I would have liked, but it’s done now and I’m glad I won’t have to do it again. While there are always going to be certain compromises that need to be made when you move from a database backed blogging app like Enki to a database-less, static site generator like Jekyll, overall I was pretty satisfied with the result.

### Let’s Encrypt

Fast forward to now, and there was one thing that still bothered me. My eyes slowly wondered up to the top of my Chrome browser window and noted a definite lack of a green padlock icon before the address to my blog. We’re living in a time when TLS encryption really should be the default for most any site and I thought my blog should be no exception. To me, paying for [SSL/TLS public key certificates](https://en.wikipedia.org/wiki/Public_key_certificate) has always felt a bit like being subject to the whims of a certificate authority cartel and it wasn’t so long ago that I wouldn’t have had much choice other than to bite the bullet and pay or go without encryption. But fortunately, we now have [Let’s Encrypt](https://letsencrypt.org), a [wonderful initiative from a collection of organisations](https://www.eff.org/deeplinks/2014/11/certificate-authority-encrypt-entire-web) who want to do public key certificates right. With the advent of Let’s Encrypt I can now get a free and legit certificate for my blog (thanks Let’s Encrypt guys!).

### Such connections, many CDNs, very encryption kind of, wow

So the certificate problem was solved, but GitHub pages is not at all the same as having a full-featured web server at my disposal, there were no configuration options I could take advantage of in order to set my certificate for use with a GitHub Pages site. For some time I’d been following the [comically long issue thread on GitHub](https://github.com/isaacs/github/issues/156) requesting that HTTPS support be added to GitHub Pages (my own redundant +1 comment is in there somewhere, sorry GitHub guys!). Obviously I was not the only one with an interest in this matter. But it was from that issue thread that I garnered a number of possible options for adding TLS support to a GitHub Pages site. As far as I can tell, as of the publication of this blog post, all of those options boil down to ‘use a [CDN](https://en.wikipedia.org/wiki/Content_delivery_network)’. It’s just a matter of which one you want to use. One unfortunate thing about putting a CDN in front of your site for the purposes of TLS support is that it’s potentially not end-to-end encryption. You might get an encrypted connection over the first leg of the journey, the connection from the end user to the CDN but the second leg, the connection from the CDN back to your origin server could well be unencrypted. So I could end up with something like:

```
End user

< - Encrypted connection - >

My chosen CDN

< - Unencrypted connection - >

My GitHub Pages site
```

Or so I thought. Can the connection between my chosen CDN and GitHub Pages also support TLS? Well, GitHub kind of supports encrypted connections for GitHub Pages sites when they are reached not via your own custom domain but rather by the standard GitHub Provided address (i.e. ‘*.github.io’). I say ‘kind of’ because it turns out that [GitHub is using their own CDN with GitHub Pages](https://github.com/isaacs/github/issues/156#issuecomment-75738734) and this CDN is adding and removing encryption on their end. So at best I end up with something like:

```
End user

< - Encrypted connection - >

my chosen CDN

< - Encrypted connection - >

GitHub’s chosen CDN

< - Unencrypted connection - >

My Git Hub Pages site
```

This is less than ideal. Encryption should really be end-to-end. But as some encryption is better than none and I’m not planning on moving my blog away from GitHub Pages in the foreseeable future, I’ll take it and make do.

So, which CDN to use? Initially it came down to two possibilities:

* [CloudFlare](https://www.cloudflare.com/)
* [Netlify](https://www.netlify.com/)

Netlify seemed pretty good, except that on their free plan your site gets a ‘Netlify Callout’ appended to it, which looks like [a little tab that pops up at the bottom right of your site’s pages](https://callout.netlify.com/) and I wasn’t too keen on that sort of thing if I could avoid it. Looking at CloudFlare, I found them more congenial. [CloudFlare say they provide SSL support](https://www.cloudflare.com/ssl/), apparently without needing to advertise at the bottom right corner of your site and I really couldn’t think of a reason not to go with them other than perhaps that it seemed like CloudFlare were providing a lot more other services that I currently don’t need for my little blog. Could this add complexity I didn’t need? Or at the very least, knowing me, would my curiosity be piqued and would I be inevitably sucked into playing with CloudFlare’s virtual bells and whistles, ending up with yet another distraction keeping me from what I should really be spending my time on?

Minor reservations relating to inconvenient personality traits aside, I was all set to sign up with CloudFlare when I happened to see a notification email come through from the previously mentioned issue thread-zilla on GitHub from a user named [nubela](https://github.com/isaacs/github/issues/156#issuecomment-193166403). He and a small team in Singapore run a bootstrapped startup named [Kloudsec](https://kloudsec.com/). I looked them up and found [their ‘Show HN’ post on Hacker News](https://news.ycombinator.com/item?id=10899461), wherein they sounded pretty legit, their set up was also free and sounded a little easier than CloudFlare. Kloudsec even had a [special on-boarding process worked out for GitHub Pages users](https://kloudsec.com/github-pages/new). I particularly like to support bootstrappers who are having a go at it, so I thought I’d give Kloudsec a try.

The whole process of setting myself up on Kloudsec’s service would have been entirely frictionless if it wasn’t for my own silliness and non-standard set up. I had trouble with the Kloudsec for GitHub Pages on-boarding process because I thought Kloudsec was asking for my blog’s GitHub Pages site URL (in my case gaelian.github.io) whereas they were actually asking for the corresponding project repository URL (in my case github.com/gaelian/gaelian.github.io). Additionally, my blog’s repository was originally set set to private and Kloudsec’s special GitHub Pages on-boarding process expects that your repository will be public, so after a little email discussion with the very responsive nubela from Kloudsec (seriously, he got back to me within 15 minutes), I decided I’d just take the standard on-boarding process and sign up with Kloudsec that way. The standard on-boarding process (found via any of the ‘Preview Dashboard’ and ‘Try it now’ buttons on the Kloudsec home page) is pretty slick and I was signed up in no time. The only remaining thing for me to do was change my [DNS](https://en.wikipedia.org/wiki/Domain_Name_System) settings to point to Kloudsec. An email was sent to me from Kloudsec that contained all the Kloudsec-related details I needed, two new DNS records would need to be created.

I logged into my account over at my DNS service provider and went about making the changes for my domain. It took longer than I hoped to get things sorted out with my DNS because aside from adding the A and TXT records (the latter being for verification purposes, similar to how Google Apps verifies domain ownership) as instructed by Kloudsec, I neglected to remove the pre-existing CNAME record I had that was pointing to gaelian.github.io and this was interfering with the new A record. I also had my records set with a [TTL](https://en.wikipedia.org/wiki/Time_to_live) of 86400 which wasn’t speeding things along. But once this was all sorted, everything fell into place. I could then log into my Kloudsec account and provision my new Let’s Encrypt certificate for use with my site[^3]. Kloudsec can also redirect from ‘http’ to ‘https’ for you, which is a nice little touch that should not be forgotten.

Kloudsec’s management interface is fairly straight forward to use, quite pretty, trendy even, being implemented as a [single-page application](https://en.wikipedia.org/wiki/Single-page_application) (single-page apps are *so in* right now). I’m still exploring Kloudsec’s other features[^4] but so far I appreciate that these features are also implemented in a straight forward kind of way that suits my current level of interest. Perhaps power users might find the available ways to slice and visualise your data to be a little lacking but this is not a concern for me.

I was told by nubela that a couple of new features are coming down the Kloudsec pipeline in the near future. The first being CNAME/ALIAS support, so that you can enter a domain name (rather than only an IP address, as is the case now) into Kloudsec’s management interface to identify your origin server. I could see this being useful for example in a case like mine, where my origin server is managed by GitHub not me, and the GitHub Pages IP address(es) may potentially change without my knowledge. Kloudsec is also looking at automatically switching to an encrypted connection between their CDN and the origin server if the origin server supports encryption. This would be awesome for my situation if - as mentioned earlier - GitHub wasn’t also employing their own CDN in front of GitHub Pages sites that does not support encryption. But hey, maybe one day…

In other news, I’ve noticed that since switching to ‘https’, the Facebook ‘like’ counts on my site have reset to zero. Same for the Google+ widget, but I’m not actually sure if I ever had any Google +1’s for any of my posts in the first place (sorry Google guys!). I therefore assume that these social media widgets treat the same URLs save for ’http’ vs. ‘https’ as different locations, which is understandable I suppose but none the less unanticipated. I tried to think of some pithy, yet profound quote about entropy and the impermanence of all things to put here, but I’m coming up empty.

### Conclusion

So that’s the long and the short of it. If you’re looking at implementing a CDN for your site or particularly how you can *kind of* get TLS sorted out for your GitHub Pages site, I hope you’ve found this post useful.

**Disclaimer:** I have not received any remuneration from any of the services I’ve mentioned in this post, I’m just a normal customer of some of these services, who might like the cut of their respective jibs.

[^1]: I hear [BitBucket](https://bitbucket.org) gives private Git repos for free, as does [Visual Studio Team Services](https://www.visualstudio.com/en-us/products/visual-studio-team-services-vs.aspx), so if you’re not as embedded in GitHub as I am, you may like to check them out.

[^2]: Yeah, I started my blog back when Textile was still the new hotness.

[^3]: Let’s Encrypt allows automated certificate provisioning which is why Kloudsec can provision a certificate for you on demand. but Let’s Encrypt do rate limit clients, so this process can potentially take a little while.

[^4]: Including ‘Offline Protection’ and ‘Page Optimizer’ via CDN caching, as well as an experimental [web application firewall](https://www.owasp.org/index.php/Web_Application_Firewall) that sounds interesting, although as my blog is static HTML by the time it gets served up, this feature doesn’t seem too relevant for this particular site (feel like [SQLi-ing](https://en.wikipedia.org/wiki/SQL_injection) my non-existent database? Or maybe [brute forcing](https://en.wikipedia.org/wiki/Brute-force_attack) my non-existent admin interface? Knock yourself out, buddy).
