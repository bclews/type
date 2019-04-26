---
layout: post
title: Extracting audio from YouTube videos
tags: [Command Line, YouTube]
---

I recently wanted to extract the audio tracks from some YouTube videos. The [youtube-dl](https://ytdl-org.github.io/youtube-dl/) command-line application was the perfect tool for the job.

[Installation](https://github.com/ytdl-org/youtube-dl#installation) on macOS was straightforward (at the time of writing, this installed version 2019.03.18):
{% highlight bash %}
brew install youtube-dl
{% endhighlight %}

The audio from a YouTube video is extracted by using the _extract audio_ option:
{% highlight bash %}
youtube-dl --extract-audio <video_url>
{% endhighlight %}

The default audio format is Ogg, however you can specify alternative formats by using the _audio format_ option:
{% highlight bash %}
youtube-dl --extract-audio --audio-format mp3 <video_url>
{% endhighlight %}

The [audio format options](https://github.com/ytdl-org/youtube-dl#options) are aac, flac, mp3, m4a, opus, vorbis, or wav.