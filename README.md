# Stitching Photo Mosaics

This project is about stitching together many different photos of the same scene into a
single bigger mosaic.

**Part A is in [`main.ipynb`](main.ipynb), Part B is in [`main_B.ipynb`](main_B.ipynb).**
There is a `requirements.txt` for all the libraries we need. I have saved the results of
the tasks into the folder called `output`.

## Part A: Image warping and mosaicing

In the first part we warp and composite some photographs that we have taken.

Our first task is to find the homographies that map the points in the first image to the
points of our target. The homography matrix is a 3x3 matrix that maps the points from one
image to another. We can solve this equation by using the least squares method.

For the warping function I am using an inverse warp and nearest neighbor interpolation.
Once we have the H, we can just apply it to our first image and get the rectified result.

To create the mosaic we have to blend the two images together. For that I first calculated
the destination of the warped corners of our source image and then created a destination
image. In the parts where they overlap I used a simple mask to alpha blend the images
together. This mask actually did not work so well, which is why I returned to using
Laplacian blending to blend the borders.

## Part B: Feature matching for auto-stitching

The goal in our second part is to implement a system that can automatically find
correspondences between feature points and stitch together the images automatically.

- **Harris interest point detector.** We use the `corner_harris` and `peak_local_max`
  functions from `skimage.feature` to find the feature points in the image.
- **Adaptive non-maximal suppression.** Here we see that there are a lot of points, some
  are just very close to each other. I reduced the amount of points detected using the
  Harris detector; the previous amount was way too much and took too long.
- **Feature descriptor extraction.** We choose a 40x40 pixel window and then downscale it,
  which gives us an 8x8 patch.
- **Feature matching.** To find good matching features we want to select features that
  have a clear match: the distance to the first nearest neighbour should be small, but the
  distance to the second nearest neighbour should be large.
- **RANSAC.** We randomly select 4 matching feature points and then calculate the
  homography. Then we use this homography to warp all the points.

Here I am choosing the top 25 points and the result is insanely good actually. I was
desperately debugging the code but had actually just mixed up the x and y coordinates.

## What I have learned

I often use the panorama feature on my phone to take panorama shots. I did not think that
creating panorama shots was this easy. It was very interesting to see how well the
homography matrix could rectify parts of images that were taken from different angles.
Compared to the manually chosen correspondences, the RANSAC method works very well.
