---
layout: post
title: Photogrammetry 3 ─ How to increase the quality and decrease resolution of your archaeological 3D models.
---

In our <a href="https://www.digitalarchns.com/masking/" target="_blank">last 
post</a> on photogrammetry, we covered how applying masks to photos can 
help in point cloud alignment. We also discovered that applying masks to Tie Points
increases the speed of producing an artifact model. Faster production can 
increase the quantity of 3D models you can produce in a given time, but what 
about quality?

I'm really excited to share a workflow that covers just that: **quality**. 

For many practitioners of photogrammetry, geometric accuracy will be 
the important measure 3D reconstruction quality. Here we will be exploring a 
workflow that can help to increase geometric accuracy. Once again in Metashape. 

Geometric accuracy is a measure of how closely a reconstruction represents the physical 
geometry of the object. As usual, accuracy is limited by precision a.k.a resolution.
Today we're only going to be simply visually evaluating geometric accuracy. 
Below we can see the product of todays post.

<p align="center">
	<img src="/images/Comparison_z10.jpg"> 
</p>
<p align="center">
<i><small>Visualization of a raw (before) and refined (after) mesh using multidirectional hillshade in ArcGIS with a 10x relief exaggeration.</i>
<i>Scaled using measurements collected by digital caliper.</small></i>
</p>

The refined mesh shows a noticable reduction in noise and increase in detail. The comparison 
becomes more interesting realizing that the raw mesh has 9.5 million triangular faces and the 
refined mesh only 722 thousand faces. Reducing the number of mesh faces equals dramatically reduced 
output file size at face value. Smaller file sizes present an opportunity for us to better 
communicate archaeological data through more accessible visualizations.
### Resolution vs. Quality

Bad puns aside, I often worried about how I would share my 3D models until uncovering 
the following workflow. The worry stems from the fact that until recently, the quality of 
a model produced in agisoft was highly correlated to resolution. If I created a 
highly accurate model I can easily share it. However, it might not be accessible 
as high resolution models bog down the majority of consumer computers, and mobile 
devices.

Ultimately, this is less than effective communication. So why bother?

My typical response is to slightly smooth, then decimate the mesh to my desired face-count 
for sharing the model. This blob of a representation is then given a high resolution 
image texture, and combined with materials and lighting effects in the sketchfab 3D viewer. 

As much as I have appreciated using this workflow to share something that people can 
interactively view. I've always been a bit anxious when sharing because so much 
detail is lost unless accesibility is comprimised. 

When decimating a model you lose important geometric detail, and there is no guarantee 
the software will filter out *only* noise. It seems like a lose-lose situation. Lowering 
the resolution **AND** quality of the model.

### <p style="text-align: center;"> "*Metashape's Photoconsistent Refine Mesh Tool!*"</p>

I've recently stumbled upon a tool that subtley came onto the scene in October 2017: **Refine Mesh**. 
In my experience using the tool, it does a great job at smoothing noise while retaining important geometric 
details. The process even adds geometric detail from the photos that was too small to be captured in the 
mesh, but can be identified within your photos.

According to Agisoft Metashape's user manual, for whom I've graciously fixed a typo: 

---
 #### <p style="text-align: center;"> "*Metashape allows to refine already reconstructed mesh with respect to camera photos. This is [an] iterative process that can further recover details on surface. For example it can recover bas-relief or ditch.*"</p>
---

So let's take a look at before and after results of using the tool. 

<p align="center">
	<img width="320" height="600" src="/images/9MilMesh_RAW.jpg"> 
	<img width="320" height="600" src="/images/Refinement1Mesh_RAW.jpg">
</p>
<p align="center">
<i><small><b>LEFT</b>: High resolution mesh after processing, no mesh refinement. <b>RIGHT</b>: High resolution mesh after 1 iteration using *Refine Mesh*.</small></i>
</p>

The results above aren't incredibly exciting. The tool definitely has potential, 
but the results led me to believe that I would need to take steps to reduce the reflectivity of 
my object because the noise is still a prominent feature in the mesh. You can likely see why I wasn't 
incredibly interested in the *Refine Mesh* tool when it was originally released in 2017.

