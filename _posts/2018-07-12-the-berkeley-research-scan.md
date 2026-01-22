---
title: "The Berkeley Research Scan"
modified_date: "2026-01-21"
last_modified_at: "2026-01-21"
redirect_from:
  - "/2018/07/server-logs-explained-part-6"
  - "/2018/07/server-logs-explained-part-6-the-berkeley-research-scan"
---
**Note:** This post has been updated since publication to include more information about UC Berkeley's Internet scanning projects.

---

Some web server log entries are particularly bizarre, like the one we'll be looking at today:
```
169.229.3.91 - - [26/Jun/2016:00:35:26 -0700] "\xD5H\xC5p*\xB7:\x8F\x91\x8A\xE1\xAA\xE0p\xD9\xF2[;\xAE\xE7c\xF7\x9C\xAB~\x98\xCB\xAD\xCB\xBE\xCE\xED\xAF\xEC\x8B\x19\xC6\x08D\xEB\xA8\x91\x1De\x10\x18 u\x01zHj\x00\x8D|\x15\x8B;\x98\x08RaSH" 400 166 "-" "-"
```

My server responded with `400 Bad Request`, but the most interesting part is the giant `$request` portion, which doesn't include any of the normal components you would expect in an HTTP request:
```
\xD5H\xC5p*\xB7:\x8F\x91\x8A\xE1\xAA\xE0p\xD9\xF2[;\xAE\xE7c\xF7\x9C\xAB~\x98\xCB\xAD\xCB\xBE\xCE\xED\xAF\xEC\x8B\x19\xC6\x08D\xEB\xA8\x91\x1De\x10\x18 u\x01zHj\x00\x8D|\x15\x8B;\x98\x08RaSH
```

## What This Means

If your first thought was that this looks like 64 bytes of garbage, then you'd be correct. As it turns out, I wasn't the first person to notice this unusual behavior. According to this [this question on StackExchange](https://security.stackexchange.com/q/115168), the log entry above is the result of an Internet-wide research scan led by the Electrical Engineering and Computer Sciences (EECS) department at [UC Berkeley](https://www.berkeley.edu/).

When I first published this post, a reverse DNS lookup of the client's IP address revealed an informative hostname, `researchscan1.EECS.Berkeley.EDU`. It was one of a few machines related to the project:
```
169.229.3.90 - researchscan0.EECS.Berkeley.EDU
169.229.3.91 - researchscan1.EECS.Berkeley.EDU
169.229.3.92 - researchscan2.EECS.Berkeley.EDU
169.229.3.93 - researchscan3.EECS.Berkeley.EDU
169.229.3.94 - researchscan4.EECS.Berkeley.EDU
```

Accessing any of those IP addresses or hostnames in a web browser revealed a brief description of the project. Those machines are no longer accessible, but archived copies of the descriptions they hosted can be found via the [Internet Archive's Wayback Machine](https://web.archive.org/). There's also more information in [this answer on StackExchange](https://security.stackexchange.com/a/115169), complete with a direct quote from the team behind the research scanning project:
> We are performing a measurement study of a particular phenomenon on the Internet. To accurately assess the behavior we're performing a daily scan of the IPv4 space by sending a single benign packet to every IP on port 80 consisting of 64 random bytes of data. [...] **No, we are not attempting to gain unauthorized access.** [...] **It's simply randomly generated data that conforms to a certain set of criteria.**

## Concluding Thoughts

I contacted the project team myself when I originally published this post, but I never heard anything back. It's possible that the original project had already ended by the time I reached out to the team.

That said, if you want to read more about UC Berkeley's research projects in this space, check out [Prof. Vern Paxson](https://www.icir.org/vern/)'s work and publications and those of his collaborators.

There's also a new iteration of the research scanner that, at the time of this edit, is currently online at [cesrscan.eecs.berkeley.edu](http://cesrscan.eecs.berkeley.edu/).
