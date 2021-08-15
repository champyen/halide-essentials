# Chapter 3 Fundamentals - Func, Var and Expr

This chapter introduces the fundamentals of Halide langugae.

## 3.1 Open and Save an Image

From Beginning, we need to know how to load and save a 'PNG' or 'JPEG' image. It can be done easily with Halide's **load_image()** and **save_image()**.
In this example, it has 3 steps
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

    save_image(output, "output.png");
    return 0;
}
```


## 3.2 Image Processing Example - Gaussian 3x3
Before we start to discuss the Halide's fundamental classes, we need to understand how image is processed by C/C++ functions. Here we use Gaussian 3x3 kernel as the example.

For a single channel planar image, we can process the image with Gaussian 3x3 as:
```{.c}
/*
 * Here we assume:
 * 1. input and output buffers have the same size.
 * 2. the pitch value of the buffers is the width.
 */
void gauss3x3(unsigned char *in, unsigned char *out, int w, int h)
{
    // The kerenl of Gaussian 3x3 filter
    int weight[3][3] = {{1, 2, 1}, {2, 4, 2}, {1, 2, 1}};
    for(int y = 0; y < h; y++){
        for(int x = 0; x < w; x++){
            // Here we get the iteration for computing of each pixel

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
            *(out + y*w + x) = (sum/16);
        }
    }
}
```

## 3.3 Func, Expr and Var

