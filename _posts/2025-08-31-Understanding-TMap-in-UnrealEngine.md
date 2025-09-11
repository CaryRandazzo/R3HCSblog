---
layout: post
title: "Understanding TMap in Unreal Engine"
date: 2025-08-31
tags: [UnrealC++]
---

## A Brief Discussion of TMap, unordered_maps in C++ STL, and Performance

One of the key data structures I have been utilizing when working on development is the TMap. TMap is an Unreal Engine(UE)-specific version of the same hash table structure in the C++ Standard Template Library(STL) template that gives “unordered_map” its average O(1) complexity. It achieves this complexity with worst case scenario giving O(N) when there are poor hashes and hash collisions. It is one of the core container classes provided by UE for C++ development, alongside TArray and TSet.

TMap functions as a key-value pair container, similar to a hash map or dictionary in other programming languages. It leverages hashing for efficient storage and retrieval of data, making it a fundamental tool for managing various types of information within Unreal Engine projects.

## The Performance of TMap and unordered_maps in Unreal Engine C++

### unordered_map TMap in UE
The complexity of lookup for std::map is O(log N) (logarithmic in the size of the container).
Per Paragraph 23.4.4.3/4 of the C++11 Standard on std::map::operator []:

Complexity: logarithmic.

The complexity of lookup for std::unordered_map is O(1) (constant) in the average case, and O(N) (linear) in the worst case.

Per Paragraph 23.5.4.3/4 of the C++11 Standard on std::unordered_map::operator []

Complexity: Average case O(1), worst case O(size()).

### TMap

TMap is effectively a reimplementation of unordered_map in the UE ecosystem. TMap is designed to work efficiently within the Unreal Engine, including with interactions with the reflection system and it will often utiliize contiguous memory blocks for better cache performance compared to potential linked-list-based bucket implementations in some std::unordered_map versions.

We can verify this by inspection of the source code which can reveal its design principles and in what ways hash table was implemented to achieve the same kind of performance.Additionally, the official UE Documentation as well as discussions with the community have revealed some of these details, but the former method of looking at the source code is the most authoritative source on the matter.

### A Quick Example of TMap

```cpp
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
```

To me, these kind of nuances are why it is important to use the tools UE has implemented over raw C++ alternatives. With respect to project development, interfacing, and performance, it is essential.

See you in the next post,
<br>Cary