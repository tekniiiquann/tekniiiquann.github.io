---
layout: post
title: "Video-Generation-from_Simulation-Pictures-and-Manipulate-Videos-Through-FFmpeg"
categories_short_name: paraview_Insitu
meta: "paraview Insitu"
type: "Draft"
---

starts from number 100 *png with input framerate 10, to combine into *mp4 video
{% highlight console %}
~$ ffmpeg -framerate 10 -start_number 0100 -i gapA-clip_t-%4d.png -c:v libx264 -r 10 -pix_fmt yuv420p tau50-tQ10.86-ARelexing493-100frame.mp4
{% endhighlight %}

Take two groups of pictures from different subfolders with glob naming patterns, then stack them vertically with captions, combine them as *mp4 video
{% highlight console %}
~$ ffmpeg -framerate 7 -pattern_type glob -i "./pMarker-slice2/pMarker-slice2_t-*.png" -framerate 7 -pattern_type glob -i "./pMarker-slice1/pMarker-slice1_t-*.png" -filter_complex "[0:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='y-normal Slice':fontsize=90:fontcolor=black:x=(w-text_w)/2:y=h-text_h-100[v0]; [1:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='z-normal Slice':fontsize=90:fontcolor=black:x=(w-text_w)/2:y=h-text_h-100[v1]; [v0][v1]hstack=inputs=2"  -c:v libx264 -pix_fmt yuv444p p-22-t0.75-periodic.mp4
{% endhighlight %}

Or only using glob for two subfolders' pictures, simply stack them without caption:
{% highlight console %}
ffmpeg -framerate 7 -pattern_type glob -i "./pMarker-slice2/pMarker-slice2_t-*.png" -framerate 7 -pattern_type glob -i "./pMarker-slice1/pMarker-slice1_t-*.png" -filter_complex "[0:v][1:v]vstack=inputs=2:shortest=1" -c:v libx264 -pix_fmt yuv420p p-22-t0.75-AdGRz-bt0.005.mp4
{% endhighlight %}
or 
{% highlight console %}
ffmpeg -framerate 11 -pattern_type glob -i "./pMarker-slice2/pMarker-slice2_t-*.png" -framerate 11 -pattern_type glob -i "./pMarker-slice1/pMarker-slice1_t-*.png" -filter_complex "\
[1:v]pad=iw:ih[bottom]; \
[0:v]pad=iw:ih[top]; \
[top][bottom]vstack=inputs=2" -c:v libx264 -pix_fmt yuv444p p-5.5-T0-0.90-AdGRz-bt0.000001-RS18025.mp4
{% endhighlight %}

combine two *mp4 side-by-side into new *mp4 video 
{% highlight console %}
~$ ffmpeg -i tau50-tQ10.86-ARelexing1000-100frame.mp4 -i tau50-tQ10.86-ARelexing320-100frame.mp4 -filter_complex "hstack,format=yuv420p" -c:v libx264 -crf 18 tauQ50-tQ1-0.86-ARelx1000-320-100frame.mp4
{% endhighlight %}

Combine three *mp4 side-by-side into new *mp4 video with caption and specific font:
{% highlight console %}
ffmpeg -i "p-5.5-T0-0.90-AdGRz-bt0.000001-RS18025.mp4" -i "p-5.5-T0-0.90-AdGRz-bt0.000001-RS256.mp4" -i "p-5.5-T0-0.90-AdGRz-bt0.000001-RS1224.mp4" -filter_complex "\
[0:v]setsar=1,drawtext=text='Failed 1':x=(w-text_w)/2:y=h-text_h-10:fontsize=50:fontcolor=black,pad=iw:ih:(ow-iw)/2:(oh-ih)/2[a]; \
[1:v]setsar=1,drawtext=text='Failed 2':x=(w-text_w)/2:y=h-text_h-10:fontsize=50:fontcolor=black,pad=iw:ih:(ow-iw)/2:(oh-ih)/2[b]; \
[2:v]setsar=1,drawtext=text='Success Seeding':x=(w-text_w)/2:y=h-text_h-10:fontsize=50:fontcolor=black,pad=iw:ih:(ow-iw)/2:(oh-ih)/2[c]; \
[a][b][c]hstack=inputs=3" -c:v libx264 -pix_fmt yuv444p p-5.5-T00.90-Specular-AdGRz.mp4
{% endhighlight %}

Combine two *mp4 videos with different length by duplicating last frame or shorter video: 
{% highlight console %}
ffmpeg -i ~/ReHD3/dyGiLa-project/project_462000960/test/pio-IX-2/insitu/p-22-t0.75-AdGRz-bt1000000.mp4 -i p-22-t0.75-AdGRz-bt0.005.mp4 -filter_complex "[0:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='Specular z-Boundary':fontsize=90:fontcolor=black:x=(w-text_w)/2:y=h-text_h-150,tpad=stop_mode=clone:stop_duration=60[v0];[1:v]drawtext=fontfile=/usr/share/fonts/truetype/dejavu/DejaVuMathTeXGyre.ttf:text='Pair Breaking z-Boundary':fontsize=90:fontcolor=black:x=(w-text_w)/2:y=h-text_h-150[v1];[v0][v1]hstack=inputs=2:shortest=1" -c:v libx264 -preset slow -crf 18 -pix_fmt yuv444p p-22-t0.7-AdGRz-bt1M-vs-bt0.005.mp4
{% endhighlight %}