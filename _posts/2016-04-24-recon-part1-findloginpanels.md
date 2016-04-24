---
layout: post
title:  "Recon Part 1 - Finding Login Panels"
date:   2016-04-24 22:07:10 
categories: Reconaissance
author: pry0cc
---

In this post, I will uncover the mysteries torward finding hidden login panels and portals, something critical to finding low hanging fruit, uncovering larger vulnerabilities.

## Reconassiance
In this multipart series, I will be demonstrating and explaining the basics of reconaissance, something that is critical to finding and successfully exploiting boxes. Instead of covering purely technical aspects, I will be covering practical aspects. So rather than focussing on just DNS recon for this part, I will be looking at finding Admin Login portals and panels for sites, something that uses multiple skills to execute successful in most engagements.

### Finding hidden subdomains with dnsenum.
The first thing I check a domain for is hidden subdomains, a lot of people don't hide their subdomains very well, and with a quick run of dnsenum you have access to panel for their login. You can get dnsnum here at ![Github](https://github.com/fwaeytens/dnsenum)

<code> ./dnsenum.pl target.com </code>

And it will return whatever subdomains, and mail servers it can find, you can also do a bruteforce using a wordlist. I find it generally returns something decent, and gives me another lead.

### Finding hidden login panels with AdminFinder
The next tool I'll run is AdminFinder, it will essentially run through a list of known rules for admin and login portals, and will try to retrieve them, if the server reponds with a 404, it will assume that it doesn't exist, but if it returns a 200 it will alert you and assume it works. While the script is simplistic, it can save you a lot of time. You can get AdminFinder here at ![Github](https://github.com/indranilbanerjee/AdminFinder)

### Finding hidden login panels with robots.txt
A lot of sites put their secretive directories in example.com/robots.txt to stop search engines like google picking them up and indexing them, so if you just put /robots.txt on the end of a site, you'll generally be surprised at how much result you get.


### Theoretical - Finding hidden login panels through an XSS Hook.
In theory, if you can find a stored site wide XSS vuln, you can run hook their browser with BeEF and watch as they navigate directories, this could lead you to gold.

### Golden bullet - Finding hidden login panels with a baseball bat. 
If worse comes to worse, people will probably give out the details to the whereabouts to their login panels through a social engineering engagement, but of course, you will need to be much more creative than the rest of these methods. 

So using this, you've hopefully found your login panel, and can now decide whether to launch a social engineering campaign using SET or manually, or a bruteforce attack with THC-Hydra.

Stay tuned 0x00er's for the next part of my reconaissance series!
pry0cc
