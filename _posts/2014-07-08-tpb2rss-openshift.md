---
layout: post
title: Using TPB2RSS with OpenShift
quote: "A small tutorial on how to generate feed based on thepiratebay.se searches using TPB2RSS (a.k.a. my new python project)"
image:
    url: /media/2014-07-08-tpb2rss-openshift/cover.jpg
    source: http://mitahav.deviantart.com/art/pirate-bay-in-code-245603343
video: false
---

Hello, everybody! Nice to see you here...

Well, you may not know, but I was working on a "The Pirate Bay search" to "RSS feed" converter on the last couple of days.

It is a really simple script that, given a search term, a TPB link or a local saved TPB page, returns (by default printing on the screen) a xml containing the feed (following [the specifications of RSS 2.0](http://cyber.law.harvard.edu/rss/rss.html)).

I won't give more details of the functioning or the functionality of this script in here. You can [take a look at the source code](https://github.com/camporez/tpb2rss/blob/master/tpb2rss.py) and ask any question [in the issues page](https://github.com/camporez/tpb2rss/issues), [Twitter](http://twitter.com/iancamporez)/[Google+](http://google.com/+IanCamporezBrunelli) or in the comments' section below.

Before I start the tutorial, you've to know that there're inumerous ways to create the feed generator on OpenShift, but in this tutorial I'll approach two of them:

- The first one, runs a cron task to update the RSS. Any RSS is created based on the content of a file placed inside your OpenShift installation. Therefore, you'll have to SSH to your application and edit this file to add/remove new feeds.
- The second one is a web-based service. To create a new feed you just need to open your application main page and insert the link in there, or open a search/user/browse page in thepirate bay and replace the main url to YOUR_OPENSHIFT_URL.

From now on, I will call the first method "**private**" and the second "**public**", so a command indicating "public method only" has to be executed only if you're following the second method, and vice-versa.

Let's get started with the tutorial...

# Installing on OpenShift

## Creating the application

Create a [Python 2.7 application on OpenShift](https://openshift.redhat.com/app/console/application_type/cart!python-2.7), then, if you're following the **private** method, add the Cron cartridge. Clone the application to your computer.

Now, you'll edit the file `setup.py` to include [the Beautiful Soup 4 library](http://www.crummy.com/software/BeautifulSoup/) on your installation. Uncomment the line `install_requires` and add `beautifulsoup4` to it. The line will look like this:

{% highlight python %}
      install_requires=['beautifulsoup4'],
{% endhighlight %}

After edit this file, it's time to place TPB2RSS' files. Browse to your repository's root and run the following commands:

{% highlight bash %}
$ git clone https://github.com/camporez/tpb2rss.git tpb2rss
$ mv tpb2rss/tpb2rss.py .
$ mv tpb2rss/openshift/*.py . # Public method only
$ mv tpb2rss/openshift/cron/ .openshift/ # Private method only
$ mkdir -p wsgi/static; touch wsgi/static/example.xml # Private method only
$ rm -rf tpb2rss
{% endhighlight %}

Commit and push the changes. I won't describe how to use Git/OpenShift, as good tutorials can be easily found.

## Finishing the installation

If you're following the "**private**" method, SSH to your application and create this file, directory and link:

{% highlight bash %}
$ touch "$OPENSHIFT_DATA_DIR/searches"
$ mkdir "$OPENSHIFT_DATA_DIR/raw"
$ ln -s "$OPENSHIFT_REPO_DIR/wsgi/static" "$OPENSHIFT_DATA_DIR/static"
{% endhighlight %}

## Managing the feeds

TPB2RSS is probably running, if you followed all the steps correctly.

If you're following the "**private**" method, you can create new feeds by editing the file `$OPENSHIFT_DATA_DIR/searches`. SSH to your application and edit this file with your favorite text editor.

Each line of this file represents a search term that will generate a XML file in `$OPENSHIFT_DATA_DIR/static`.

This file is available online by the URL `http://YOUR_OPENSHIFT_URL/static/SEARCH_STRING.xml`, and will look like this:

{% highlight xml %}
<rss version="2.0">
	<channel>
		<title>TPB2RSS: Under The Dome S02 !720p [eztv]</title>
		<link>http://thepiratebay.org/search/Under%20The%20Dome%20S02%20!720p%20[eztv]/0/3/0</link>
		<description>The Pirate Bay search feed for "Under The Dome S02 !720p [eztv]"</description>
		<lastBuildDate>Wed, 09 Jul 2014 01:22:20 GMT</lastBuildDate>
		<language>en-us</language>
		<generator>TPB2RSS 1.0</generator>
		<docs>http://github.com/camporez/tpb2rss</docs>
		<webMaster>ian@camporez.com</webMaster>
			<item>
				<title>Under the Dome S02E02 HDTV x264-LOL [eztv]</title>
				<link><![CDATA[ magnet:?xt=urn:btih:9759f086c714589f9d75ad04800cf99ce2bd9b19&amp;dn=Under+the+Dome+S02E02+HDTV+x264-LOL+%5Beztv%5D&amp;tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80&amp;tr=udp%3A%2F%2Ftracker.publicbt.com%3A80&amp;tr=udp%3A%2F%2Ftracker.istole.it%3A6969&amp;tr=udp%3A%2F%2Fopen.demonii.com%3A1337 ]]></link>
				<pubDate>Tue, 08 Jul 2014 07:24:00 GMT</pubDate>
				<description>![CDATA[ Link: https://thepiratebay.org/torrent/10507825/Under_the_Dome_S02E02_HDTV_x264-LOL_[eztv]<br>Uploader: eztv<br>Size: 269.91 MiB<br>Seeders: 7546<br>Leechers: 1339 ]]></description>
			</item>
			<item>
				<title>Under the Dome S02E01 HDTV x264-LOL [eztv]</title>
				<link><![CDATA[ magnet:?xt=urn:btih:97b265826f18f6183d12257d26d7948092c43bb0&amp;dn=Under+the+Dome+S02E01+HDTV+x264-LOL+%5Beztv%5D&amp;tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80&amp;tr=udp%3A%2F%2Ftracker.publicbt.com%3A80&amp;tr=udp%3A%2F%2Ftracker.istole.it%3A6969&amp;tr=udp%3A%2F%2Fopen.demonii.com%3A1337 ]]></link>
				<pubDate>Tue, 01 Jul 2014 04:34:00 GMT</pubDate>
				<description>![CDATA[ Link: https://thepiratebay.org/torrent/10465762/Under_the_Dome_S02E01_HDTV_x264-LOL_[eztv]<br>Uploader: eztv<br>Size: 376.38 MiB<br>Seeders: 7443<br>Leechers: 684 ]]></description>
			</item>
	</channel>
</rss>
{% endhighlight %}

If you're following the "**public**" method, just browse to `http://YOUR_OPENSHIFT_URL` to generate a new feed URL. The content of this URL will look like the above XML.

You can add this URL to your feed client, torrent client (µTorrent, for example, [supports RSS](http://www.utorrent.com/intl/en/help/guides/rss)), [IFTTT](https://ifttt.com) (to receive a notification when a new item arrives, like in the screenshot below) or any other RSS-tool.

{% include image.html url="/media/2014-07-08-tpb2rss-openshift/Captura de tela de 2014-07-09 09:15:56.png" width="100%" description="If a new Halt and Catch Fire episode is uploaded (in WEB-DL and 720p), then send me a note using Pushbullet." %}

This is it! If you've found any issue with this tutorial, please comment below.
