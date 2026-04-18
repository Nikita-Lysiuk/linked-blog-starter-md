---
type: concept
title: "Collisions"
created: 2026-04-18
updated: 2026-04-18
tags:
  - concept
  - collisions
  - physics
  - math
  - learning-campus
status: developing
complexity: intermediate
domain: game-dev-physics
aliases:
  - "Collision Detection"
  - "Collision Resolution"
related:
  - "[[ECS]]"
  - "[[concepts/_index]]"
  - "[[sources/Reference Library]]"
sources:
  - "[[sources/Reference Library]]"
---

# Collisions

**Domain:** Game Physics / Computational Geometry
**Complexity:** `intermediate` → `advanced` (depends on algorithm depth)
**Learning Campus Track:** [[wiki/meta/kanban-campus]] — "New Topics" → "In Depth"

---

## Definition

**Collision detection** is the computational problem of determining whether two or more objects in a simulation intersect, touch, or are within a defined proximity — and *when* this occurs. **Collision resolution** is the subsequent process of computing the appropriate response (separation, impulse, damage, trigger).

Two sub-problems:
1. **Broad phase** — quickly eliminate pairs that *cannot* be colliding (spatial hashing, BVH, sweep-and-prune)
2. **Narrow phase** — precisely test the remaining candidate pairs (AABB, Sphere, SAT, GJK)

---

## Core Algorithms

### 1. AABB — Axis-Aligned Bounding Box

**What:** Two boxes are colliding if and only if they overlap on all three axes simultaneously.
**Test:** For each axis X, Y, Z: `A.max > B.min && B.max > A.min`. If all three pass → collision.

```cpp
struct AABB {
    FVector Min, Max;
};

bool Intersects(const AABB& A, const AABB& B) {
    return (A.Max.X > B.Min.X && B.Max.X > A.Min.X) &&
           (A.Max.Y > B.Min.Y && B.Max.Y > A.Min.Y) &&
           (A.Max.Z > B.Min.Z && B.Max.Z > A.Min.Z);
}
```

**Pros:** Extremely cheap. Cache-friendly in broad phase.
**Cons:** Axis-aligned only — objects that rotate need AABB rebuilding each frame.

> Source: [[Reference Library]] — Book #3 (Ericson), Ch. 4, p. 77–78

---

### 2. Sphere-Sphere Collision

**What:** Two spheres collide if the distance between centers is less than the sum of their radii.
**Test:** `|A.center - B.center|² < (rA + rB)²` — use squared distance to avoid `sqrt`.

```cpp
struct Sphere {
    FVector Center;
    float Radius;
};

bool Intersects(const Sphere& A, const Sphere& B) {
    float RadiusSum = A.Radius + B.Radius;
    return FVector::DistSquared(A.Center, B.Center) < (RadiusSum * RadiusSum);
}
```

**Pros:** Single comparison. Best broad-phase primitive for round objects.
**Cons:** Poor fit for elongated shapes — excessive false positives.

> Source: [[Reference Library]] — Book #3 (Ericson), Ch. 4, p. 88

---

### 3. SAT — Separating Axis Theorem

**What:** Two convex shapes are NOT colliding if there exists *any* axis along which their projections do not overlap. Test all potential separating axes (face normals + cross products of edges).

**Key insight:** If you can find *one* gap, they don't collide. No gap on any axis → collision.

```cpp
// 2D conceptual implementation
bool SATIntersects(const ConvexPolygon& A, const ConvexPolygon& B) {
    // Test A's edge normals
    for (const FVector2D& Axis : A.GetEdgeNormals()) {
        auto [minA, maxA] = A.Project(Axis);
        auto [minB, maxB] = B.Project(Axis);
        if (maxA < minB || maxB < minA) return false; // Separating axis found
    }
    // Test B's edge normals
    for (const FVector2D& Axis : B.GetEdgeNormals()) {
        auto [minA, maxA] = A.Project(Axis);
        auto [minB, maxB] = B.Project(Axis);
        if (maxA < minB || maxB < minA) return false;
    }
    return true; // No separating axis found → collision
}
```

**Pros:** Works for any convex polygon/polyhedron. Produces penetration depth + collision normal for resolution.
**Cons:** 3D is more complex — must test cross products of edge pairs (up to 15 axes for OBB). Fails for concave shapes.

> Source: [[Reference Library]] — Book #3 (Ericson), Ch. 5; Paper P1 (dyn4j.org SAT tutorial)

---

### 4. GJK — Gilbert-Johnson-Keerthi Algorithm

**What:** Determines the minimum distance between two convex shapes using the Minkowski difference. Iteratively finds the simplex (point, line, triangle, tetrahedron) closest to the origin.

**Key insight:** Two shapes collide if and only if their Minkowski difference contains the origin.