I recently decided to revisit *Refine Mesh* as a tool to help 
retain geometric features as part of a slow, iterative process lowering 3D model 
resolution. The goal was to create a low resolution 3D model preserving more 
geometric detail than the refine mesh tool. To my surprise, I actually ended up 
with a low resolution output that was higher quality than my original high resolution 
input. You can see the difference below.

<p align="center">
	<img width="320" height="600" src="/images/9MilMesh_RAW.jpg"> 
	<img width="320" height="600" src="/images/Refinement8Mesh25k_RAW.jpg">
</p>
<p align="center">
<i><small><b>LEFT</b>: High resolution mesh (9.5 million faces) after processing, no mesh refinement. <b>RIGHT</b>: Low resolution mesh (25 thousand faces) after integrating <i>Refine Mesh</i> into this workflow.</small></i>
</p>

In the images above, it is apparent that a smoother low-resolution model still retaining 
detail, such as large flake scars, is produced by this workflow. I do still have issues with 
the result and its rounded edges, but it definitely does a great job at improving on the 
original in both accesibility and quality. 

I noticed there seems to be a 'sweetspot' for how many times you apply this 
workflow. The workflow was repeated on the model above 8 times before decimating to the 
25 thousand faces needed for accessible viewing. In the case of this object, it 
was 5 iterations that produced the best result. Check it out below.

<p align="center">
	<img width="320" height="600" src="/images/Refinement5Mesh_RAW.jpg">
</p>
<p align="center">
<i><small>Mesh after 5 iterations of the workflow integrating <i>Refine Mesh</i>.</small></i>
</p>

The above mesh is definitely superior to the 25 thousand face mesh shown prior to this, 
and is definitely a huge improvement on the original output. This is due in part to 
the higher resolution of the mesh.

### My mesh refinement workflow

So I'm going to assume that you already have a mesh to work from. As I 
mentioned above, the workflow is an iterative process that will bring out 
details **and** reduce the resolution of your mesh. The base process has 
three steps:

* Smooth Mesh 
* Decimate Mesh
* Refine Mesh

These three steps will allow you to pull out details in your model and reduce noise. 
At the end of the process, you will be able to decimate your mesh to an easily 
accessible size while increasing the quality of the model.

### Smoothing and decimating the mesh

The first step is to slightly smooth the model.

A somewhat subjective description. But I mean it to be. Actually, all of the following 
steps are subjective. 

What smoothing in this workflow is intended to do is to smooth some noise 
to help  *Refine Mesh* produce a more accurate surface when decimating. Smoothing 
the mesh will help emphasize features larger than the ~0.2mm diameter noise present 
on the raw, high-resolution mesh shown above. You'll likely want to **save your project** or 
duplicate the 3D model right before running *Smooth Mesh*. I frequently over-smooth the mesh and have to 
revisit a previous version to apply a smaller smoothing **strength** that only removes the noise. 

### Refining the mesh

After running *Smooth Mesh*, the next step is running *Refine Mesh*. 

The input parameters in *Refine Mesh* are Quality, Iterations, and Smoothness. 

<p align="center">
	<img src="/images/RefineMesh.png">
</p>
<p align="center">
<i><small>Refine Mesh dialog box.</small></i>
</p>

Quality refers to the mesh refinement quality, and I reccomend using the highest setting possible
for the first few steps in this process. Though it doesn't directly relate to mesh resolution, the number of 
mesh faces in the output are correlated with the quality setting chosen.

Iterations refers to the number of times the tool repeats the process on the object. Unfortunately, 
The actual underlying process isn't explained by Agisoft's 
<a href="https://www.agisoft.com/pdf/metashape-pro_1_5_en.pdf" target="blank">User Manual</a>, but 
I'll hazarded a vague guess. I think the algorithm may incorporate depth maps, colour maps, 
and edge maps from the highest quality photos to identify features to be excluded from multiple iterations 
of localized smoothing. The same result could be achived in plenty of other ways, with much more 
sophisticated algorithms being applied.

Finally, the Smoothing parameter relates to the weight of the smoothing filter applied. Frustratingly, the 
Agisoft User Manual documents what effect changing the values of the parameter has on the output, but not 
what the smoothign factor is applied to. It could relate to the size or weight of a localized smoothing filter, 
the weight of a global smoothing filter, or somethign completely unrelated. Agisoft just doesn't tell us. I have 
noticed that decreasing the smoothing value between successive iterations of *Refine Mesh* seems to work well to 
produce better results.

