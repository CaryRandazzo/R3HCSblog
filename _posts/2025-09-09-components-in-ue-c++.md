---
layout: post
title: "Components in Unreal Engine C++"
date: 2025-09-09
tags:[UnrealC++]
---

Good day,

### Introduction
As I have said elsewhere, it is my belief that the component system in UE allows for a comfortable amount of encapsulation and modularity. Also, in the event that I want to use or expose pieces of a system to blueprints, it would allow for a smooth transition.

Here is the requirements for creating a Component in UE. I'll go through doing it via the editor as much as possible first, then transition to C++ as needed, and finally demonstrate settup up a new component entirely in C++.

### For the editor method:
- First, go to Tools (fyi, I am using Unreal Engine version 5.5.4 at the time of this post) and the Add C++ Class window should pop up. 

In the list, you can see actually there are two kinds of Components to choose from: Actor Component and Scene Component. 

According to the text, An Actor component is a reusable component that can be added to any actor while a Scene Component is a component that has a scene transform and can be attached to other Scene Components. 

A Scene Component maybe a slightly different process with some different nuances. For now, the remaining steps will assume we are interested in the Actor Component, so click on that and click Next. 

- Now, you can choose to make the Actor Component Class as either a Public Class or Private Class. 

The choice between the two primarily relates to the visibility and accessibility of the class across different modules within your Unreal Engine project. 

A public class is accessible from any other module within the Unreal Engine project, including other game modules, editor modules, or plugins. 

The header file (.h) for a public class is typically placed in the Public directory of its module (In the project source code directory). 

Public classes are suitable for components, actors, or other core functionalities that need to be exposed and used by various parts of your project or even external plugins. 

For example, if you create a custom actor that should be instantiable in the Unreal Editor, it needs to be a public class. 

The other choice, a Private Class is only accessible from within the module where it is defined. 

It cannot be directly accessed or used by other modules. The header file (.h) for a private class is typically placed in the Private directory of its module. 

Private classes are useful for internal helper classes, implementation details, or functionalities that are specific to a particular module and do not need to be exposed externally. 

This promotes encapsulation and helps in maintaining a cleaner API for your module. So let's choose public for this demo.

**Note**: It is possible to change a class's public/private designation after creation, but it requires manual file movement and regeneration of project files. 

To do that, you would close UE and any IDE open with the project, navigate to the project directory and manually move the header and/or source files between the public and private directories within your module's source folder, then you need to move to higher level directory where the .uproject usually lives and delete the .vs, Intermediate, Binaries, Saved, and DerivedDataCache folders from your project directory. 

After doing so, you then need to regenerate the project files by right clicking on the .uproject file and select "Generate Visual Studio project files", and finally you reopen your IDE and build the solution.

- Next, choose the name you want for the actor component such as MyActorComponent. 

The right drop down box allows you to choose the target module for your new class and should default to your current project runtime.
- Next, select the path to where the class files will live - the window shows based on the parent path where the header and source file paths go to respectively. 

I recommend leaving this at the default location unless you have reason to change it.
- Finally, we click the create class button and the files are created with the template setup according to our choices, and Visual Studio may open up following creation of this class.

These Component type classes are structures native to the engine and as such all the code related to the reflection system will be in it by default and should not be removed, else the component itself should become useless at best and at worst corrupt and crash the build. 

Removing a component from the system would likely follow a similar process to changing the class from public/private in the note above.

### For the C++ Method:

- Close both UE and your IDE
- Navigate to project folder under source
- Create MyActorComponent.h and MyActorComponent.cpp in your chosen access modifiers (private / public based on the directory the files go). You can change MyActorComponent to whatever you wish the name to be
-  Now, open the header file and cpp file in IDE and add the following code

```cpp
// MyActorComponent.h
void PlaceHolderCode();
```

```cpp
//MyActorComponent.cpp
// etc.
```
- and finally ...

### Example of when it might make sense to use an Actor Component:
- You have some logic you want to encapsulate, but should be accessible and exercised by an Actor that will/does exist in a scene
- You want modularity of the encapsulated logic such that you can move that logic onto other and/or multiple actors


### Outro

Perhaps you have simulated a heat conduction system and you want the user/player to be able to choose from or experience multiple heat source objects at will or simultaneously.

See you in the next post,
<br>Cary