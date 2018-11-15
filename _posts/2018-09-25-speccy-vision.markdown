---
layout: post
title:  "Speccy Vision!"
date:   2018-09-25 12:58:00 +1000
categories: jekyll update
---

<a data-flickr-embed="true" href="https://www.flickr.com/photos/alessandrogrussu/6990446366/in/photolist-bDHSam-dSuPEw-g6Jan8-bPnzSM-bUHFuJ-5LfGjX-bPz6JT-g6Jrn5-bPyCjK-7nydhS-4ZSY7e-7Tuiw-5LQvcZ-cxcPgb-FWhqLb-t97wF-6R6xAN-8H1MHC-d8VkXY-r9dgzx-anniTt-5LUHd1-cZfRwd-bjGBa-5gHtBu-fnAASe-bPzaxn-5LUJ7C-x4RL6-4nNoVo-bn57Vi-fYaTMe-FNPC6d-FNPCdY-pXq2Lj-qYiNv6-4NnfDX-4P7fR2-bn4zPZ-2gM4q-bn59Q2-5LUHTu-bthpbF-8bEuN9-4yrALU-7hKoip-bPghTc-ite8sr-5LUKJW-bjGAK" ><img src="https://farm8.staticflickr.com/7213/6990446366_f07263fdf3_b.jpg" width="1024" height="683"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

Growing up in the 80s, before we had super personal computers in our pockets, a PC would plug into a TV and you would load games on to it with audio cassettes. I remember one day my mum came home with a second hand zx spectrum and a handfull of games. After spending hours loading games and longer playing them, I found this thing called Basic.

I have fond memories of typing in line after line of code from various magazines as a child. The fact that this one little machine could be or do anything drove my imagination wild. I remember trying to simulate ants or write a fishing game(with simulated fish). I remember thinking, with the right code, can a computer think too? This one device is probably why I became a programmer.

So with that in mind, I decided to write a little spectrum filter as a nostalgic trip.

## Background

Old school computers didn't have much memory and they had to employ various tricks to optimise memory usage; the display was no exception.

The resolution of my phone is 2560x1440 pixels. Assuming there is no compression and we are using 24bit color that will use
2560x1440x3 bytes or 11 mb of ram. This is not a problem for my phone since it's got 3gb; however, even the pumped up zx + 2 only had 128k(and some of that is not really usable). The resolution of the zx spectrum was only 256x192, but even at this resolution a 24bit display is 196k. Which is not feasible.

The solution to this problem was to render the screen via blocks with attributes.  The spectrum had 32x24 blocks each 8x8 with 2 possible colors. Each block is stored as 8 bytes for the pixels plus 1 byte for colors. This used 32 x 24 x 9 = 6912 bytes which easily fits into the memory of a spectrum.

The number of colors possble was 15 ( there was 2 blacks )

![alt text](/assets/pal.png){:height="100"}

This, as you can imagine, creates a callenge to make the graphics look nice. In fact, there is an entire sub culture around doing this and it's amazing what people can achieve. I'm not aiming for anything as good a what a professional artist can make but something that looks a little spectrum like.

I've using p5.js for the coding, I've wanted to check it out for a while. check it out here: [p5.js]

## Dithering

First thing we are going to do is convert our input image into two colors. This is generally done via a process called dithering.

Here is a stock image of a sunrise. Let's retro it up!

[![alt text](/assets/sun.jpg "woman-happiness-sunrise-silhouette")][stocksilhouette]

I'm using a simple threshold dithering with the following 4x4 matrix from this wiki site: [dithersrc]

{% highlight javascript %}

let dither_matrix = [
    [0,  8,  2,  10],
    [12, 4,  14, 6 ],
    [3,  11, 1,  9 ], 
    [15, 7,  13, 5 ]
];

{% endhighlight %}

The dithering works using the following steps:

- Convert each pixel into a grey scale shade using the following code

{% highlight javascript %}
let shade = 0.21 * red + 0.72 * green + 0.07 * blue;
{% endhighlight %}
[greyscalesrc]