### Iterations and choosing when to stop

Choosing where to stop depends on what your outcomes are. Assuming you've followed along this far, 
you will have realized that the objective is accesibility of the models, or finding a balance between 
quality and the obvious loss of geometric accuracy when reducing the face count of a 3D model from 9.5 million to 
25k.

<p align="center">
	<img width="320" height="600" src="/images/mesh-gif.gif">
</p>
<p align="center">
<i><small>Visualizing outputs of the workflow.</small></i>
</p>

For our purposes, subjectivity is key, unless you want to make some measurements with a caliper and really find 
the optimal parameters for accuracy retention. I didn't take this route here because my objective 
was to illustrate this technique. 

### Recap

So to recap, the general workflow is decimate, smooth, and refine, saving your project or 
duplicating the mesh at every step. In each succesive iteration of this workflow you can gradually 
reduce the parameter values. Also, smoothing can be dropped from the workflow if there is no noise 
present in the mesh. You won't end up with a perfectly accurate 3D representation of the object, but 
you will get a high quality mesh with a lower resolution for sharing. 

However, refine mesh can also be used to simply refine a mesh without first decimating and smoothing. The 
result is a higher quality mesh of higher, if not only slightly higher resolution than the original mesh. For 
GIS products, a single use of the *Refine Mesh* tool might be able to retain geometry while smoothing noise. 

Actually I tested this quickly on a model I produced with a set of aerial photos captured by a Phantom 3 Pro. 
Shown is an orthophoto with raw and refined digital surface models (DSMs) of tire tracks in the fine dirt of a baseball 
diamond. These tracks barely show in the raw DSM, but after one iteration of Refine Mesh at Ultra high quality, the tire 
tracks are well defined. The noise on the smooth surface stemming from a lack of highly accurate location information is 
also reduced. I haven't done more field testing but I was impressed with this quick experiment.

<p align="center">
	<img src="/images/ballfield.jpg"></p>
<p align="center">
<small><b>Left</b>: Orthophoto. <b>Centre</b>: Raw DSM. <b>Right</b>: DSM after running <i>Refine Mesh</i></small>
</p>

Below is the model shown in the examples throughout this post. It is a re-processed model of the 
<a href="https://skfb.ly/6PHrD" target="blank"> Georgetown Flint Projectile Point</a> model 
shown in the previous blog post on photogrammetry. 


<div class="sketchfab-embed-wrapper" align="center">
    <iframe title="A 3D model" width="640" height="480" src="https://sketchfab.com/models/97f07017e61a4d7a839c55b9af18e147/embed?transparent=1" frameborder="0" allow="autoplay; fullscreen; vr" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

<p style="font-size: 13px; font-weight: normal; margin: 5px; color: #4A4A4A;" align="center">
    <a href="https://sketchfab.com/3d-models/georgetown-flint-stemmed-point-refined-mesh-97f07017e61a4d7a839c55b9af18e147?utm_medium=embed&utm_source=website&utm_campaign=share-popup" target="_blank" style="font-weight: bold; color: #1CAAD9;">Georgetown Flint Stemmed Point - Refined Mesh</a>
    by <a href="https://sketchfab.com/weatherbeewesley?utm_medium=embed&utm_source=website&utm_campaign=share-popup" target="_blank" style="font-weight: bold; color: #1CAAD9;">weatherbeewesley</a>
    on <a href="https://sketchfab.com?utm_medium=embed&utm_source=website&utm_campaign=share-popup" target="_blank" style="font-weight: bold; color: #1CAAD9;">Sketchfab</a>
</p>
</div>

Next week I'll be moving away from photogrammetry to introduce a method I've been working on 
in applying <a href="https://opencv.org/" target="_blank">openCV</a> 
in <a href="https://www.python.org/" target="_blank">Python</a> to artifact analyses. 
I hope that this post helped spawn some ideas about how accessibility of 3D models can be leveraged 
to better communicate archaeology. If you have any concepts that you would like to see 
covered in the future, please comment on the social media post that directed you to this blog 
and I will try to implement them in future posts! 

*Wesley Weatherbee*




