# homework-3-case-study

# Student Info

- Name: Mauricio Mutai
- PennKey: `mmutai`

# Electron Orbitals

## Reference Animation

- Taken from the [original homework repo](https://github.com/CIS-566-2018/homework-3-case-study).

![](ellipse/ellipse.gif)

## My Implementation

- [https://www.shadertoy.com/view/Mt2BDt](https://www.shadertoy.com/view/Mt2BDt)

## Techniques Used

### Single Ellipse, Bidirectional

- I noticed the original animation had one base ellipse animation, which is then repeated with slightly different parameters in order to create the overall animation. So I tried to create a single ellipse animation.
- This is handled by `getEllipseColor()`. This is a slightly misleading name, because it really is used to draw circles.
    - The aim of this function is to return a color as follows:
      - Red, if the point is "on" the circle.
      - White, if the point on the moving white point (see reference image).
      - Blue otherwise.
    - We find the angle `angle` of the input position `p` using `atan()`.
    - We compute the point on the circle with a fixed radius of 1 and angle `angle`. This could be accomplished by just normalizing `p` as well.
    - We compute a time-dependent `angleLimit` using a cosine. The idea is that points with an angle "beyond" `angleLimit` are not on the circle. This creates the effect of the white moving circle "tracing" the red ellipse.
      - Conveniently, `cos()` and `atan()` return the same range of values (`[-1, 1]`). This made it easy to compare them initially, but an additional complication was added later.
      - Note I say the angle is "behind" `angleLimit` instead of just less than `angleLimit` because the definition of "behind" changes over the animation.
    - We use `angleLimit` to compute the position of the moving white dot.
    - If `p` is within a certain distance of the moving white dot, we return white.
    - If `p` is within a certain distance of the point on the circle AND `angle` is "behind" `angleLimit`, we return red.
    - Otherwise, we return blue.
- The trick to making `getEllipseColor()` to draw ellipses instead of circles is to simply scale the input position `p`. I initially attempted a more "pure" ellipse drawing function that would preserve line thickness along the ellipse, but I noticed the original animation itself has varying line thickness caused by the scaling.
    - The moving white dot is also clearly affected by this scaling effect in the original animation.

### Single Ellipse, Unidirectional

- As described above, `getEllipseColor()` draws a circle where the white moving dot moves back and forth, "tracing" the full circle when going in one direction, and "erasing" it as it goes in the other.
- However, the original animation has the white moving dot going only in one direction.
- To fix this, I introduced a `goingBack` boolean variable. 
    - `goingBack` is computed so as to look like a square wave with 50% duty cycle and frequency of 2 `PI`. This is accomplished with `mod(adjTime, 2.0 * PI) > PI`, where `adjTime` is a scaled time variable.
    - This makes `goingBack` false for the first half of the animation, and true for the second half.
    - If `goingBack` is false, the function behaves like above. If it is true, then we flip the sign of the `adjTime` used to compute `angleLimit`. We also remove an offset of `-PI` from `adjTime` that is usually added to make `adjTime` start on a rising part of the cosine curve.
      - These changes to `adjTime` essentially shift it to the correct part of the cosine curve such that `angleLimit` increases again for the second half of the animation, instead of decreasing.
    - If `goingBack` is true, we also negate the `isBehind` boolean that says whether `angle` is "behind" `angleLimit`.

### Tweaks to Single Ellipse

- In order to make the white moving dot spend more time closer to its poles (i.e. the points where the ellipse is either not fully drawn or fully drawn), I use a Perlin gain function with `gain < 0.5`. This gives the desired effect of spending more time at the edges of the `[0, 1]` range, rather than the middle.
- I noticed the white moving dot does not start at the "top" of the ellipse, but it is instead offset by a little bit (see image below).
![](ellipse/offset.png)
  - This can be fixed by adding a constant offset (say, `LIMIT_ORIGIN`, or `L_O`, for short) to `angleLimit` to shift its range to `[-PI + L_O, PI + L_O]`. However, this makes the comparison with `angle`, which is in `[-1, 1]`, not work anymore. Why? Assuming `L_O < 0`, the region `[PI + L_O, PI]` of `angle` is visually in the `[-PI + L_O, -PI]` range, but not numerically. The solution to this is explained with more depth in the comments for `isBehindAngleLimit()`, but it essentially treats the `[PI + L_O, PI]` region of `angle` as a special case.

# Assignment Description

For this assignment, you will re-create various animations demonstrating a combination of toolbox functions and the rendering techniques you've already learned. The motivation for this is to help you become more familiar with toolbox functions as well as give you experience in producing a desired aesthetic.

Below are multiple examples of periodic animations with assigned point values. You are required to complete EITHER:
* One intermediate animation and one hard animation
* Three intermediate animations
* One Duck-Level-hard animation (at the very bottom)

You have three options for implementing these:
* Rasterizer, like homework 1
* Raymarcher / implicit surfaces, like homework 2
* Shadertoy (probably the easiest)

You can also implement different scenes with different methods.

If you'd like to implement any animations that are not listed below, you can make a _private_ post on Piazza and we will decide if they are appropriate for the assignment, and their difficulty classification. Check [/r/loadingicon](https://www.reddit.com/r/loadingicon) on Reddit for inspiration.

*Extra Credit*: you can earn extra credit by creating interesting twists e.g. materials and animations that are not in the original reference. Since we are grading reference matching, additional features must be toggleable, either by a GUI option in your webgl site or by a #define statement in your Shadertoy code. This must be described in your writeup. Additionally, some of the references have some attributes that are exceptionally difficult, and do not need to be implemented for full credit; we will note what these are and they can count for extra credit.

# Submission

You must submit a writeup for this assignment by 11:59 PM on Thursday, Feb. 8th to canvas *as a .pdf or .txt*. If your projects are hosted on Github you should copy your writeup to a readme.

Writeups must include, for each scene:
* A link to the online implementation
* A link to the reference animation
* *Detailed* description of techniques used to emulate the reference, for both motion and rendering
* If you implement extra credit, explain what it is and how to toggle it


# Evaluation

Each intermediate scene is worth 1/3 credit. Difficult scenes are 2/3 credit. Duck-level scenes are worth 100% credit.
Extra credit is at grader discretion and cannot exceed 20 points.

*If we cannot view your work online it will receive no credit.*

All shaders will be graded by this scheme:
* 65% Reference matching: does this show understanding of the motion, colors, and rendering techniques required to create the animation? This does not have to be pixel-perfect for full credit.
* 20% Writeup completeness
* 15% Performance considerations: motion should be fluid, ideally no less than 30 FPS at Shadertoy-resolution on a gaming laptop. If your shader (without EC) is seriously under-performant there will be a point penalty.
