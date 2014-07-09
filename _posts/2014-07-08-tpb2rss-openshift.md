---
layout: post
title: Using TPB2RSS with OpenShift
quote: "A small tutorial on how to generate feed based on thepiratebay.se searches using TPB2RSS (a.k.a. my new python project)"
image:
    url: /media/2014-07-08-tpb2rss-openshift/cover.jpg
video: false
---

Hello, everybody! Nice to see you here...

Well, you may not know, but I was working on a "The Pirate Bay search" to "RSS feed" converter on the last couple of days.

It is a really simple script that, given a search term or a local saved search page, returns (by default printing on the screen) a xml containing the feed (following [the specifications of RSS 2.0](cyber.law.harvard.edu/rss/rss.html)).

I won't give more details of the functioning or the functionality of this script in here. You can [take a look at the source code](https://github.com/camporez/tpb2rss/blob/master/tpb2rss.py) and ask any question [in the issues page](https://github.com/camporez/tpb2rss/issues), [twitter](http://twitter.com/iancamporez)/[Google+](http://google.com/+IanCamporezBrunelli) or on the comments' section below.

Now, let's get started with the how-to...

# Installing on OpenShift

## Creating the application

Create a Python application on OpenShift, then add the Cron cartridge. Clone the application to your computer.

Now, you'll edit the file `setup.py` to include [the Beautiful Soup 4 library](http://www.crummy.com/software/BeautifulSoup/) on your installation. Uncomment the line `install_requires` and add `beautifulsoup4` to it. The line will look like this: `install_requires=['Django>=1.3','beautifulsoup4'],`.

After edit this file, it's time to place TPB2RSS' files. Browse to your repository's root and run the following commands:

{% highlight bash %}
$ git clone https://github.com/camporez/tpb2rss.git tpb2rss
$ mv tpb2rss/tpb2rss.py .
$ mv tpb2rss/openshift/tpb2rss-task.sh .openshift/cron/minutely/
$ mv tpb2rss/openshift/tpb2rss-cleaner.sh .openshift/cron/daily/
$ mkdir -p wsgi/static; touch wsgi/static/example.xml
$ rm -rf tpb2rss
{% endhighlight %}

Commit and push the changes. I won't describe how to use Git/OpenShift, as good tutorials can be easily found.

## Finishing the installation

Now that you application is up, running and with the required software installed, SSH to it.

Give permissions and create the required files and links:

{% highlight bash %}
$ chmod +x "$OPENSHIFT_REPO_DIR/tpb2rss.py"
$ chmod +x "$OPENSHIFT_REPO_DIR/.openshift/cron/minutely/tpb2rss-task.sh"
$ chmod +x "$OPENSHIFT_REPO_DIR/.openshift/cron/daily/tpb2rss-task.sh"
$ ln -s "$OPENSHIFT_REPO_DIR/wsgi/static" "$OPENSHIFT_DATA_DIR/static"
$ ln -s "$OPENSHIFT_REPO_DIR/.openshift/cron" "$OPENSHIFT_DATA_DIR/cron"
$ touch "$OPENSHIFT_DATA_DIR/searches"
{% endhighlight %}

## Managing the feeds

TPB2RSS is probably running, if you followed all the steps correctly.

You can create new feeds by editing the file `$OPENSHIFT_DATA_DIR/searches`. SSH to your application and edit this file with your favorite text editor.

Each line of this file represents a search term that will generate a XML file in `$OPENSHIFT_DATA_DIR/static`.

This file is available online by the URL `YOUR_OPENSHIFT_URL/static/SEARCH_STRING.xml`, and will look like this:

{% highlight xml %}
<rss version="2.0">
	<channel>
		<title>TPB2RSS: Under The Dome S02 !720p [eztv]</title>
		<link>http://thepiratebay.org/search/Under%20The%20Dome%20S02%20!720p%20[eztv]/0/3/0</link>
		<description>Search feed for "Under The Dome S02 !720p [eztv]"</description>
		<lastBuildDate>Wed, 09 Jul 2014 01:22:20 GMT</lastBuildDate>
		<language>en-us</language>
		<generator>TPB2RSS 1.0</generator>
		<docs>http://github.com/camporez/tpb2rss</docs>
		<webMaster>ian@camporez.com</webMaster>
			<item>
				<title>Under the Dome S02E02 HDTV x264-LOL [eztv]</title>
				<link><![CDATA[ magnet:?xt=urn:btih:9759f086c714589f9d75ad04800cf99ce2bd9b19&amp;dn=Under+the+Dome+S02E02+HDTV+x264-LOL+%5Beztv%5D&amp;tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80&amp;tr=udp%3A%2F%2Ftracker.publicbt.com%3A80&amp;tr=udp%3A%2F%2Ftracker.istole.it%3A6969&amp;tr=udp%3A%2F%2Fopen.demonii.com%3A1337 ]]></link>
				<pubDate>Tue, 08 Jul 2014 07:24:00 GMT</pubDate>
				<description>Link: https://thepiratebay.org/torrent/10507825/Under_the_Dome_S02E02_HDTV_x264-LOL_[eztv]
Uploader: eztv
Size: 269.91 MiB
Seeders: 7546
Leechers: 1339</description>
			</item>
			<item>
				<title>Under the Dome S02E01 HDTV x264-LOL [eztv]</title>
				<link><![CDATA[ magnet:?xt=urn:btih:97b265826f18f6183d12257d26d7948092c43bb0&amp;dn=Under+the+Dome+S02E01+HDTV+x264-LOL+%5Beztv%5D&amp;tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80&amp;tr=udp%3A%2F%2Ftracker.publicbt.com%3A80&amp;tr=udp%3A%2F%2Ftracker.istole.it%3A6969&amp;tr=udp%3A%2F%2Fopen.demonii.com%3A1337 ]]></link>
				<pubDate>Tue, 01 Jul 2014 04:34:00 GMT</pubDate>
				<description>Link: https://thepiratebay.org/torrent/10465762/Under_the_Dome_S02E01_HDTV_x264-LOL_[eztv]
Uploader: eztv
Size: 376.38 MiB
Seeders: 7443
Leechers: 684</description>
			</item>
	</channel>
</rss>
{% endhighlight %}

You can add this URL to your RSS client, torrent client (some of them support RSS), IFTTT (to receive a notification when a new item arrives, for example).

This is it! If you've found any problem with this tutorial, please comment below.
