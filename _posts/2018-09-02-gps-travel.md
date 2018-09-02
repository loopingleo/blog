---
layout: post
title: Tracking your travel adventures using GPS data from your phone and Python/folium
comments: True
---
Having a visual GPS log of your trips to foreign places can be very beneficial for telling your friends and family a cool story about your adventures. I recently started tracking my location and other data such as accelerometer with the iOS app ['SensorLog'](https://itunes.apple.com/us/app/sensorlog/id388014573?mt=8) to visualise it using Python/R scripts. In this blog post I will briefly guide you through setting up the GPS tracking on your phone and how you can create beautiful maps with your data.



GPS data logging with your smartphone
---

[SensorLog](https://itunes.apple.com/us/app/sensorlog/id388014573?mt=8) is a great app for iOS devices to record all your phone's sensor data. You can simply let the app run in the background -- whether your hiking in New Zealand, enjoying Renaissance paintings in Rome or sipping Moscow Mules on a cruise ship.

<!--
Since there are many articles online about creating GitHub Pages, I will only write down simplified steps creating a personal page:

1. Create a folder locally and initialize `git` repository in that folder by `git init`.
2. Create a simple `index.html` and add it to the repo by `git add index.html`.
3. Commit you changes by doing `git commit -m 'init'`.
4. Create a remote repository on Github as described [here](https://help.github.com/articles/create-a-repo/). Name it `ANYNAME.github.io`.
5. If needed setup `ssh` keys as described [here](https://help.github.com/articles/generating-ssh-keys/).
6. Add remote repository to your local repository by `git add remote REMOTE_URL`. More detailed steps are can be found [here](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/).
-->

### Transferring GPS logging data to your computer

Once you recorded your sensor data, you can transfer it from your phone to your computer. Personally, I save these files using iCloud.

<!--
Here are some good websites :

* [Html5up](https://html5up.net)
* [WIX](http://www.wix.com/website/templates/html/personal/1)
* [FreeWebTemplates](http://www.freewebtemplates.com/personal/)
-->

### Reading in the data in Python

Reading in your sensor data into Python is easy. My choice for this use case is pandas and locally stored files since it's relatively simple and it works great. You can read single files. But in my case I had been tracking my location for many days and had multiple files ready to be visualised in my iCloud. Let's have a look how this works:

``` python
import os, glob
import pandas as pd

# read multiple files with SensorLog data
# adjust to your directory - I use iCloud directory on my Mac
path = os.path.expanduser("~/Library/Mobile Documents/com~apple~CloudDocs/SensorLogData")  
allFiles = glob.glob(path + "/*.csv")
frame = pd.DataFrame()
list_ = []

# read all files in directory
for file_ in allFiles:
  df = pd.read_csv(file_, index_col=None, header=0)
  list_.append(df)

# concat all files into one DataFrame
df_sensorData = pd.concat(list_, sort=True)
```

Once you've read in the file, you can have a brief look at the data:


``` python
df_sensorData.info()
with pd.option_context('display.max_rows', 1, 'display.max_columns', None):
    print(df_sensorData)
```

Or you could do some simple plotting with matplotlib.pyplot.


Once you have had a first look at the data, you can decide which variables are most important to you. In this example, I selected the following columns from my pandas DataFrame and shortened the column names for easier handling:

``` python
# select relevant columns
df_compressed = df_sensorData[["loggingTime(txt)",
                               "locationTimestamp_since1970(s)",
                               "loggingSample(N)",
                               "locationLatitude(WGS84)",
                               "locationLongitude(WGS84)",
                               "locationAltitude(m)",
                               "locationSpeed(m/s)",
                               "accelerometerTimestamp_sinceReboot(s)",
                               "accelerometerAccelerationX(G)",
                               "accelerometerAccelerationY(G)",
                               "accelerometerAccelerationZ(G)",
                               "gyroRotationX(rad/s)",
                               "gyroRotationY(rad/s)",
                               "gyroRotationZ(rad/s)"]]

# shorten names of columns
df_compressed = df_compressed.rename(index=str, columns={"loggingTime(txt)":"time",
                               "locationTimestamp_since1970(s)":"time_s1970",
                               "loggingSample(N)":"sample_ind",
                               "locationLatitude(WGS84)":"lat",
                               "locationLongitude(WGS84)":"lon",
                               "locationAltitude(m)":"altitude",
                               "locationSpeed(m/s)":"speed",
                               "accelerometerTimestamp_sinceReboot(s)":"accel_time",
                               "accelerometerAccelerationX(G)":"x",
                               "accelerometerAccelerationY(G)":"y",
                               "accelerometerAccelerationZ(G)":"z",
                               "gyroRotationX(rad/s)":"Xgyro",
                               "gyroRotationY(rad/s)":"Ygyro",
                               "gyroRotationZ(rad/s)":"Zgyro"})
```

There you go. Now your data is ready to be put on a map.



Plotting GPS data on a map using folium
---

Recently I learned about [Jekyll](http://jekyllrb.com/) - static websites generator and for me it seemed like a great and simpler alternative to dynamic platform like Wordpress, Joomla, etc. Besides its simplicity, it makes backups so much easier and avoids most common security concerns caused by running dynamic websites. Jekyll allows to write posts in [Markdown](https://en.wikipedia.org/wiki/Markdown) which is another big plus. Moreover, code examples are very nicely embedded in the website. So combining with a fact that GitHub provides free hosting for Jekyll blogs, I was completely sold for it.

### Poole

Even though setting up Jekyll is relatively easy, there exists a really nice repository - [poole](https://github.com/poole/poole) to simplify the process even more. Poole calls itself "the butler for Jekyll" and it indeed reduces the time spend to minutes. Of course the default design is no unique, but it allows to have a very quick start.

![demo.png](https://camo.githubusercontent.com/913caa38048fb4c6ed8767ab206f02b3fbc10f40/68747470733a2f2f662e636c6f75642e6769746875622e636f6d2f6173736574732f39383638312f313833313232382f34326166366336612d373338342d313165332d393866622d6530623932336565303436382e706e67)


### Integrating blog as a part of original website

After you clone the poole repository, it's time to link it to your page. For that just edit `_config.yml`:

``` YAML
# Setup
title:               Your Blog Titme
tagline:             ''
description:         ''
home:                http://username.github.io
url:                 http://username.github.io/blog_name
baseurl:             /blog_name
paginate:            5
paginate_path:       "/page:num/"
exclude:             ['posts']
```

Push the changes to your repository and you will be able to see the blog under `%USERNAME%.github.io/blog_name`

### Comments

One of the main things that I was concerned about when creating static blog is having comments functionality. But [Disquis](https://disqus.com/) seems to solve that problem very well. Create a file `_includes/comments.html` which includes the code provided by Disqus after registration. After that add modify the file `_layouts/default.html` to include the line:

``` html
{% include comments.html %}
```

Setting the comments this way allows easy enabling/disabling of comments on a page-by-page basis. All you need have to do is set `comments: True` in the YAML header of the post.

### Analytics

Adding Google Analytics to the blog is similar to adding comments. First, create another account through Google Analytics Admin section as you did for peronal page. Google will give you the javascript tracking code to embed on every website, which you should put in `_includes/google_analytics.html`. Finally, to enable analytics on all of the pages of the blog add the include to `_layouts/default.html`:

``` html
{% include google_analytics.html %}
```

### Social Buttons

There are different options for ways to adding sharing buttons to you blog (for example [this](https://github.com/carrot/share-button) and [this](https://github.com/lipis/bootstrap-social)), but after some searching, I went for simplicity described in this [article](http://codingtips.kanishkkunal.in/share-buttons-jekyll/) on [CodingTips blog](http://codingtips.kanishkkunal.in/). I would encourage reading an original article, but I also wanted to provide here the summary.

As in other steps, create `_includes/share.html` with following contents:

``` html
<div class="share-page">
    Share this on &rarr;
    <a href="https://twitter.com/intent/tweet?text={{ page.title }}&url={{ site.url }}{{ page.url }}&via={{ site.twitter_username }}&related={{ site.twitter_username }}" rel="nofollow" target="_blank" title="Share on Twitter">Twitter</a>
    <a href="https://facebook.com/sharer.php?u={{ site.url }}{{ page.url }}" rel="nofollow" target="_blank" title="Share on Facebook">Facebook</a>
    <a href="https://plus.google.com/share?url={{ site.url }}{{ page.url }}" rel="nofollow" target="_blank" title="Share on Google+">Google+</a>
</div>
```

Next add this to `css/_share.scss`:

``` css
.share-page {
    text-align: center;
    background: $secondary-color;
    color: $light-color;
    padding: 8px 15px;
    border-radius: 5px;
    margin: 1.5 * $spacing-unit 0;

    a {
        font-weight: 700;
        color: #fff;
        margin-left: 10px;

        &:hover {
            border-bottom: 1px dashed #fff;
        }
    }
}
```

And and import in `styles.scss`:

``` css
@import "share";
```

Finally, to enable sharing on all of the pages of the blog add the include to `_layouts/default.html`:

``` html
{% include social.html %}
```

Also keep in mind that same as for comments, sharing can be turned off by setting `comments: False` in the YAML header of the post.

### Basic SEO

To improve search engine recognition of the blog, it's good idea to creat `sitemap.xml`. You can either create one by hand or [use the one automatically generated by GitHub](You can have one automatically generated by GitHub Pages. See this one I use for my site). Mine can be viewed [here]().

### Interesting deployment systems

While setting up personal page and a blog already give a boost in productivity, there are deployment systems that make the process even more efficient:

* [R + knitr](http://kevinushey.github.io/blog/2015/01/03/first-post/). R is a great language primarily used by statisticians and one of it's amazing features is a great support for generating reports using [knitr](http://yihui.name/knitr/). It the linked blog post the author describes blogging using R stack which might be very useful for data analysis related posts.

* [Python + Pweave](http://iaingallagher.tumblr.com/post/41359279059/python-pweave-and-pandoc-howto). Similar to `R + knitr`, [Pweave](http://mpastell.com/pweave/) for `Python` provides an interesting system for generating `markdown` and `html` while embadding code.
