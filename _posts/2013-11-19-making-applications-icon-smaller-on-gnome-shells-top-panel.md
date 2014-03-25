---
layout: post
title: Making application's icon smaller on GNOME Shell's top panel
image:
   url: /media/2013-11-19-making-applications-icon-smaller-on-gnome-shells-top-panel/cover.jpg
   source: https://gitorious.org/gnome-design/gnome-design/source/50d3b20057f797d8ba37579a9baf9792a92aa0a7:wallpaper/3.12
quote: I'm a huge fan of GNOME Shell. Using it since the release of version 3.0, is great to see how things has been polished.
comments: true
---

I'm a huge fan of GNOME Shell. Using it since the release of version 3.0, is great to see how things has been polished. However, one of the things that always annoyed me is the app icon on the top bar, because it's unnecessarily big, in my opinion.

A long time ago I found [this good tutorial](http://www.gregfreeman.org/2011/remove-application-icon-from-gnome-3/) showing how to remove the icon, but it was not what I wanted, I just wanted to make it smaller, like this:

{% include image.html url="https://github.com/gnome-design-team/gnome-mockups/raw/master/music/music-albums.png"  description="GNOME Music's mockup showing a smaller icon on the top panel, via GNOME Wiki" %}

So, after a lot of unsuccessful tentatives on changing *gnome-shell.css*, I realized that this setup is made with *javascript*, and not just *css*, and I decided to write this small tutorial.

<div class="message"><strong>Disclaimer</strong>: the following procedures were made on version 3.10 of gnome-shell. I don't guarantee that it'll work in previous versions, neither in later versions, nor even in version 3.10, and I'm not responsible for possible damage to your system.</div>

The files you'll have to change are:

1. **/usr/share/gnome-shell/js/ui/panel.js**: to change the size of the icon.
2. **/usr/share/gnome-shell/theme/gnome-shell.css** (or, if you're using a custom theme, its respective *css*): to move the icon slightly to the left.

On *panel.js*, you will need to apply the following patch:

{% highlight diff %}
295c295
<         let icon = this._targetApp.get_faded_icon(2 * PANEL_ICON_SIZE, this._iconBox.text_direction);
---
>         let icon = this._targetApp.get_faded_icon(PANEL_ICON_SIZE, this._iconBox.text_direction);
350,351c350,351
<         alloc.min_size = minSize;
<         alloc.natural_size = naturalSize;
---
>         alloc.min_size = minSize + Math.floor(PANEL_ICON_SIZE * 2);
>         alloc.natural_size = naturalSize + Math.floor(PANEL_ICON_SIZE * 2) ;
397,398c397,398
<             childBox.x1 = Math.floor(iconWidth / 2);
<             childBox.x2 = Math.min(childBox.x1 + naturalWidth, allocWidth);
---
>             childBox.x1 = Math.floor(iconWidth + 3);
>             childBox.x2 = Math.floor(iconWidth + 3) + Math.min(childBox.x1 + naturalWidth, allocWidth);
400c400
<             childBox.x2 = allocWidth - Math.floor(iconWidth / 2);
---
>             childBox.x2 = allocWidth - Math.floor(iconWidth + 3);
{% endhighlight %}

You can change these lines manually, or download the diff [here](https://gist.github.com/camporez/8566608/download), and apply it with the following command:
`sudo patch /usr/share/gnome-shell/js/ui/panel.js panel.js.diff`

The changes on the css are small so you just need to open it with your favorite editor and add the line `padding-left: 10px;` to the selector `#appMenu`, like this:

{% highlight css %}
#appMenu {
    spinner-image: url("process-working.svg");
    spacing: 4px;
    padding-left: 10px;
}
{% endhighlight %}

Now, you just need to restart gnome-shell (press ALT + F2 and type r + ENTER) and it's done.

This is how it (plus another tweaks on gnome-shell.css) made my desktop looks like:
![My desktop today](/media/2013-11-19-making-applications-icon-smaller-on-gnome-shells-top-panel/Captura_de_tela_de_2013_11_19_13_12_42.png)
If you have any suggestion or feedback, please contact me [via Google+](http://google.com/+IanCamporezBrunelli). Thank you!
