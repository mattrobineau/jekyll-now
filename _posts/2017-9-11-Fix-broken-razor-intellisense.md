---
layout: post
title: Fixing razor intellisense
---

I recently found myself unable to use the intellisense for razor pages in Visual Studio Community edition. The fix is relatively simple albeit not obvious.
Simply go to _Tools_ -> _Options_ -> _Environment_ -> _General_ and uncheck _Automatically adjust visual experience base on client performance_ as well as both _Enable rich client visual experience_ and _Use hardware graphics acceleration if available_.

![Visual Studio 2017 Community Edition screenshot of general environment options]({{ site.url }}/images/posts/2017-9-11/vs2017_config_screenshot.png)

The only other workaround is to close the file without intellisense and reopen it.