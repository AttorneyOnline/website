---
layout: post
title: "The quest for WebM"
author: "longbyte1"
---

AO2 only became what it became after getting away with using a label and a QMovie widget to play sprites. But it worked: it could play GIF animations without any pain.

GIF animations are easy enough. But for AC/AO3, I want to aim higher: play WebM files, specifically VP8 files. VP8 has much better compression than GIF and allows for 8-bit transparency, while GIF, that decades-old format whose palette issues get painted over with advanced dithering algorithms, only supports 1-bit transparency.

Here’s the problem, though: Virtually the only way to decode VP8, with alpha, is using an up-to-date version of libvpx, often contained in some larger AV library, which brings us to the one and only FFmpeg (or Libav if you are that kind of person).

If we use the Pyglet library, great, it does support FFmpeg decoding. But not so fast – it does not support FFmpeg decoding with alpha! After many hours of attempting to hack in support, I was unable to do so due to some segfault or another. It seems that any video file tainted with alpha will cause a segfault on `/usr/lib/python3.6/lib-dynload/_ctypes.cpython-36m-x86_64-linux-gnu.so`. Any other file, however, runs perfectly fine.

All right – let’s try plain old FFmpeg bindings. But the examples are made for Python 2, there is no Windows support, and the maintainer is fixing to leave. So, scratch that one.

What about just putting in a movie player – mpv? It works with one instance, but right when we put in the second instance, it falls apart because it is predominantly designed for one instance per process (in mpv’s case, we get a “double free or corruption” error if we try to initialize two instances with the OpenGL video output driver). If we force the libvpx library, which I believe contains the “reference” implementation of VP8, mpv/FFmpeg do actually attempt to decode the alpha, despite the furious rage by the mpv developer two years ago for FFmpeg supporting encoding, but not decoding, of VP8.

Finally, let’s examine how Ren’Py does it. Long story short, it does it using a rather convoluted native C interface that has various guards to work with mobile platforms as well. But it is so tied into the rest of Ren’Py and Screenlang that it would be impractical to extract it. It also includes audio components, but since I am going to use Qt anyway, I do not wish to use Ren’Py’s native audio interface.

This leaves me with no choice other than to write the core renderer in SDL using C and interface it with PyQt in some way, or abandon WebM due to lack of support.

The annoying part is that Pyglet’s FFmpeg decoder does work – this means that a chatroom that supports any sort of video is indeed something that is viable. However, videos that support alpha are not very well supported even in browsers or media players.

Despite its slightly higher convenience, APNG is just as nonstandard as WebM alpha, and most encoding implementations inefficiently stack frames next to each other instead of compressing blocks across frames, leading to greater file sizes compared to GIF. I do not wish to use APNG for this reason.

There are, of course, other obscure formats out there, such as MNG and FLIF, which remain so obscure that they have practically no encoding support.

The quest for a decent animation format continues. But for now, I will have to support only GIF and PNG until I can pin down what the best approach will be for decoding other formats.
