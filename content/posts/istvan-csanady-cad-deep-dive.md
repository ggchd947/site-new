+++
title = "István Csanády CAD Deep Dive"
date = 2024-07-20T23:12:11-07:00
+++
{{< x user="istvan_csanady" id="1804582455116214616" >}}

## Why is your CAD slow?
So why is your CAD slow? Because it’s not using all your CPU cores, right?

No.

Let me debunk the “CAD is slow because it’s not multi-core” myth and explain how to build an exceptionally, mind-blowingly, crazy fast CAD.

So what are the typical issues that make your CAD slow? There are two main reasons:

1. Wrong assumptions made by the software engineer.
2. Computational geometry is inherently difficult, and computers struggle with parametric surfaces and curves.

Let’s elaborate on these.

### 1. Wrong Assumptions

Software engineer thinks: “Oh, this list will never contain more than a few hundred items; it’s perfectly fine if I use this O(N^2) algorithm.”

User does: “Let me just use this feature in a completely unexpected way and fill up that list with 4,500 items.”

CAD does: Takes 17 seconds to respond to a click.

Solution: The software engineer replaces the algorithm with an O(log(N)) algorithm.

Another example:

Software engineer thinks: “It’s totally impossible that a few hundred bodies would need a lot of memory.”

User does: “Let me just load these super funky bodies with 98 super complex spline surfaces each and create a nice little pattern with 200 bodies.”

CAD does: Allocates 46 gigabytes of memory, swaps it out to background storage, and crashes the entire operating system.

Solution: The software engineer finds a more efficient way to tessellate funky spline surfaces and implements Level of Detail (LoD) and smarter memory management.

Of course, similar issues are typically the source of slowdowns in most software. The problem with CAD is that these issues are more common because users (rightfully) expect unlimited scalability, and geometry computations often produce some really extreme corner cases. The solution is to always assume that everything will be 100x larger than you expected, do everything incrementally, cache everything, be very smart about memory management, and expect the worst. There is no such thing as too many octrees. This is the exact kind of behavior that most software companies would consider over-engineering, but you simply can't over-engineer a CAD system.

### 2. Computational Geometry is Hard

Parametric surfaces and curves are extremely hard to process, and very hard math problems are very common in this domain. That means geometry kernels need to perform a lot of computational geometry magic that simply cannot be parallelized (not because we are not smart enough, but because that’s how math works) and can take a lot of time to finish in some corner cases. When the performance issue comes from the geometry kernel, it’s usually due to poorly handled corner cases, a strange combination of surfaces, unusual tolerances, or something similar. A fast kernel is fast because, over decades, all of these corner cases have been addressed one by one to ensure the kernel is almost always fast.

But let’s say you just know math better than the rest of the world. You figure out how to make everything 16x faster on 16 cores. How much faster is your CAD going to be? If you have an exceptionally well written CAD, then it’s ~3-4x total. Why? Because even in the most computationally heavy use cases, no more than 70% of the processing time is spent in the kernel; the rest is application-level code.

So what is the solution?

1. Use a really good kernel, or reinvent the representation, and replace b-rep with something better, like implicits.

2. Always expect the worst, assume very large data sets, and implement your algorithms accordingly.

How do I know this? Because we are working on making Shapr3D the fastest CAD in the world. We are already blazing fast, and in less than a year, by most benchmarks, we’ll become the fastest. Exciting times.

## History of CAD
https://www.shapr3d.com/blog/history-of-cad

## Why you shouldn't write another b-rep kernel
Every two years, the idea of creating a new CAD kernel resurfaces, but these teams always fail (at least since 1984) - even though it would be great if they succeeded. Here's how I would think about developing a "new" geometry kernel.

The main issue is that these teams approach the problem from the technology's perspective, rather than focusing on the customer's workflow.

B-rep is designed to implement workflows, and thanks to the maturity of current b-rep kernels (especially Parasolid), any customer workflow that can be solved using b-rep can be already handled by these kernels exceptionally well.

