---
published: false
---
## Reconassiance
In this multipart series, I will be demonstrating and explaining the basics of reconaissance, something that is critical to finding and successfully exploiting boxes. Instead of covering purely technical aspects, I will be covering practical aspects. So rather than focussing on just DNS recon for this part, I will be looking at finding Admin Login portals and panels for sites, something that uses multiple skills to execute successful in most engagements.

### Finding hidden subdomains with dnsenum.
The first thing I check a domain for is hidden subdomains, a lot of people don't hide their subdomains very well, and with a quick run of dnsenum you have access to panel for their login. You can get dnsnum here at ![Github](https://github.com/fwaeytens/dnsenum)

<code> ./dnsenum.pl target.com </code>

And it will return whatever subdomains, and mail servers it can find, you can also do a bruteforce using a wordlist.


