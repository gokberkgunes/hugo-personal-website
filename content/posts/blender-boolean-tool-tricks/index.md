---
title: "Blender Boolean Tool Tricks"
date: 2023-02-14T18:33:19+03:00
authors: ["Gokberk Gunes", ]
tags: [Blender, Mesh]
header_image:
draft: false
---

Here are two tricks of Blender's boolean modifier tool. These are handy if you
are creating a bounding box around inlet and outlet for computational fluid
dynamics simulations.

## Union: Alter Sides of the Main Object
This video will present when producing the union of two objects how to reverse
direction of the the main object. This could be only the case when boolean
operator is used with a plane.

{{< video name=union-flip-normals.webm >}}

## Union: Keep Objects Selectable Afterwards

Here how to properly get union of two objects without loosing the information
of past objects is demonstrated. Basically, this means that we can select,
seperate and alter the union seperately.


{{< video name=union-select-after.webm >}}
