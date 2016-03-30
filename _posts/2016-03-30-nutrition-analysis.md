---
layout: post
title: "Data analysis with python, plot.ly, fitocracy, and myfitnesspal"
modified: 2016-03-30 10:30:00 -0800
tags: [automation,data,fitness,lifting,python,myfitnesspal]
image:
  feature: barbell.jpg
  credit: 
  creditlink: 
comments: true
share: 
---

Over this past week I've been having a lot of fun revisiting python and doing some data analysis with a myfitnesspal library. After losing a little bit of weight over the past few years (320+ pounds down to 210 pounds), and people asking me how I did it, I came to the realization that all of the data was right there waiting to be looked at! Not only that, but I am in a rare position that I've logged almost every single meal, snack, whatever, since August of 2013. Below are some of the graphs that came out of me playing with that data.

People do ask me exactly what I ate during that time, and I feel like the best advice I can give is I attempted to follow this diet plan pretty closely: [The Stronglifts diet.](http://stronglifts.com/stronglifts-diet-muscle-gains-strength-building-fat-loss/)

I didn't do everything (like eating every 3 hours, or carbs post workout only) but it was a good guide on eating healthier. If your someone who exercises a lot or lifts weights, then I recommend checking it out. If you don't do those things, then do them! :D But in all seriousness, there's a lot of good information detailed there that would work for everyone. You just have to do your homework and figure out what works for you.

### Nutrition Analysis Script

Recently I discovered a pretty nifty python library called [myfitnesspal](https://github.com/coddingtonbear/python-myfitnesspal). It's essentially a web scrapper that, given a day, will scrape the html for that log entry on MFP and return all of the data that was stored. It's pretty jank, but it works\*!

Given that, I was able to take that data and generate some pretty cool plots with [plot.ly](http://plot.ly/) and their python library! It was super easy, so shout out to them for being awesome.

The script itself isn't too flexible, as some chart methods expect lifting data from fitocracy. However if you wanted to use it yourself, you should be able to just comment out those methods, edit your start date in the main method, and add your username and password to a yaml file (check the [readme](https://github.com/briancain/nutrition-analysis#login-file-format) on how to do that)

<small>\* Ok it mostly works. Sometimes myfitnesspal craps out and returns some bogus html. This is super frustrating if you're trying to retrieve 3 years worth of data.</small>

# Graphing The Data

Below are some of the graphs I generated from the data I gathered from myfitnesspal and fitocracy.

In each case, you can click the `View Larger Version` link for each chart to view an interactive graph with a lot more detail (and for the more complicated line charts you can even hide some data or zoom in).

## Average Macro Nutrients

This chart is a breakdown of macronutrients (Carbohydrates, Protein, and Fat) that myfitnesspal keeps track of for me. This is just an average percentage from calories for each macro over the entire span of 3 years.

This text is kind of tiny, but the breakdown is as follows:

- Carbohydrates
  + 40.1%
- Protein
  + 37.1%
- Fat
  + 22.8%

I've dialed in these macros lot better as of the past year so carbs and protein have actually switched positions. As a powerlifter (especially one attempting to lose weight), you really have to get the most protein out of your meals since you are limiting how my calories per day you can have. If you look at the bar chart down below, you can see the time period where I cut the hardest in 2015. My carbs were extremely low and protein was pretty high during that time period.

<figure>
  <img src="/images/macropie.png" alt="macro pie chart">
  <figcaption>Pie chart with average macro nutrients as a percentage from total calories for 7-28-2013 to 3-29-2016</figcaption>
</figure>

[View Larger Version](/charts/avg-macro-pie.html)

Next we have my macros for each day represented as a stacked bar chart over time. As I mentioned before you can see in 2015 I was able to increase protein in take and decrease carbohydrate intake. Doing this allowed for me to be able to recover from my workouts, without having to eat a bunch of calories (usually from carbohydrates). This part was pretty important for me if I wanted to continue to lift at the level from where I started at, and for a short while even gain muscle.

<figure>
  <img src="/images/macrostacked.png" alt="macro bar chart">
  <figcaption>Bar chart with macro nutrients for 7-28-2013 to 3-29-2016</figcaption>
</figure>

[View Larger Version](/charts/stacked-bar-macros.html)

## Body Weight Vs Calories

This is the chart I initially had in mind when I started playing with this data. I think it's probably one of the most interesting ones. It graphs my weight loss starting at 320 pounds all the way to 210 pounds. I was heavier than that I'm sure, I just didn't have a scale and didn't use myfitnesspal before that weight.

One of the most important changes I made for losing weight was just controlling how many calories I ate each day and made sure my average calories for the week fit within the goal I had set for myself. It wasn't any crazy diet plan or exercise program. Doing this is where I saw the most results, and I think the graph shows that. Now of course as a result of that, I was able to discover what foods were contributing the most calories throughout the day and cut those foods from my diet. The first thing I remember cutting was calories from drinks. Stuff like soda and juice were contributing almost 1,000 extra calories a day!

The obvious trend to me from this chart was the time when my weight dropped the most. Around January 2015 I decided to start cleaning up my diet and become a bit more serious about nutrition. That included being more consistent about how many calories I was eating as well as keeping my macro nutrients at a certain percentage that allowed myself to continue to lift at a level I was happy with. This was all leading up to a long period of stagnation for weight loss, but it looks like that helped break through that plateau.

**Note:** The calorie days that are 0 are just days that I didn't log. The high spikes are super cheat days.... ;-)

<figure>
  <img src="/images/weightvcals.png" alt="weight vs calories">
  <figcaption>Calories per day and total body weight over 7-28-2013 to 3-29-2016</figcaption>
</figure>

[View Larger Version](/charts/weight-cals-series.html)

## Body Weight Vs Lifts

This chart takes my [fitocracy](http://fitocracy.com/) data and graphs it along side my body weight over time. Each data point is the "top set" of my workout for that day (or in other words the most weight that I did for that day).

This chart was mostly for my own interest. I've only been powerlifting for about 3 years (around the time where I started logging my food), so the big question I had was am I getting stronger after all that weight loss? It looks like yes! Especially within the last few months. I've stopped trying to lose weight and I've started focusing on getting stronger.

<figure>
  <img src="/images/liftvsweight.png" alt="top sets vs body weight">
  <figcaption>Top sets for deadlift, bench, and squat, with body weight over 7-28-2013 to 3-29-2016</figcaption>
</figure>

[View Larger Version](/charts/weight-lift-series.html)

## Progress

2016 has been about rebuilding some of the strength lost from 2015. So far I think it's going well! I've increased my numbers beyond what they were at any point in my lifting career, and I've signed up for my first powerlifting meet happening in April.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Officially signed up as a drug tested, raw powerlifter for the America Powerlifting Association world qualifier competition next month! <a href="https://twitter.com/hashtag/yas?src=hash">#yas</a></p>&mdash; Brian Cain (@brian_cain) <a href="https://twitter.com/brian_cain/status/711690489477464064">March 20, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I feel like this also warrants some proof, or as Instagram would call it #ProgressPics #FitFam #LiftIt (why do they use so many hashtags in Instagram?) On the left is me about 2.5 years ago in 2013. I think this was a few months before I started logging all of my meals. The photo on the right was me a month ago!

<blockquote class="instagram-media" data-instgrm-captioned data-instgrm-version="6" style=" background:#FFF; border:0; border-radius:3px; box-shadow:0 0 1px 0 rgba(0,0,0,0.5),0 1px 10px 0 rgba(0,0,0,0.15); margin: 1px; max-width:658px; padding:0; width:99.375%; width:-webkit-calc(100% - 2px); width:calc(100% - 2px);"><div style="padding:8px;"> <div style=" background:#F8F8F8; line-height:0; margin-top:40px; padding:50.0% 0; text-align:center; width:100%;"> <div style=" background:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACwAAAAsCAMAAAApWqozAAAAGFBMVEUiIiI9PT0eHh4gIB4hIBkcHBwcHBwcHBydr+JQAAAACHRSTlMABA4YHyQsM5jtaMwAAADfSURBVDjL7ZVBEgMhCAQBAf//42xcNbpAqakcM0ftUmFAAIBE81IqBJdS3lS6zs3bIpB9WED3YYXFPmHRfT8sgyrCP1x8uEUxLMzNWElFOYCV6mHWWwMzdPEKHlhLw7NWJqkHc4uIZphavDzA2JPzUDsBZziNae2S6owH8xPmX8G7zzgKEOPUoYHvGz1TBCxMkd3kwNVbU0gKHkx+iZILf77IofhrY1nYFnB/lQPb79drWOyJVa/DAvg9B/rLB4cC+Nqgdz/TvBbBnr6GBReqn/nRmDgaQEej7WhonozjF+Y2I/fZou/qAAAAAElFTkSuQmCC); display:block; height:44px; margin:0 auto -44px; position:relative; top:-22px; width:44px;"></div></div> <p style=" margin:8px 0 0 0; padding:0 4px;"> <a href="https://www.instagram.com/p/BCZdIc9Nn7O/" style=" color:#000; font-family:Arial,sans-serif; font-size:14px; font-style:normal; font-weight:normal; line-height:17px; text-decoration:none; word-wrap:break-word;" target="_blank">I know it isn&#39;t #transformationtuesday yet but I gotta post this anyway. It&#39;s been about 2.5 years since I started taking my diet seriously and lifting seriously. The photo on the left is from summer 2013 and the one on the right is tonight in 2016. Down about 110 pounds and stronger than I was in 2013!</a></p> <p style=" color:#c9c8cd; font-family:Arial,sans-serif; font-size:14px; line-height:17px; margin-bottom:0; margin-top:8px; overflow:hidden; padding:8px 0 7px; text-align:center; text-overflow:ellipsis; white-space:nowrap;">A photo posted by Brian Cain (@brian.cain) on <time style=" font-family:Arial,sans-serif; font-size:14px; line-height:17px;" datetime="2016-03-01T04:54:36+00:00">Feb 29, 2016 at 8:54pm PST</time></p></div></blockquote>
<script async defer src="//platform.instagram.com/en_US/embeds.js"></script>

# Resources

- [Nutrition analysis script](https://github.com/briancain/nutrition-analysis)
- [Stronglifts diet](http://stronglifts.com/stronglifts-diet-muscle-gains-strength-building-fat-loss/)