When I say exceptionally well, I mean pretty damn well. There's no potential for a 10x improvement here. Sure, you could make it a bit more robust or a little faster, but you won't achieve a 10x improvement in b-rep workflows due to the maturity of the existing kernels. The biggest issues with b-rep workflows today come come from applications that are built on the top of the kernels, and not from the kernels.

That being said, to figure out what the next geometry kernel should look like, it's essential to identify the workflows that are highly valuable but can't be handled well by b-rep.

It's also worth noting that b-rep has some excellent properties, such as parametric surfaces and topology. Topology is especially great because humans easily understand the concepts of faces and edges.

Developing a new kernel can't happen outside the existing ecosystem. Even if the new kernel uses a different representation, it must be somehow compatible with b-rep. Otherwise, adoption will be challenging. B-rep is the standard protocol for all workflows today.

If you want to solve all the b-rep workflows and even more to disrupt the b-rep world, you need something truly novel that resembles b-rep but isn't b-rep.

One of the biggest flaws to eliminate is the double bookkeeping intrinsic to b-rep. Since b-rep works with tolerances, the geometry and topology data are somewhat independent, requiring handling an endless amount of corner cases and inconsistencies that are the inevitable by-product of b-rep operations.

This double bookkeeping is at the root of many b-rep problems. You should look for a geometry representation that eliminates this issue. Implicits could be a great option, and [@nTopology](https://x.com/nTopology) has done excellent work pioneering this technology.

If we could somehow add topology to implicits, it could be a strong candidate for a next-generation kernel. You'd have infinite resolution, topology, the ability to run it directly on GPUs, real-time functionality, and extreme accuracy.

There are, of course, countless reasons to write another b-rep kernel: it's fun (I'd do it for fun!), legal considerations, etc. But for a real breakthrough, we need to focus on workflows and figure out how to build a better geometry tech stack to support more workflows.

Despite this, I'm rooting for everyone aiming to improve the world of computational geometry. However, I believe we should set more ambitious goals than just creating another b-rep kernel.

## CAD formats 101

There are a zillion 3D CAD formats, and understanding when to use which one can dramatically enhance your workflow. By understanding the basic concepts behind the different formats, you can avoid data loss and data translation issues.

There are three main categories of 3D CAD formats:

Neutral Formats: These include IGES, STEP, and in some industries, JT. These are industry-standard formats supported by every professional engineering software.
Kernel Formats: Every computational geometry kernel has its own formats. The most common are Parasolid (.x_t or .x_b) and ACIS (.sat).
Application Formats: Every application has its own format, such as SOLIDWORKS (.sldprt, .sldasm) and NX (.prt).
+1. Non-CAD Formats: Formats like STL, 3MF, and OBJ are mesh formats. These do not contain boundary representation data, only triangulation.

### Why Are There So Many Different Formats?

Neutral formats exist to provide a standard way of transferring information between b-rep based systems. These formats have undergone a long standardization process and are usually your second best option for moving geometry between two CAD systems. STEP (ISO 10303-21) is the most popular and best-supported neutral format. Generally, b-rep works fairly well with STEP files, but metadata, annotations, and everything beyond geometry are typically less well supported.

In some industries, JT (developed by Siemens, but now an ISO standard) is the most common neutral format. It typically has better support for metadata and PMI, but fewer applications support it. Fun fact: when Daimler switched from CATIA to NX, their legacy data was simply archived to JT, which solved the data management part of the transition according to them.

The biggest drawback of neutral formats is that they are a middle ground for b-rep data. Moving from one CAD system to another means translating data twice: CAD1 representation → neutral representation → CAD2 representation. Just like with natural languages, a direct German → English translation will perform better than a German → Dutch → English translation.

Kernel formats are the native representation of b-rep data in a geometry kernel. When you want to transfer geometry (without metadata) from one CAD to another, often the kernel format is the best option if both CAD systems are running on the same kernel. For example, moving geometry between NX and Shapr3D is best done with the Parasolid format since both use the same geometry engine. If you import an NX file directly to Shapr3D, we’ll read out the embedded Parasolid geometry from the NX file and also parse connected metadata, but that’s a bit slower as we need to interpret the NX format. When using kernel formats for geometry transfer between two CAD systems running on the same kernel, compatibility is basically guaranteed, and you won’t run into any issues, though most metadata will be lost.

