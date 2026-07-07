---
title: "Software Renderer in C ( Part 1 ) "
date: 2026-05-11
draft: false
---

Creating a software rasterizer is, imo, one of the best ways to actually learn computer graphics. Especially in C, where there is little to no hand holding. There are some really good resources out there like [tinyrenderer](https://github.com/ssloy/tinyrenderer) (in C++) and [Gabriel Gambetta's graphics programming series](https://gabrielgambetta.com/computer-graphics-from-scratch/), and i've gone through pretty much all of them. This is my attempt to break each part down as much as i can 

WARNING: most of this will make more sense if you've know atleast something on this topic or even my code for gevurah. Computer graphics is very visual and a written blog can only do so much, but intuition goes a long way.

---

## Windowing

Windowing is handled by SDL2, which is the only external library i'm planning to use here. SDL2 gives you just enough to deal with creating a window, handling events, and getting input without having to touch the OS directly. Things like `SDL_GetTicks()` for timing and `SDL_QUIT` for catching the close event are what we'll be using throughout. We'll get into specifics as we go.

---

##  Game Loop

The game loop is exactly what it sounds like. A loop that keeps everything running until the user closes the window. 

```c
while (is_running == TRUE) {
    process_input();
    update();
    render();
}
```

---

## Color Buffer

The color buffer is an array that holds the color value for every pixel on the screen. When `render()` runs, it reads from this array and draws the colors to the window.

How many entries do we need? One per pixel, so `window_width * window_height` total.

so here's where the type matters. Each color is stored as a 32-bit RGBA value, something like `0xFFFFFFFF` (fully opaque white). You might think "just use `int`," but the problem is that `int` size is not guaranteed. On most modern machines it's 4 bytes (32 bits), but C doesn't really promise us that, and we need exactly 32 bits here. So we use `uint32_t`.

The `u` means unsigned (no negative nos ), `int` is integer, `32` is the bit width, and `_t` is just a naming convention from `<stdint.h>` for fixed-width types. You'll also see `uint8_t` (8 bits, 0-255, good for individual channel values) and `uint64_t` (64 bits) in other contexts.

The RGBA layout stores red, green, blue, and alpha (opacity) each as 8 bits, packed into the 32-bit value. So `0xFF0000FF` is fully opaque red, for example.

```c
uint32_t* color_buffer = NULL;

color_buffer = (uint32_t*)malloc(sizeof(uint32_t) * window_width * window_height);
```

We initialize it as `NULL` first and then `malloc` the memory we actually need.This is objectively a better way to do things so we dont end up with things we dont want in our code , so always intialize with null. `sizeof(uint32_t)` is 4 bytes, multiplied by the total pixel count gives us the full size of the allocation.

---

## 3D and Vectors

There are two types of quantities in physics and math and they are : scalars and vectors. A scalar is just a number just a single magnitude. Mass, temperature, time whereas a vector has both magnitude and direction. Velocity, acceleration, force.

In graphics, almost everything is a vector for most parts 

 2D vector is just a point or direction in 2D space, with an `x` and a `y` component:

```c
typedef struct {
    float x;
    float y;
} Vec2_t;
```

 3D vector adds a `z` component for depth:

```c
typedef struct {
    float x;
    float y;
    float z;
} Vec3_t;
```

with tehse we can do basic operations like 

```c
static inline float vec2_len(Vec2_t v) {
    return sqrtf(v.x * v.x + v.y * v.y);
}

static inline Vec2_t vec2_add(Vec2_t a, Vec2_t b) {
    Vec2_t result = {
        .x = a.x + b.x,
        .y = a.y + b.y
    };
    return result;
}

static inline Vec2_t vec2_sub(Vec2_t a, Vec2_t b) {
    Vec2_t result = {
        .x = a.x - b.x,
        .y = a.y - b.y
    };
    return result;
}
```

---

## Projections

 We have 3D objects in our scene, but our screen is 2D. We need to "project" those 3D points down to 2D coordinates. There are a few ways to do this.

**Orthographic projection** just yoinks the Z axis and throws it away

```c
Vec2_t orthographic_project(Vec3_t point) {
    Vec2_t projected_point = {
        .x = point.x,
        .y = point.y
    };
    return projected_point;
}
```

This works for certain use cases (like 2D games or CAD tools) but it has no sense of depth. A cube drawn orthographically looks flat. so stuff dosent get smaller or bigger depending on your pov

**Isometric projection** is a specific type of orthographic projection where the three coordinate axes are equally foreshortened and the angles between them are all 120 degrees. It's not true perspective either, it just gives the illusion of 3D from a fixed angle. Just think of old strategy games like Age of Empires or classic Zelda.

For a rasterizer that actually looks 3D, we want something known as **perspective projection**.

The idea behind perspective is quite simple: things that are farther away should appear smaller and We achieve this with the perspective divide:

```
P'x = Px / Pz
P'y = Py / Pz
```

You divide the x and y coordinates by the z coordinate (the depth). When something is far away, Pz is large, so the projected x and y shrink. When it's close, Pz is small, so x and y are closer to their original values. That's what creates the sense of depth.

---

## Coordinate Systems

X and Y are easy to assume for the most parts , but the case of Z is not that simple , Z can face us that is outwards from the screen or go in an entirely opposite direction which is into the screen

There are two conventions: **left-handed** and **right-handed**.

In a **right-handed coordinate system**, if you point your right hand's fingers along X and curl them toward Y, your thumb points in the direction of Z. This means Z points out of the screen toward the viewer , for eg OpenGL uses right-handed coordinates.

In a **left-handed coordinate system**, the same trick with your left hand means Z points into the screen, away from the viewer and DirectX uses left-handed coordinates.

---

## Triangles and Meshes

so we know what vectors are. now we need to actually use them to represent 3D objects.

A **mesh** is bascially a collection of triangles arranged in a 3D space to give the impression of a solid object. Why triangles specifically thought right could be anything ? Because three points always define a flat plane. Four points might not be coplanar (think of somehting like a wobbly table), but three always are. 

Each triangle comes with 3 vertices. A single face of a cube is just 2 triangles sharing an edge if you think about it , giving you 4 unique vertices total. A full cube is 6 faces, so 12 triangles, 8 unique vertices. and thats all a mesh is collection of these triangles trying to form a particular object or shape or anything.

---

## Drawing Lines

Before we can draw these triangles we yapped about , we need to be able to draw lines. The math behind it is quite straightforward: a line is just `y = mx + b`,which we all learned in school ,  where `m` is the slope. Slope is just `delta_y / delta_x`

and then we come into rasterizing these lines here there are 2 common algorithms for actually rasterizing a line onto pixel grid: **DDA** and **Bresenham's line algorithm**. DDA is simpler to understand but Bresenham is faster , i used DDA in mine , well actually there were a lot of perfomnace tradeoffs but this is for learning and its done on cpu not gpu we are not trying to make it fast anyway because if we were we would be actulaly writing a 3d renderer in gpu with proper graphics apis. 

### DDA (Digital Differential Analyzer)

DDA works by figuring out which axis has the longer span (x or y), then stepping along that axis one pixel at a time and incrementing the other axis proportionally.

```c
void draw_line_dda(int x0, int y0, int x1, int y1, uint32_t color) {
    int delta_x = (x1 - x0);
    int delta_y = (y1 - y0);

    int side_length = abs(delta_x) >= abs(delta_y) ? abs(delta_x) : abs(delta_y);

    float x_inc = delta_x / (float)side_length;
    float y_inc = delta_y / (float)side_length;

    float current_x = x0;
    float current_y = y0;

    for (int i = 0; i <= side_length; i++) {
        draw_pixel(round(current_x), round(current_y), color);
        current_x += x_inc;
        current_y += y_inc;
    }
}
```

The problem with DDA is that it uses floating point arithmetic the whole way through. Every step involves a float increment and a `round()` call. That adds up if we use models that use a lot of lines or need lot of lines so something other than low poly gets stuck or hangs or whatver.

### Bresenham's Line Algorithm


The idea here is to track an "error" term that accumulates as you step along x. When the error crosses a threshold, you increment y and adjust the error back down. 
```c
void draw_line_bresenham(int x0, int y0, int x1, int y1, uint32_t color) {
    int dx = abs(x1 - x0);
    int dy = abs(y1 - y0);

    int sx = (x0 < x1) ? 1 : -1;
    int sy = (y0 < y1) ? 1 : -1;

    int err = dx - dy;

    while (1) {
        draw_pixel(x0, y0, color);

        if (x0 == x1 && y0 == y1) break;

        int e2 = 2 * err;

        if (e2 > -dy) {
            err -= dy;
            x0 += sx;
        }

        if (e2 < dx) {
            err += dx;
            y0 += sy;
        }
    }
}
```

`sx` and `sy` handle the direction of travel (positive or negative), so this version works for lines going in any direction, all 8 octants. The `err` term is just tracking how far off we are from the "true" line position. 

This is likely part 1 of this and it wont make sense to anyone except me who will read this in 2 weeks 
here is the code if anyone ever reads this 

[gevurah](https://github.com/loveucifer/gevurah)
