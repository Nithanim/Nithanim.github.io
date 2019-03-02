---
layout: posts
title:  "Capture twitch livestreams"
date:   1970-01-01 00:00:00 +0100
author: Nithanim
categories: misc
---

## On desktop (with GUI)

It is easy to capture a livestream on twitch with the [program called livestreamer](https://github.com/chrippa/livestreamer/releases) and the [VLC media player](http://www.videolan.org/vlc/). I found a [quick and decent tutorial on how to capture it on reddit](http://www.reddit.com/r/starcitizen/comments/2dkv5i/psa_watchrecord_twitch_livestream_using_vlc/).

Note though that you may want to restart VLC after setting your destination path because it will save the stream to the old directory (at least it happened for me).
Additionally, you really want to make sure that you add "/" or "\" (depending on the OS) to the destination if you want VLC to save the stream into a directory! Otherwise (if the directory does not exist (anymore)), it will save all the livestreams in one file and therefore will be lost.

## On headless

Sometimes you want to capture a stream but you do not want to let your computer running. If you have some kind of server laying around, you can also use it to capture the stream for you.

At first you need to install livestreamer and VLC (for me the debian package was appropriate) after that. The trickiest part on this is to get VLC to cope with the fact that the server has no GUI and to tell it to save the stream. We are lucky to be able to pass VLC additional arguments through livestreamer. As only VLC is concened it is as easy as:
```
--sout file/ts:/home/<user>/streams/stream.ts --intf dummy
```
This can be passed to VLC via the `--player` argument. With the tutorial on reddit livestreamer is handling the stream and gives it to VLC constantly. This did not work out for me, I fixed it with using `--player-passthrough`.

When me merge everything together we get a simple command ready to save into a .sh file:
```
livestreamer --player-passthrough hls --player "vlc --sout file/ts:/home/<user>/streams/stream.ts --intf dummy" twitch.tv/<twitchuser> source
```
If you also like to have a date in the filename and not accidentally overwrite the previous stream you can use the following:
```
TIMESTAMP=`date +%Y-%m-%d_%H-%M-%S`
livestreamer --player-passthrough hls --player "vlc --sout file/ts:/home/<user>/streams/${TIMESTAMP}.ts --intf dummy" twitch.tv/<twitchuser> source
```
Do not forget to make your .sh executable with `chmod +x <yourShFile>`. You can run it with `./<yourShFile>`.
