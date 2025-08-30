---
layout: post
title: "Enforcing Continuity and AMR in Unreal Engine"
date: 2025-08-27
---

Good day,

Recently, I had been thinking about the construction of Adaptive Meshing Systems for R3HCS and wanted to describe and record my learning, understanding, and design for it.

The more I work with the Unreal Engine, the more I am learning nuances and advantages to one approach over the other. Today, I have been going over and researching the process by which continuity across mesh objects, specifically faces for hexahedral elements, is enforced during Adaptive Mesh Refinement (AMR) for Finite Element Method(FEM) based solutions. 

The primary issue is that when a coarse element shares a face with refined elements, that face and the other face that shares that space must be geometrically consistent, or else it will cause rendering issues, solver issues, and interpolation and surface smoothing issues.

Three of the main methods for enforcing continuity between coarse and refined faces in the mesh are the methods of Hanging Node Constraints (FEM style), Conforming Refinement (refine the neighbors, I believe it was called green refinement), and Render-Time Patch Tesselation. Enforcing continuity in one of these ways is necessary for rendering AMR for heat conduction, deformations, or physics based shading for example, as well as avoiding visible seams, cracks, or inconsistent normals between adjascent elements of different refinement levels.

My preference would be to keep unrefined neighboring elements coarse, so I will choose to use the first of those mentioned methods.

Lets think about a specific example. We have a mesh and somewhere in the mesh AMR was triggered based on field error and a piece of the mesh was refined. On the boundary of those refined elements, coarse and refined elements exist whose nodes do not necessary match. First, the coarse element's face has 4 nodes, but now the matching coarse neighbor with 4 nodes has been refined to 4 elements (on the boundary) where there are now 4 vertices per element on that shared face boundary. All 4 refined elements will create a set of 9 unique vertices, 4 of which is shared between it and the coarse face, thus, the coarse face needs vertices added in the same way the refined elements were created (using midpoints).

Topological refinement where we split the coarse block face to match the refined face should be best for physics simulation continuity. 

<b>Step 1</b>
So, the first step is to construct a method to identify and reference faces on elements for detecting if a face is shared between elements. I have been using hexahedral elements for simplicity.

<b>A discussion of TMap, unordered_maps in C++ STL, and Performance</b>

One of the key data structures I have been utilizing when working on development is the TMap. TMap is an Unreal Engine(UE)-specific version of the same hash table structure in the C++ Standard Template Library(STL) template that gives "unordered_map" its average O(1) complexity. It achieves this complexity with worst case scenario giving O(N) when there are poor hashes and hash collisions. It is one of the core container classes provided by UE for C++ development, alongside TArray and TSet.

TMap functions as a key-value pair container, similar to a hash map or dictionary in other programming languages. It leverages hashing for efficient storage and retrieval of data, making it a fundamental tool for managing various types of information within Unreal Engine projects.

<b>The Performance of TMap and unordered_maps in Unreal Engine C++</b>

- unordered_map TMap in UE
The complexity of lookup for std::map is O(log N) (logarithmic in the size of the container).

Per Paragraph 23.4.4.3/4 of the C++11 Standard on std::map::operator []:

Complexity: logarithmic.

The complexity of lookup for std::unordered_map is O(1) (constant) in the average case, and O(N) (linear) in the worst case.

Per Paragraph 23.5.4.3/4 of the C++11 Standard on std::unordered_map::operator []

Complexity: Average case O(1), worst case O(size()).

- TMap

TMap is effectively a reimplementation of unordered_map in the UE ecosystem. TMap is designed to work efficiently within the Unreal Engine, including with interactions with the reflection system and it will often utiliize contiguous memory blocks for better cache performance compared to potential linked-list-based bucket implementations in some std::unordered_map versions.

We can verify this by inspection of the source code which can reveal its design principles and in what ways hash table was implemented to achieve the same kind of performance.Additionally, the official UE Documentation as well as discussions with the community have revealed some of these details, but the former method of looking at the source code is the most authoritative source on the matter.

A quick example:
// Declare the TMap.
TMap<uint64, TArray<FIntPoint>> MyMap;

// 1. Add a key-value pair.
TArray<FIntPoint> NewPoints;
NewPoints.Add(FIntPoint(10, 20));
NewPoints.Add(FIntPoint(30, 40));
MyMap.Add(12345, NewPoints);

// The TMap now owns a copy of NewPoints.
// Changing NewPoints afterwards will not affect the map's contents.

// 2. Get a value by its key.
// The Find() method is safer than operator[] because it returns a pointer.
TArray<FIntPoint>* FoundPoints = MyMap.Find(12345);
if (FoundPoints)
{
    // You can now safely access and modify the array through the pointer.
    FoundPoints->Add(FIntPoint(50, 60)); // The TMap's internal copy is modified.
}

// 3. Iterate over the map.
for (const TPair<uint64, TArray<FIntPoint>>& Pair : MyMap)
{
    uint64 Key = Pair.Key;
    const TArray<FIntPoint>& Value = Pair.Value;

    UE_LOG(LogTemp, Log, TEXT("Key: %llu"), Key);
    for (const FIntPoint& Point : Value)
    {
        UE_LOG(LogTemp, Log, TEXT("  Point: (%d, %d)"), Point.X, Point.Y);
    }
}

// 4. Remove a key-value pair.
MyMap.Remove(12345);


To me, these kind of nuances are why it is important to use the tools UE has implemented over raw C++ alternatives. With respect to project development, interfacing, and performance, it is essential.

<b>Implementation</b>
Returning to the task at hand, the goal is to map each face (face hash) of each block in the mesh to all blocks that share that same face (face hash) along with which face (face index) this shared face has relative to the original face (face hash). So, the key will be the face hash and the value will be an array of block (block index) that are connected to that face (face hash) along with which face (face index owned by block index) is connected.

With that implemented, ... (to be continued)

See you in the next post,
<br>Cary