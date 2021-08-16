# Chapter 3 Fundamentals - Func, Var and Expr Abstract Objects

This chapter introduces the fundamentals of Halide langugae. All Halide implementation work around Func, Var and Expr objects.

## 3.1 Open and Save an Image

From Beginning, we need to know how to load and save a 'PNG' or 'JPEG' image. It can be done easily with Halide's **load_image()** and **save_image()**.
In this example, it has 3 steps:
* **load_image()** is used to open input image "input.png". And output buffer is created with the same size inforamtion of the input image.
* The processing flow is quite simple, for all the channel values of each pixel are set to half of original values.
* At last the image is saved as "output.png" with **save_image()** function.

```{.cpp}
#include <iostream>
using namespace std;

#include "Halide.h"
#include "halide_image_io.h"
using namespace Halide::Tools;

int main(int argc, char **argv) {

    // open an image
    Halide::Buffer<uint8_t> input = load_image("input.png");

    cout << "width: " << input.width() << endl;
    cout << "height: " << input.height() << endl;
    cout << "channels: " << input.channels() << endl;

    // create output image
    Halide::Buffer<uint8_t> output(input.width(), input.height(), input.channels());

    // get raw data buffer from the object
    uint8_t *inbuf = input.get()->data();
    uint8_t *outbuf = output.get()->data();
    int len = input.width() * input.height() * input.channels();
    for(int i = 0; i < len; i++)
        *(outbuf + i) = *(inbuf + i)/2;

    // write out the result
    save_image(output, "output.png");

    return 0;
}
```
Here we use the book cover as input image. The luminance of the output image will just be half of the input.


## 3.2 Image Processing Example - Gaussian 3x3
Before we start to discuss the Halide's fundamental classes, we need to understand how image is processed by C/C++ functions. Here we use Gaussian 3x3 as the example.

For a single channel planar image, we can process the image with Gaussian 3x3 as:
```{.c}
#include <stdint.h>

/*
 * Here we assume:
 * 1. input and output buffers have the same size.
 * 2. the pitch value of the buffers is the width.
 */
void gauss3x3(uint8_t *in, uint8_t *out, int w, int h)
{
    // The kerenl of Gaussian 3x3 filter
    int weight[3][3] = {{1, 2, 1}, {2, 4, 2}, {1, 2, 1}};
    for(int y = 0; y < h; y++){
        for(int x = 0; x < w; x++){
            // Here we get the iteration for computing of each pixel
            int sum = 0;

            // apply 3x3 filter and accumulate
            for(int oy = -1; oy <= 1; oy++){
                for(int ox = -1; ox <= 1; ox++){

                    // boundary handling
                    int X = (x+ox) > 0 ? ((x+ox) < w ? x+ox : w-1) : 0;
                    int Y = (y+oy) > 0 ? ((y+oy) < h ? y+oy : h-1) : 0;
                    sum += *(in + Y*w + X) * weight[oy+1][ox+1];
                }
            }

            // calculate final result and write out
            *(out + y*w + x) = (uint8_t)(sum/16);
        }
    }
}
```

Even it is a single channel processing, in the straightforward implementation the depth of the for-loops is still 4. From the perspective of Halide, the reason of nested loops and computations is the mixture of algorithm, scheduling and boundaries. And Human only can afford limitied complexity. Therefore, engineers choose comprehensive ways to implement the image functions despite of possible optimization.

In this example, the concept of Gaussian 3x3 is quite easy. For each output pixel of specific position, the pixel set required to calculate the value of output can be inferred by its coordinate.

[//]: # (TODO add kernel image)

As the shown in the illustration, Gaussian 3x3 multiply 3x3 values to target postion and get the result. Therefore for every coordinate (x, y) the output of Gaussian 3x3 can be represented as a pure math function like:
```
out(x, y) = (
                1*in(x-1, y-1) + 2*in(x, y-1) + 1*in(x+1, y-1) +
                2*in(x-1, y  ) + 4*in(x, y  ) + 2*in(x+1, y  ) +
                1*in(x-1, y+1) + 2*in(x, y+1) + 1*in(x+1, y+1)
            ) / 16;
```


## 3.3 Func, Expr and Var