**Use:** GJK finds *whether* shapes collide; EPA (Expanding Polytope Algorithm) finds *how much* (penetration depth + contact normal).

> Source: [[Reference Library]] — Paper P2 (Casey Muratori's GJK series)

> [!key-insight] When to use GJK
> GJK is the industry standard for precise convex collision (PhysX, Bullet, UE5's Chaos). Use AABB for broad phase, SAT for moderate complexity, GJK+EPA when you need accurate contact normals for character ragdolls, physics joints, and constraint solvers.

---

## Algorithm Comparison

| Algorithm | Shapes | 3D? | Cost | Output | Best Use |
|-----------|--------|-----|------|--------|----------|
| AABB | Boxes | Yes | O(1) | Boolean | Broad phase |
| Sphere | Spheres | Yes | O(1) | Boolean | Broad phase, explosions |
| SAT | Convex polygons | Yes (complex) | O(n axes) | Boolean + MTV | General convex shapes |
| GJK + EPA | Any convex | Yes | O(n iterations) | Distance + normal | Physics solver, ragdolls |

**MTV** = Minimum Translation Vector (how far to push apart to resolve penetration)

---

## Learning Path

**Prerequisite knowledge:**
- [ ] Vector dot product and cross product
- [ ] Projection of a vector onto an axis
- [ ] AABB data structure
- [ ] Basic rigid body concepts (position, velocity, mass)

**Recommended sequence:**
1. [ ] Master AABB (implement from scratch in C++)
2. [ ] Master Sphere collision (implement from scratch)
3. [ ] Understand projection — read [[Reference Library]] — Book #5 (Dunn & Parberry), Ch. 9
4. [ ] Implement SAT in 2D, then extend to 3D
5. [ ] Study Minkowski difference — watch Casey Muratori's GJK video series (Paper P2)
6. [ ] Implement GJK in C++
7. [ ] Study broad-phase: BVH, spatial hashing
8. [ ] Integration: plug into UE5's collision pipeline or a custom physics layer

---

## Project Idea: Collision Sandbox

**Goal:** Build a standalone C++ (UE5) project that visualizes each collision algorithm in real-time with debug drawing. No game logic — pure algorithmic exploration.

**Stack:** Unreal Engine 5 / C++

### Stage 1 — Math Setup
- Implement `FAABB`, `FSphere`, `FConvexPolygon` as plain C++ structs
- Write unit tests (UE5 Automation Framework) for each intersection function
- Debug draw: `DrawDebugBox`, `DrawDebugSphere` — red = colliding, green = clear

### Stage 2 — Resolve Logic
- Implement MTV extraction from SAT (push-apart vector)
- Implement basic velocity reflection for sphere-sphere bounce
- Add mass: `Impulse = (2 * relativeVelocity · normal) / (1/mA + 1/mB)`
- Debug draw: contact normal as arrow

### Stage 3 — Debug Visuals
- Toggle between algorithms at runtime (keyboard shortcut per shape type)
- Display: algorithm name, test result, penetration depth, contact normal in HUD
- Stress test: 100 AABB vs 100 AABB — measure ms/frame
- Optional: implement a BVH and visualize tree traversal

**Deliverable:** A reusable C++ collision library extracted from this sandbox, usable in Persival.

---

## Resources

| Resource | Type | Relevance | Link |
|----------|------|-----------|------|
| Real-Time Collision Detection — Ericson | Book #3 | AABB, Sphere, SAT, GJK — complete treatment | [[Reference Library]] |
| Physics for Game Developers — Bourg & Bywalec | Book #4 | Collision response, impulse resolution | [[Reference Library]] |
| 3D Math Primer — Dunn & Parberry | Book #5 | Vector/matrix prerequisites, geometric tests | [[Reference Library]] |
| dyn4j SAT tutorial | Paper P1 | Best SAT walkthrough with diagrams | [[Reference Library]] |
| Casey Muratori's GJK series | Paper P2 | Gold standard GJK explanation | [[Reference Library]] |
| Wikipedia: Collision Detection | P3 | Overview + spatial data structures | [[Reference Library]] |

---

## Contradictions / Caveats

> [!contradiction] AABB for Rotating Objects
> AABB must be rebuilt every frame when objects rotate. For fast-rotating objects, the AABB grows large and generates many false-positive broad-phase hits. Use OBB (Oriented Bounding Box) instead — but OBB requires SAT for the narrow phase test.

> [!gap] Continuous Collision Detection (CCD) not covered
> This page covers discrete collision (test at current frame). Fast-moving objects can tunnel through thin geometry between frames. CCD (sweep tests, time-of-impact) is a separate topic — file a note when studying high-speed projectiles.

## Related
- [[ECS]] — collision queries integrate with ECS via Physics/Collision components
- [[sources/Reference Library]] — full book/paper list with citations
- [[concepts/_index]]
