---
layout: post
title: "FFmpeg Video Generation and Manipulations from Pictures"
categories_short_name: paraview_Insitu
meta: "paraview Insitu"
type: "publication"
---
If you're sitting on a mountain of simulation snapshots and need to transform them into a video for your next big presentation, [FFmpeg](https://ffmpeg.org/) is your best friend. Here is a list of tested command lines to make the process completely painless.

Combine *.png pictures to *.mp4 video with input framerate 10 and starting frame 100:
{% highlight console %}
~$ ffmpeg -framerate 10 -start_number 0100 -i clip_t-%4d.png \
-c:v libx264 -r 10 -pix_fmt yuv420p \
v1-output.mp4
{% endhighlight %}

But what about pulse few seconds before new frames are imported?
The next lines of commands combine *.png pictures to *.mp4 video inaddtion to pulse 0th frame 3 seconds:
{% highlight console %}
~$ ffmpeg -framerate 10 -start_number 100 -i clip_t-%4d.png \
-vf "tpad=start_mode=clone:start_duration=3" \
-c:v libx264 -pix_fmt yuv420p \
v1-output.mp4
{% endhighlight %}


Take two groups of pictures from different subfolders with glob naming patterns, then stack them horizontally with captions (text_h counted from top, the less it's extracted, the lower text is), combine them as *mp4 video
{% highlight console %}
~$ ffmpeg -framerate 7 -pattern_type glob -i "./slice2/slice2_t-*.png" \
-framerate 7 -pattern_type glob -i "./slice1/slice1_t-*.png" \
-filter_complex "\
[0:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='y-normal Slice':fontsize=90:fontcolor=black:x=(w-text_w)/2:y=h-text_h-100[v0]; \
[1:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='z-normal Slice':fontsize=90:fontcolor=black:x=(w-text_w)/2:y=h-text_h-100[v1]; \
[v0][v1]hstack=inputs=2" \
-c:v libx264 -pix_fmt yuv444p v1-output.mp4
{% endhighlight %}
Other almost same application, with different font size:
{% highlight console %}
ffmpeg -framerate 15 -pattern_type glob -i "./short/short-frames.*.png" \
-framerate 15 -pattern_type glob -i "./long/long-frames.*.png" \
-filter_complex "\
[0:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='short-axis distribution':fontsize=20:fontcolor=black:x=(w-text_w)/2:y=h-text_h-20[v0]; \
[1:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='long-axis distribution':fontsize=20:fontcolor=black:x=(w-text_w)/2:y=h-text_h-20[v1]; \
[v0][v1]hstack=inputs=2" \
-c:v libx264 -pix_fmt yuv444p v1-output.mp4
{% endhighlight %}

Or only using glob for two subfolders' pictures, simply stack them without caption:
{% highlight console %}
~$ ffmpeg -framerate 7 -pattern_type glob -i "./slice2/slice2_t-*.png" \
-framerate 7 -pattern_type glob -i "./slice1/slice1_t-*.png" \
-filter_complex "\
[0:v][1:v]vstack=inputs=2:shortest=1" \
-c:v libx264 -pix_fmt yuv420p v1-output.mp4
{% endhighlight %}

One can also stack video vertically, by using `[top][bottom]vstack=inputs=2` instead:
{% highlight console %}
~$ ffmpeg -framerate 11 -pattern_type glob -i "./slice2/slice2_t-*.png" \
-framerate 11 -pattern_type glob -i "./slice1/slice1_t-*.png" \
-filter_complex "\
[1:v]pad=iw:ih[bottom]; \
[0:v]pad=iw:ih[top]; \
[top][bottom]vstack=inputs=2" \
-c:v libx264 -pix_fmt yuv444p v1-output.mp4
{% endhighlight %}

Combine two *mp4 side-by-side into new *mp4 video 
{% highlight console %}
~$ ffmpeg -i v1-output.mp4 \
-i v2-output.mp4 \
-filter_complex "
hstack,format=yuv420p" \
-c:v libx264 -crf 18 v3-output.mp4
{% endhighlight %}

Combine three *mp4 side-by-side into new *mp4 video with caption and specific font with explicit location:
{% highlight console %}
~$ ffmpeg -i "v1-output.mp4" \
-i "v2-output.mp4" \
-i "v3-output.mp4" \
-filter_complex "
[0:v]setsar=1,drawtext=text='Failed 1':x=(w-text_w)/2:y=h-text_h-10:fontsize=50:fontcolor=black,pad=iw:ih:(ow-iw)/2:(oh-ih)/2[a]; \
[1:v]setsar=1,drawtext=text='Failed 2':x=(w-text_w)/2:y=h-text_h-10:fontsize=50:fontcolor=black,pad=iw:ih:(ow-iw)/2:(oh-ih)/2[b]; \
[2:v]setsar=1,drawtext=text='Success Seeding':x=(w-text_w)/2:y=h-text_h-10:fontsize=50:fontcolor=black,pad=iw:ih:(ow-iw)/2:(oh-ih)/2[c]; \
[a][b][c]hstack=inputs=3" \
-c:v libx264 -pix_fmt yuv444p v4-output.mp4
{% endhighlight %}

Combine two *mp4 videos with different length by duplicating last frame or shorter video: 
{% highlight console %}
~$ ffmpeg -i ~/path/to/v1-output.mp4 \
-i v2-output.mp4 \
-filter_complex "\
[0:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='Specular z-Boundary':fontsize=90:fontcolor=black:x=(w-text_w)/2:y=h-text_h-150,tpad=stop_mode=clone:stop_duration=60[v0];\
[1:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='Pair Breaking z-Boundary':fontsize=90:fontcolor=black:x=(w-text_w)/2:y=h-text_h-150[v1];\
[v0][v1]hstack=inputs=2:shortest=1" \
-c:v libx264 -preset slow -crf 18 -pix_fmt yuv444p v3-output.mp4
{% endhighlight %}