Application-specific formats contain all the information created in the application: feature tree, metadata, geometry, etc. These formats are usually proprietary and closed. Supporting these formats means reverse engineering them. The best data translator components can give direct access to the kernel formats embedded in the application-specific formats, while some other data translators will always convert the data to an intermediary format. Application-specific formats might perform better if the metadata is important, but if you only need the geometry, you are better off with a kernel format (if the two applications are running on the same kernel) or with a neutral format (primarily STEP).

## CAD data exchange
Why is CAD data translation so hard, and why will your CAD mess up your geometry when you are moving data from one CAD to another?

A deep dive on CAD data exchange... What happens when you export a file and import it to another CAD system? Why is it that one format might work when the other format doesn’t for the same geometry?

Welcome to the world of data translation.

So what is data translation? Why does it matter? Why is it hard? Why is it so fragile?

I wrote about different CAD file formats before. TLDR; a CAD format can contain metadata, a feature tree, geometry (boundary representation), and other application-specific information. Feature tree translation (transferring the design history from one CAD to another) is virtually impossible to implement in a robust way, although there are products that do this - but I would not rely on this in a production environment. So data translation typically means transferring:

1. Geometry defined as boundary representation
2. Metadata
3. Other application-specific data

Translating application-specific data and metadata is typically not very different from translating any other kind of data between two applications. There may or may not be a 1-1 mapping between the two systems, and the data translator will try to do its best to create a good mapping between the two. For example, one CAD system might support layers, the other CAD system doesn’t support layers but supports folders. In this case, it’s a quite reasonable decision to map layers into folders if the typical use cases of folders in CAD2 and layers in CAD1 are similar.

The real tricky part is translating geometry. And again, just like with many other things in CAD, the problem is boundary representation.

Boundary representation defines geometry with its boundaries as parametric curves and surfaces. There are two fundamental problems with translating boundary representation data:

1. Topology and geometry are represented separately in b-rep (which is pretty much the source of all evil in b-rep, this would be worth a separate post itself). Topology (what is connected with what) is defined by the CAD system’s geometry kernel based on the geometry and tolerances. B-rep as a mathematical concept does not define how to manage tolerances, it depends on the kernel. For example, CAD1 is running on Kernel1, CAD2 is running on Kernel2. Kernel1 uses 10^-5 meters as a tolerance for equality, Kernel2 uses 10^-6 meters. CAD1 creates a cube, where the end points of the three lines in a corner are 0.5 * 10^-5 meters away from each other, which in Kernel1 means that they are at the same point. When we export this cube to CAD2, if we just directly map the Kernel1 geometry to Kernel2 geometry, our solid cube ends up being a set of disconnected faces according to the geometry, but the topology still says that it’s a solid body. This will force Kernel2 to do all sorts of magic (called geometry healing) to make topology and geometry consistent. It can either blow up our cube and turn it into 6 separate faces, or it can try to manipulate the geometry to turn it into a solid object in the world of Kernel2. Either way, we will end up with different geometry, which can cause all sorts of manufacturing errors. Now, this is an extremely simple example, just imagine what happens when there are 6 spline surfaces connecting at a single vertex. A large part of writing data translators is about finding workarounds for these kinds of problems.

2. B-rep as a mathematical concept defines that the geometry is defined by parametric curves and surfaces, but it doesn’t limit what kind of curves and surfaces those could be. A completely generic implementation could decide that every curve and surface will be defined as NURBS at the cost of a huge performance, accuracy and memory penalty. Hence high-end kernels have many different surface and curve representations. For example, surfaces can be planes, cylinders, splines, conical surfaces, and in some high-end kernels, like Parasolid, even meshes. While there is a significant overlap between the types of surfaces and curves that different kernels can represent, there are always exceptions, and some neutral CAD formats limit the type of surfaces and curves further. A data translator needs to find a way to map these missing surface and curve types to each other, and often the only way is to fit a NURBS surface or curve on the geometry as a common representation. However, fitting always comes with inaccuracies and other unintended manufacturing issues.