- Figure out which dither matrix cell we are going to apply to the current pixel. The dither matrix is tiled over the image so we simple use a mod

{% highlight javascript %}
let i = y % 4;
let j = x % 4;
let dither_value = dither_matrix[i][j];
{% endhighlight %}

- Finally we threshold the dither_value with the shade we calculated earlier

{% highlight javascript %}
const ditherFactor = 16
if (dither_value * ditherFactor >= shade) {
    // black
}
else {
    // white
}
{% endhighlight %}

The ditherFactor is the range divided by the number of tones. The range is 256 ( the max shade value ), The number of tones is 16 ( the size of the dither matrix ). So the dither factor is  256 / 16 = 16.
However, you can also change the dither factor for a different results. See the live demo to play with this.

And that's it! here's the result.

![alt text](/assets/sundither.png)

It's looking better (or worse I guess) but time for some color in the blocks.

## Block coloring

For each block we need to select two colors; one for the light dithering and one for the dark dithering.
To start I'll fix the dark dithered pixels to black, this way we only have one color to select for now. The simplest way you might think of for selecting the color is to keep an average for each block and select the nearest speccy color. However there is a problem with doing this since averaging is basically interpolation and if the block is on an edge with two distict colors the resulting color may not fit either color. 

![alt text](/assets/sunaverage.png)

*Color selection using the average color of each block*

The way we are going to get around this problem is to find the best speccy color for each light pixel and bin the results. At the end we will take the max binned, i.e. the dominate color and use that for each block.

![alt text](/assets/sunbinned.png)

*Color selection by binning the best color for each pixel in a block*

What I haven't mentioned is how the best color is selected. The example above is using the [HSL] colorspace. Each pixel is converted to HSL and a simple conditional statement is used to select the appropriate speccy color using the hue attribute.

{% highlight javascript %}
if (sat > 32 && hue >= 0) {
    if (hue > 270) {
        return 13; // magenta
    }
    else if (hue > 210) {
        return 12; // blue
    }
    else if (hue > 150) {
        return 14; // cyan
    }
    else if (hue > 90) {
        return 10;   // green
    }
    else if (hue > 30) {
        return 11; //yellow
    }
    else if (hue >= 0) {
        return 9; // red
    }
}
else {
    // if hue is < 0 or sat is low grey or white is selected.
    let i = 8;
    if (light >= 215) i = 15;
    
    return i;
}
{% endhighlight %}

HSL isn't the only colorspace we can use. RGB will work, but I found HSL worked better in most cases. With RGB the best is selected by calculating the distance between the pixel color and each speccy color.

Here are a few more examples:

[![alt text](/assets/kingfisher.png "kingfisher")][stockfisherattr] | [![alt text](/assets/parrot.png "parrot")][stockparrot]

## Conculsion

This whole thing was a bit of fun really and I don't intend to do anymore work on it. However, a few ideas that might be fun: hooking this up to a web cam, trying more color spaces or even training a neural network to learn the conversion.

Check out the live demo below, Just drag and drop your favourite image and bask in the speccy glory. It supports RGB and HSL and you can fix the dark pixel to black or not.
{% include speccy.html %}

Check out the full source code on github: [speccy].

Also check out [zxart] to see how the professionals do it.

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[zxart]: https://zxart.ee/eng/graphics/top-rated/
[p5.js]: https://p5js.org/
[stockowlattr]: https://pixabay.com/en/bird-owl-eyes-animal-looking-3732867/
[stockfisherattr]: https://pixabay.com/en/kingfisher-animal-avian-beak-bird-1850234/
[stocksilhouette]: https://pixabay.com/en/woman-happiness-sunrise-silhouette-570883/
[stockparrot]: https://pixabay.com/en/ara-parrot-animal-world-3695678/


[dithersrc]: https://en.wikipedia.org/w/index.php?title=Ordered_dithering&oldid=855315468
[greyscalesrc]: https://en.wikipedia.org/w/index.php?title=Grayscale&oldid=858408161
[HSL]: https://en.wikipedia.org/w/index.php?title=HSL_and_HSV&oldid=868777696
[speccy]: https://github.com/rangdo/speccy


