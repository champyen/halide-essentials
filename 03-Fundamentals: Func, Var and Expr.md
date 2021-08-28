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

<img src="./assets/03_gausian3x3.svg" title="Fig 3-1. applying gaussian 3x3 kernel" width="400" height="400">

As shown in the Fig. 3-1, the process of each pixel of Gaussian3x3 is to multiply 3x3 kernel to target 3x3 square which is centered at the same coordinate of the corresponding output. Then the output value is done by summing of the results of multiplications and a division (divied by the sum of the 3x3 kernel values). Therefore for every coordinate (x, y) the output of Gaussian 3x3 can be represented as a pure math function like:

```
out(x, y) = (
                1*in(x-1, y-1) + 2*in(x, y-1) + 1*in(x+1, y-1) +
                2*in(x-1, y  ) + 4*in(x, y  ) + 2*in(x+1, y  ) +
                1*in(x-1, y+1) + 2*in(x, y+1) + 1*in(x+1, y+1)
            ) / 16;
```

And let's try to apply **gauss3x3()** in the example 3.1. The code will look like:

```
// TODO - use gauss3x3 as the process in example of 3.1
```


## 3.3 Func, Expr and Var

Halide is a programming language embedded in C++. It provides three fundamental classes to programmers for image processing (mostly, in fact Halide can be used for many applications). Programmers must make use of the three classes to define the "Functions" used to manipulate images. The three classes are: Var, Expr and Func.

**Func** - A Func object represents a stage in processing pipeline and it is the only schedulable object in Halide programming language. A **processing pipeline** is the whole flow of steps(of specific filter/kernel/function) from input source to output data. A Func object defines the correlation between arguments and expressions. An intuitive way to understand Func object is that it can be applied to an input data buffer and generates an data output buffer. It is worth to know that arguments of all Func objecst only have integer type. Since a Func is used for image process, the values of arguments come from the coordinate space of images.

**Expr** - In official document Expr is described as: "implemented as reference-counted handle to a concrete expression node, but it's immutable, so you can treat it as a value type." In Halide programming's view, any manipulations or calculations of Var, Expr and Func objects generates Expr objects. And a Func object is defined by assigning an (implicit or emplicit) Expr object (even an expression of a simple Var object or complicated calculation of Var objects). For beginner it is difficult to tell the usage of Func and Expr objects. There are some key points of Expr object:
* Expr object has no arguments but Func object does.
* Expr object can't be scheduled (only Func can be scheduled)
* Expr object can't be used for scheduling(only Var can be used for scheduling).
A Expr is used as a symbol of complex flow. For optimization consideration, Expr objects are good points considered to be replaced by Func objects with same calculation expressions.

**Var** - In official Halide document or tutorial, Var objects are describe as names of variables used in the definitions of Expr or Func objects and has no other meaning. Even it is true, but this statement makes programmer carelessly to think about Var. The Func and Expr objects consists of calculations and manipulations of Var objects. All of the scheduling of computation and storage are considered in the temporal or spatial granularity of Var objects. It is important to understand the relationship between a Var object and its role in a snippet of C code.