And the worst part? All of these fixes and workarounds often go without any warnings, the CAD system just silently messes up the geometry. That being said: the best way to move data between two CAD systems is to avoid data translation. If you want to pick the right format, read my other thread about picking the right CAD format for data exchange.

## What is boundary representation?
Boundary representation is the democracy of 3D CAD: it’s the worst form of geometry representation except for all those other forms that have been tried from time to time.

### So what is b-rep? Why is it so great and so terrible at the same time?

B-rep is the mathematical foundation of how CAD systems represent geometry. B-rep represents shapes as parametric curves and surfaces that are connected to each other.

Unfortunately, parametric surfaces [s(u, v)=…] don’t necessarily have boundaries and don’t have a way to represent holes, like a plane (infinite in the u and v parameters) or a cylindrical surface (periodic in the v parameter, infinite in the u parameter) or a sphere (periodic in both parameters). B-rep solves this by adding boundaries and holes defined as curves laying on the surfaces. A face is a set of curves (that define the boundaries and the holes of the surface) and a single surface (on which the curves lay), and an edge is a curve segment of the boundary of the face (a trimmed curve, laying on the surface as part of the outer boundary or the inner hole).

Example 1: A finite cylindrical face is a cylindrical surface with two circles at the top and bottom of the cylinder, laying perfectly on the cylindrical surface.

Example 2: A square with a circular hole in the middle is a plane, bounded by four lines laying on the plane, and a circle laying on the plane in the middle of the square.

In these cases, all the edges (the boundaries of the faces) are so-called open edges because they are not connected to other faces. In other words, we have open sheet bodies, not solids. To create a solid body, we often need to connect multiple faces unless we have a closed surface, like a torus or a sphere. When two faces are connected, an edge is the intersection of the two surfaces.

### B-rep has some excellent properties

The parametric surfaces and curves are extremely accurate and can be used for manufacturing high-precision parts.
The concept of faces and edges is natural and easily understood by humans, providing not only topology and geometry but also an interaction model to select and manipulate geometry. Operations like “make this edge rounded” or “move this face by 2mm” align with how humans think.

And b-rep has some… absolutely terrible properties that are intrinsic to the representation and not possible to overcome:

Nothing in this representation guarantees that the topology and the geometry will be consistent. A few examples:

I can create a square with a circular hole where the curve of the hole overlaps with the boundary of the square, creating faulty geometry.

I can create a face structure where one of the four lines of a square is missing, so the boundary of the square is invalid.

I can create a face where the boundary of a hole or the outer boundary of the face is self-intersecting.

I can create a body that consists of two cubes connected only through one shared edge (top view: 🔷🔷), creating a zero-thickness region (non-manifold geometry) that cannot exist in the physical world.

Tolerances. Since b-rep works with floating-point numbers and numerical algorithms, equality is defined with tolerances: two points, curves, and surfaces are the same if they are closer than a small number (typically 10^-6 meter or less). This means that a b-rep kernel needs to maintain two separate data structures: the topology (what is connected to what) and the geometry. While the two data structures are independent, what they represent is not. Thus, the kernel continuously needs to do double bookkeeping of two complex data sets. Welcome to b-rep hell. 🔥😈🔥

So the issue is that there is nothing in the representation itself that guarantees that the geometric data makes sense. The role of a b-rep kernel (like Parasolid) is to ensure that every single operation creates consistent, high-quality data. These issues make writing a kernel a bit like self-driving cars: it’s fairly easy to get to 99%, but the remaining 1% takes forever. Why? Because to get to 99.99999% robustness, you need millions of users running into all the (combinations) of the above-mentioned use cases, reporting those issues, then implementing workarounds one by one for each of them. There is no easy or quick way to achieve that—yet boundary representation has become the industry standard for manufacturing in the last 50 years. Thus, creating a kernel for an alternative representation would come with insurmountable challenges.
