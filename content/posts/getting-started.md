---
title: "Getting Started with Box3D"
date: 2025-02-24
draft: false
---

Okay, so I've decided to embark on what's probably going to be the most challenging engineering project I've undertaken so far: I'm going to fork Box2D, the popular 2D physics engine, and extend it to
support 3D physics.

Why? I've been a physics enthusiast since my college days, and I've dabbled in game development on and off for the past decade. I've always wanted to combine these passions, and creating my own
physics engine is the obvious next step.

I know, this is a huge undertaking. It's not something I expect to finish in a weekend. But I'm excited about the learning opportunity, and I figured I'd
document my journey here, starting with the very first steps.

## Why Box2D?

Box2D is a great starting point for several reasons

#### Mature and Robust

Box2D has been around for years, it's well-tested, and it's used in countless games like Angry Birds. This means the core 2D logic is solid.
Box2D is open sourced, so I can dig into the source code, and modify it to my heart's content (thanks to the MIT license).

#### Syntax

I'm comfortable with C and C++, the languages the original Box2D is written in, which is perfect for performance-critical physics calculations. I'm going to go with C++17.

#### 2D Foundation

Starting with a working 2D engine gives me a solid base. I can gradually add 3D features, rather than starting completely from scratch.

#### Understanding the Terrain

Before writing a single line of 3D code, I knew I had to get comfortable with the existing Box2D codebase. I'm talking *really* comfortable. This isn't a "skim the docs and jump in" kind of project.
I spent a good chunk of time just reading the code. Here's what I focused on.

* **The b2World Class:** This is the heart of the simulation, the container for everything. I needed to understand how it manages the time step, collision detection, and constraint solving.

* **b2Body, b2Fixture, b2Joint:** These are the core building blocks. Bodies are the rigid objects, fixtures define their shapes and collision properties, and joints connect them. I traced how these
  classes interact.

* **b2Math:** This is where all the 2D vector and matrix operations live. I needed to refresh my memory on linear algebra (dot products, cross products, etc.) and see how Box2D implements them.

* **Collision Detection (b2Collide*.cpp):** This is where the magic happens. I studied the algorithms used for detecting collisions between different shapes (especially the Separating Axis Theorem –
  SAT).

* **Constraint Solver (b2ContactSolver.cpp):** This is probably the most complex part. It's responsible for making sure objects don't interpenetrate and that joints behave correctly. I spent a lot of
  time trying to wrap my head around the sequential impulse method.

#### Laying the Groundwork (3D Math)

The first real coding step is to start replacing the 2D math with 3D math. This is where a good 3D math library comes in. I decided to go with GLM (OpenGL Mathematics). It's a header-only C++ library
that's designed to be similar to GLSL (the shader language), which I'm somewhat familiar with. Since GLM is header-only, I just need to make sure the GLM directory is in my compiler's include path.

I'm also going to need classes for:

* 3x3 Matrices `glm::mat3` For rotations.
* 4x4 Matrices `glm::mat4`: for full transformations.
* Quaternions `glm::quat`: For representing rotations efficiently and avoiding gimbal lock.
* Transform `Box3D::Transform` update: The transform class needs to handle 3D. Here's a basic structure:

```c++
namespace Box3D {
class Transform {
public:
    glm::vec3 position;
    glm::quat rotation;
    // ... (scale, methods for combining transforms, etc.)
};
}
```

I'm starting with the vector class, adding basic operations (addition, subtraction, dot product, cross product, normalization), and then gradually replacing `b2Vec2` throughout the codebase. I'm
expecting a lot of compiler errors at this stage, but that's okay. It's part of the process.

My project structure includes a Box3D directory to house all the new 3D code, with subfolders for math, shapes, collision, etc., to keep things organized.

```c++
// My new 3D vector class (using GLM)

#include <glm/glm.hpp>
#include <glm/ext.hpp> // For potentially useful extensions like glm::to_string

// Use a namespace for Box3D to avoid naming conflicts
namespace Box3D {

// Use 'using' instead of typedef for better type aliasing
using vec3 = glm::vec3;

} // namespace Box3D

// Example Usage (can be in a separate .cpp file or a test file)
#include <iostream> // For printing

int main() {
    Box3D::vec3 a(1.0f, 2.0f, 3.0f);
    Box3D::vec3 b(4.0f, 5.0f, 6.0f);
    Box3D::vec3 c = a + b; // Vector addition

    std::cout << "c = " << glm::to_string(c) << std::endl; // Output using GLM's string conversion

    // Demonstrating dot product:
    float dot_product = glm::dot(a, b);
    std::cout << "Dot product of a and b: " << dot_product << std::endl;

     // Demonstrating cross product:
    Box3D::vec3 cross_product = glm::cross(a, b);
    std::cout << "Cross product of a and b: " << glm::to_string(cross_product) << std::endl;
    return 0;
}
```

#### Thinking in 3D (Shapes)

The plan of attack for 3D shapes is to extend the `b2Shape` class hierarchy. Here is the roadmap so far:

* `b2CircleShape` becomes `Box3D::SphereShape`.
* `b2PolygonShape` becomes `Box3D::ConvexHullShape`.
* `b2EdgeShape` becomes `Box3D::PlaneShape`
* Add `Box3D::BoxShape`

#### 3D Collision Detection

Implementing the Separating Axis Theorem (or GJK) in 3D is a significant hurdle.

* SAT (Separating Axis Theorem): Checks for collisions by projecting shapes onto axes. If projections don't overlap on *any* axis, the shapes are separated. Overlap on *all* axes means a collision.

* GJK (Gilbert-Johnson-Keerthi): An iterative alternative to SAT, often more efficient for complex shapes. It uses the Minkowski difference (all vector differences between shape points) and checks if a simplex (tetrahedron in 3D) within it contains the origin.

* Broad Phase vs. Narrow Phase: Collision detection uses two phases for efficiency.  *Broad phase* quickly eliminates non-colliding pairs (e.g., using AABBs or octrees). *Narrow phase* (SAT/GJK) performs precise checks on the remaining pairs.

#### Constraint Solver

Adapting the sequential impulse solver to 3D, especially the rotational constraints, is going to be complex.

#### Joints

Extending the joint types to 3D will require careful consideration of how they work in 3D space.

#### Testing and Visual Debugging

I'm planning to use Google Test as my testing framework. I'm going to need to write extensive unit tests to make sure the physics is functioning correctly, including individual functions (like vector
operations and matrix multiplications) and small simulations (integration tests) to test the interactions of multiple components.

I'll be aiming to follow a Test-Driven Development (TDD) approach where possible, writing tests before implementing the features. And visual debugging tools will be essential. I plan to create a
`Box3D::DebugDraw` class, similar to Box2D's `b2Draw`, that will allow me to render the simulation state in real-time. I'll be using the SDL2 library for window creation, input handling, and basic 3D rendering. 

This will enable me to visualize shapes, collision points, normals, joints, and other debugging information directly, making it much easier to understand what's happening in the simulation.

#### Conclusion

I'm fully expecting to hit roadblocks, get stuck, and have to rewrite code multiple times. But that's part of the learning process. I'll be documenting my progress (and struggles!) here, so stay tuned
for more updates.

If you're interested in following along, the code will be available on [GitHub](https://github.com/KelCodesStuff/Box3D). I'll be posting updates as I make progress.