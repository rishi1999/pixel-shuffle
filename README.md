![GitHub](https://img.shields.io/github/license/rishi1999/pixel-shuffle)
[![This Repository uses a generated Social Preview from @pqt/social-preview](https://img.shields.io/badge/%E2%9C%93-Social%20Preview-blue)](https://github.com/pqt/social-preview)

# pixel-shuffle

## intro

This is not your traditional image morphing app.

There are two primary ways to do image morphing. The simplest method works by interpolating between two images (i.e., one fades out and the other fades in).
The other involves stretching the features of the image and distorting the colors to blend between two images.

This project differs from both in that it preserves the color of every single pixel in the original image.
Instead of distorting colors, it rearranges all the pixels to produce the second image.
The result is a picture that has the form and structure of the second image and the colors of the first image.

To the best of my knowledge, this repo contains the first publicly available implementation of this approach.

## algorithm

We have two `m` by `n` images, each of which can be interpreted as a set of `mn` RGB tuples.
For each tuple in the first set, we want to find the tuple in the second set that is most similar.
To judge what is most similar, we minimize Euclidean distance (but it would be interesting to experiment with other metrics as well).
Given any tuple from the first set and any tuple from the second set, we can associate with the pair a "cost", given by the Euclidean distance.
Another way to think of this setup is that we have a complete bipartite graph, where the tuples of one set are on one side and the tuples of the other set are on the other side.
We assign each edge a weight based on the cost associated with the pair of endpoints.

Finding the optimal assignment [1] of pixels to construct the image morph is equivalent to finding a minimum weight bipartite matching in this graph.
This is exactly the [balanced assignment problem](https://en.wikipedia.org/wiki/Assignment_problem#Balanced_assignment).
Luckily for us, included in the SciPy library is an [efficient method for solving this problem](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.linear_sum_assignment.html).

## installation

Clone the repo. Or, if you want a lightweight installation, you could download just `pixel-shuffle.py` by itself.

I would recommend creating a Conda environment to avoid messing with your base installation, but this is optional.
You'll need to install a few standard packages (e.g., NumPy, SciPy, Matplotlib).

## usage

### Jupyter Notebook
Place the images you want to morph in the `input/` folder.
Specify their filenames in the code. Run the Notebook.

### Python script
Execute the following command in your terminal:
```bash
python pixel-shuffle.py
```
Add the `--help` flag for specific usage instructions.

## tips

The results tend to turn out better if you
- use images of similar dimensions and decent quality
- use a relatively colorful image for the filler image; it doesn't matter all that much for the skeleton image

The program will take a couple minutes to run on average.
Larger images will be more pixelated to compensate for the increased computational load.
Expect execution time to increase rapidly when using higher `PRECISION` values.

## sample output

This animation is the result of having a chameleon morph into a woman sitting in a chair with `PRECISION=12000`. The original images can be found in the `sample/` folder. Both images were obtained from [Pexels](https://www.pexels.com/).

![GIF of a chameleon morphing into a woman sitting in a chair](sample/sample_out.gif)

[1]: In theory, an adversarial pair of inputs could confuse the algorithm into producing meaningless output. For example, we could pair a filler image in which every pixel has RGB value `(255,0,0)` or `(0,255,0)` with a skeleton image in which every pixel has RGB value of the form `(0,0,x)`. Since the algorithm is color-agnostic, the program would not be smart enough to use red pixels for light green and blue pixels for dark green or some analogous strategy. Many of these adversarial situations can be easily fixed, however, by adding an RGB perturbation of `(0,1,2)`, for example, to each pixel in the skeleton image.
