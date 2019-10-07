---
layout: post
title: "TIL: resize images with convert"
categories: TIL
---
Imagemagick provides, among others, a really useful tool for modifying images -- `convert`.
I use it for creating backgrounds for my computer - example in this post.
Resolution 1440x900, which means that aspect ratio of 16:10.

First find size of image with ffprobe.

Calculate the new size, I use bc but you can use others:

```bash
bc <<< "size/10*16"
```

Then to set the new size, use

```bash
convert input_file.jpg -resize xNEWSIZE\! output_file.jpg
```

The `\!` means to ignore aspect ratio, has to be escaped because is special shell character.
Size is specified WIDTHxHEIGHT, if you omit WIDTH it's calculated automatically.
