---
# CS 184 Project 1 Write-Up

## Overview

In this project, we implemented a triangle rasterizer, simple matrix transform functions, and various antialising methods, namely supersampling, pixel sampling, and level sampling. We gained insight on the speed, memory, and antialiasing tradeoffs for each method. We also gained experience working with barycentric coordinates, mipmaps, and the CGL library. 
**

## Task 1: Drawing Single-Color Triangles

To rasterize a triangle, we iterate through each sample point and perform the point-in-triangle test, which uses each edge of the triangle as the 3 line functions. If the point is inside the triangle, then all 3 functions will have a value >= 0. We generate the 3 line functions in both the clockwise and counterclockwise direction to capture the case where the normal vectors for all line functions point towards the inside of the triangle. Lastly, we call fill_pixel to fill the corresponding pixel at the given sample point. 

Our algorithm calculates the bounding box of the triangle by taking the minimum and maximum of the x, y coordinates of the three points.  This prevents us from having to sample the whole image.


![basic/test4.svg](https://paper-attachments.dropbox.com/s_9A43E6EB0936DC6BBACC13DE9773AC1CDA1A8E175AA0A4C1D858D987347DCBEA_1644968973401_basic_test4_ss1.png)



## Task 2: **Antialiasing by Supersampling**

Supersampling is useful because we can sample more points per pixel and average the sample values to get the final value for each pixel. When a pixel is partially within a triangle, it will be colored according to the proportion of supersamples within the triangle, rather colored completely by the triangle color. We thus we have more data to reduce rendering artifacts and smoothen lines (reduce jaggies). 

Our supersampling algorithm is based on the implementation of task 1 where we have a nested for loop to sample each point. But instead of sampling one point per pixel, we sample more points by incrementing x and y by `1/sqrt(sample_rate)`, rather than 1. For example, if the sampling rate is 4, we sample 4 times per pixel by incrementing x and y by 0.5 instead 1. We store these supersampled points inside `sample_buffer`, which we resized from dimensions `width * height`  to `width * height * sample_rate`. In the function `resolve_to_framebuffer`, we transfer the sample points in `sample_buffer` to the `rgb_framebuffer_target`, which is the actual buffer vector for the color pixel. We add up the color values from `sample_buffer` and divide the total by `sample_rate` to get the average value of the points we super sampled. For points and lines, we changed `fill_pixel` so that if we are rasterizing a point, we will fill in the `sample_buffer` multiple times with the same color value so that when we average the `sample_buffer` in the function `resolve_to_framebuffer`, we will still get the same value for points and lines. The accessing pattern for sample_buffer is `sample_buffer[(y + sy) * width * sqrt(sample_rate) + x + sx]`, where x and y = the actual pixel coordinate, sx and sy = the indexes of the extra points we super-sample. We also changed `set_sample_rate` and `set_framebuffer_target` to account for the `sample_rate` when resizing the `sample_buffer`: `this->sample_buffer.resize(width * height * sample_rate, Color::White)`.


![basic/test4.svg: sample rate 1](https://paper-attachments.dropbox.com/s_9A43E6EB0936DC6BBACC13DE9773AC1CDA1A8E175AA0A4C1D858D987347DCBEA_1644969036927_basic_test4_ss1.png)
![basic/test4.svg: sample rate 4](https://paper-attachments.dropbox.com/s_9A43E6EB0936DC6BBACC13DE9773AC1CDA1A8E175AA0A4C1D858D987347DCBEA_1644969054260_basic_test4_ss4.png)

![basic/test4.svg: sample rate 16](https://paper-attachments.dropbox.com/s_9A43E6EB0936DC6BBACC13DE9773AC1CDA1A8E175AA0A4C1D858D987347DCBEA_1644969056775_basic_test4_ss16.png)



## Task 3: Transforms

The PNG shows a cubeman running and chasing a bus. It was created with basic rotation and translation of arms and legs.

![transforms/robot.svg](https://paper-attachments.dropbox.com/s_04A241634DD58DDBCC0251E0197A4C823C6852D83D0B1DC74263F148A0F279E8_1644807958043_screenshot_2-13_19-4-33.png)



## Task 4: Barycentric Coordinates

Barycentric coordinates are a coordinate system that represents the location of a point within a shape (triangle in our case). It tells us the location of the point by the proportion distances between the given points and the point we sample. 

![](https://paper-attachments.dropbox.com/s_04A241634DD58DDBCC0251E0197A4C823C6852D83D0B1DC74263F148A0F279E8_1644913823935_screenshot_2-15_0-30-16.png)


Here is a simple example of a triangle with 3 vertices (one green, one red, and one blue). The color inside the triangle is calculated by the barycentric coordinates: For every sample point, we calculate the proportion distances from the sample point to the 3 vertices respectively, then we assign the 3 colors with their ratio (alpha, beta, gamma) to the sample point. 


![basic/test7.svg](https://paper-attachments.dropbox.com/s_04A241634DD58DDBCC0251E0197A4C823C6852D83D0B1DC74263F148A0F279E8_1644912656087_screenshot_2-15_0-9-36.png)




## Task 5: "Pixel Sampling" for Texture Mapping 

For texture mapping, we used barycentric coordinates to map image coordinates (x, y) to texture space coordinates (u, v) (since barycentric coordinates are conserved in the transformation). Now given sample point (u, v), we retrieved the appropriate texture value with either nearest-pixel sampling or bilinear sampling.  

For nearest pixel sampling, we returned the texture value at the pixel closest to our sample point. For bilinear sampling, we find the four pixels closest to our sample point and perform 3 linear interpolations (2 horizontally, then 1 vertically) to get our final texture value. 

***svg/texmap/text6.svg:*

![nearest sampling at 1 sample per pixel](https://paper-attachments.dropbox.com/s_9A43E6EB0936DC6BBACC13DE9773AC1CDA1A8E175AA0A4C1D858D987347DCBEA_1644913950717_Screen+Shot+2022-02-15+at+12.31.06+AM.png)
![bilinear sampling at 1 sample per pixel](https://paper-attachments.dropbox.com/s_9A43E6EB0936DC6BBACC13DE9773AC1CDA1A8E175AA0A4C1D858D987347DCBEA_1644913950681_Screen+Shot+2022-02-15+at+12.31.12+AM.png)

![nearest sampling at 16 samples per pixel](https://paper-attachments.dropbox.com/s_9A43E6EB0936DC6BBACC13DE9773AC1CDA1A8E175AA0A4C1D858D987347DCBEA_1644913950621_Screen+Shot+2022-02-15+at+12.31.21+AM.png)
![bilinear sampling at 16 samples per pixel](https://paper-attachments.dropbox.com/s_9A43E6EB0936DC6BBACC13DE9773AC1CDA1A8E175AA0A4C1D858D987347DCBEA_1644913950571_Screen+Shot+2022-02-15+at+12.31.27+AM.png)


Bilinear sampling results in a less “pixelized image”, in that the color of neighboring pixels change less drastically. This is more obvious with 1 sample per pixel images (upper two), since the 16 samples per pixel images (lower two) are heavily blurred when viewed with the pixel inspector. 

There will be a large difference between nearest and biliear pixel sampling when screen pixel area is smaller than the texture pixel area (minification). A small change in screen pixel location will result in a relatively large change in texture value location. Bilinear pixel sampling mitigates this drastic jump in texture value by taking the linear interpolation of the 4 nearest texture pixel values. 


## Task 6: "Level Sampling" with Mipmaps for Texture Mapping 

Level sampling uses mipmaps to store different resolutions of a texture map so that we can use different textures in the same picture. If we only have one level, then objects that are “further away” in the picture might have aliasing while the closer objects are fine. In this case, we can use a higher level texture map with less texels to “down-sample” the objects that appear to be aliased. 

We implemented texture mapping by first using the sample’s texture coordinates to find its appropriate mipmap level using the formula in lecture. Then we applied either sample_nearest or sample_bilinear with the level that we calculated. Note that we “clamp” the level so that the level that we calculated is always within the level bound (>= 0 and <= mipmap.size() - 1)

    

**Pixel Sampling: sample nearest or bilinear**

- Sample nearest has constant time lookup for getting the texture value of a sample, whereas sample bilinear looks up 4 texture values and uses 3 linear interpolations to get the final texture value. More computation is thus required to perform these linear interpolations (vector operations), but the amount of additional memory required is negligible. 
- Bilinear pixel sampling is effective in reducing aliasing, since it reduces the difference in texture value between neighboring screen pixels by finding an averaged texture value based on each sample’s texture coordinates. Compare this to antialiasing via supersampling, which ignores how screen samples are mapped to texture space. 

**Level Sampling: level 0, nearest, or linear**

- Level Sampling requires storing additional mipmaps, which are lower resolution textures. The additional memory is, however, not significant (only 4/3x the full resolution texture). Nearest and linear level sampling require computing each sample’s mipmap level, which is 2 square root, 2 power, and 1 log operation. Linear level sampling performs an additional texture lookup and linear interpolation, compared the other two level sampling methods.
- Level Sampling is only effective for reducing antialiasing when an image contains objects that appear at varying depths of field (changing the level sampling method bears no effect for the images in the basic directory). Further objects use a lower resolution texture map, which reduces aliasing. When all screen samples have the same mipmap level, we should adjust the sampling rate to reduce aliasing. 

**Supersampling: 1, 4, 9, or 16 samples per pixel**

- Computation time and memory usage increases according to the sample rate. If we are sampling 16x per pixel, we must compute and fill 16x more sample values. 
- Increasing the sampling rate has an effect of “blurring” the image, which makes edges less sharp by reducing jaggies. However, this is not the most effective method of antialiasing, as it averages pixel texture values instead of finding a more “accurate” texture value per pixel, ignoring how screen samples are mapped in texture space. 

Based on some experimentation with the GUI, supersampling may require the most additional computational time, followed by pixel sampling, then level sampling. Additional memory usage for each method can be ranked as supersampling, level sampling, then pixel sampling, with sampling rate antialiasing using the most additional memory. Lastly, pixel sampling is effective at reducing the difference in texture values between neighboring screen pixels; level sampling is effective at reducing aliasing in “far away” objects in the image; and supersampling is simply effective at blurring the image to reduce jaggies. 



![level zero and nearest pixel sampling](https://paper-attachments.dropbox.com/s_04A241634DD58DDBCC0251E0197A4C823C6852D83D0B1DC74263F148A0F279E8_1644918093771_image.png)
![level zero and bilinear pixel sampling](https://paper-attachments.dropbox.com/s_04A241634DD58DDBCC0251E0197A4C823C6852D83D0B1DC74263F148A0F279E8_1644918147459_image.png)

![nearest level and nearest pixel sampling](https://paper-attachments.dropbox.com/s_04A241634DD58DDBCC0251E0197A4C823C6852D83D0B1DC74263F148A0F279E8_1644918455423_image.png)
![nearest level and bilinear pixel sampling](https://paper-attachments.dropbox.com/s_04A241634DD58DDBCC0251E0197A4C823C6852D83D0B1DC74263F148A0F279E8_1644918513645_image.png)


