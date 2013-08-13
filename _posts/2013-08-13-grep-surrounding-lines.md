---
layout: post
title: Grep surrounding lines
---

Today I was scanning the logs of one our node apps because a client was having issues. I wanted to find all of the lines before any line with the string '500 -' because this is where the exceptions actually start spitting out their stack trace. Turns out this is really easy with grep.

The trick is with these flags. 

```bash
Context control:
  -B, --before-context=NUM  print NUM lines of leading context
  -A, --after-context=NUM   print NUM lines of trailing context
  -C, --context=NUM         print NUM lines of output context
```

References:
* http://stackoverflow.com/questions/9081/grep-a-file-but-show-several-surrounding-lines