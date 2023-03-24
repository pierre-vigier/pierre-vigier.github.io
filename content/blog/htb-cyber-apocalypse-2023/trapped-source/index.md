---
title: "[web] Trapped source"
date: 2023-03-24T13:40:27+08:00
summary: HTB Cyber apocalypse 2023 - Inspect, inspect, inspect
draft: false
tags: [ "ctf", "web" ]
---
In this challenge, we can start a docker image, but no source is provided, the website that is shown represent a combination lock.

![Locked](img/locked.png)

As we do not have much to start from, let's look at the source code of the page, which happen to contains the following:

```html
<body>
	<script>
		window.CONFIG = window.CONFIG || {
			buildNumber: "v20190816",
			debug: false,
			modelName: "Valencia",
			correctPin: "8291",
		}
	</script>
```

Using the given PIN, the lock can be open

![Unlocked](img/unlocked.png)
