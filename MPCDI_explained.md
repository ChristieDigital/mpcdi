The MPCDI spec is a VESA spec, and the full specification can be found online.

However, many have found the doc less than clear, so here is some additional notes; ie my understanding of the spec.

_These are NOT a substitute for reading and following the official VESA spec_

MPCDI the file, the zip, the mystery
----------

An MPCDI file is actually a zip file containing a XML file (typically called mpcdi.xml) and then warp and blend files, etc.

*HOWEVER* Mystique doesn't write out warp and blend info (we just send it directly to the projectors), so we only write out the XML file.  By convention, we use the extension `.mpcdi.xml`, so for the "foo" project, the file will be named `foo.mpcdi.xml`.  (The user can change the MPCDI filename and location from the Mystique UI.)


_The rest of this document is about the MPCDI XML format and the information that Mystique exports to that file_

Header
--------

As mentioned, it's XML:

    <?xml version="1.0" encoding="utf-8"?>
    <MPCDI profile="sl" geometry="2" color="1" date="2018-11-16 19:30:11" version="2.0">

It's MPCDI version 2.0.  For 3D, we use the "sl" (shader lamp) profile, which means we will export projector "poses" (position, orientation, etc) via the `<region>` node (see below).

Overall structure
--------

Next in the XML is a single `<display>` node, which describes the scene.  Within the display there _could be_ multiple `<buffer>` nodes (ie for projecting on completely separate surfaces/objects) but, really, it can all be done with a single buffer, so, by convention, Mystique just writes a single `<buffer>` and sets the buffer id to the name of the Mystique file of the project.

Within a buffer, there are multiple `<region>` nodes - basically one for each projector.

    <display>
        <buffer id="wolf_90_markerless.cal">
            <region id="Staging Projector_01" xResolution="1400" yResolution="1050" x="0" y="0" xsize="1" ysize="1">
            ...
            </region>
            <region...>
            ...
            </region>
            <region...>
            ...
            </region>
        </buffer>
    </display>

All the good info (ie per-projector info) is in the `<region>` nodes.

`<region>`
---------

A region looks like:

            <region id="Staging Projector_01" xResolution="1400" yResolution="1050" x="0" y="0" xsize="1" ysize="1">
                <frustum>
                    <yaw>-54.649008076582298</yaw>
                    <pitch>30.086445865802251</pitch>
                    <roll>-35.258246325136348</roll>
                    <rightAngle>26.536123041449411</rightAngle>
                    <leftAngle>-26.579532980277939</leftAngle>
                    <upAngle>20.556835397199524</upAngle>
                    <downAngle>-20.543387795160175</downAngle>
                </frustum>
                <coordinateFrame>
                    <posx>0.99906</posx>
                    <posy>-0.70781</posy>
                    <posz>-0.70934</posz>
                    <yawx>0.00000</yawx>
                    <yawy>0.00000</yawy>
                    <yawz>-1.00000</yawz>
                    <pitchx>1.00000</pitchx>
                    <pitchy>0.00000</pitchy>
                    <pitchz>0.00000</pitchz>
                    <rollx>0.00000</rollx>
                    <rolly>1.00000</rolly>
                    <rollz>0.00000</rollz>
                </coordinateFrame>
            </region>


The `<frustum>` defines the projector's orientation, and its field of view. See below.  
The `<coordinateframe>` defines the projector's position... and is complicated. See below.  

Let's look at each part.


`<coordinateframe>`
------------

The `<coordinateframe>` node in the MPCDI xml spec is not... obvious:

    <coordinateFrame>
        <posx>1.76840</posx>
        <posy>-2.25391</posy>
        <posz>3.21997</posz>
        <yawx>0.00000</yawx>
        <yawy>0.00000</yawy>
        <yawz>-1.00000</yawz>
        <pitchx>1.00000</pitchx>
        <pitchy>0.00000</pitchy>
        <pitchz>0.00000</pitchz>
        <rollx>0.00000</rollx>
        <rolly>1.00000</rolly>
        <rollz>0.00000</rollz>
    </coordinateFrame>



Section 2.2.2:
> “Positions can simply be expressed as a 3D point. Each of the 3D vectors relates to the Section 2.2
> definitions provided for yaw, pitch, and roll. They will be expressed as orthonormal unit vectors for
> each of the yaw, pitch, and roll directions. (See Figure 2-2 for a graphical representation of these
> unit vectors.)”

That's not much to go on.  By what is now convention, Mystique uses `<coordinateframe>` as so:

The intent of the node is to communicate where the object (ie projector) is, but that should just be 3 numbers:

        <posx>1.76840</posx>
        <posy>-2.25391</posy>
        <posz>3.21997</posz>

The other numbers explain "where" those numbers are.  ie for `<posx>`, _which way is x?_

"Figure 2-2" is this picture of an airplane:

![alt text][airplane]


Now imagine that we've turned the plane so that it is facing into the computer screen.

ie the airplane (and Roll Axis) is facing “forward”,
and you - or the audience - are sitting in the plane,
and you and the plane are facing the screen or object we want to project onto.
(similarly, when designing in 3D software, imagine forward is _into_ the computer screen).

Now, in this scenario, the Pitch Axis arrow is pointing to the right - let's call that the X axis (as everyone seems to agree X is to the right).
So we encode that into the `<coordinateframe>`:

        <pitchx>1.00000</pitchx>
        <pitchy>0.00000</pitchy>
        <pitchz>0.00000</pitchz>

This says _the tip of the Pitch Axis arrow_ is at (1,0,0). And we know Pitch is to the right, so X is to the right! (And the arrow is one unit in length.)

We have now defined what "X", for `posx`, means!

Now the tricky parts - up and forward.  For many OpenGL-based systems, Y is up and -Z is forward.  This would give:

        <yawx>0.00000</yawx>
        <yawy>-1.00000</yawy> // Y is up, so -1 because Yaw axis points down
        <yawz>0.00000</yawz>
        <rollx>0.00000</rollx>
        <rolly>0.00000</rolly>
        <rollz>-1.00000</rollz> // -Z is forward

HOWEVER  Mystique uses a “surveyor” system, where **Z is up, Y is forward, and X is to the right.**

To encode the Mystique coordinate system, we fill out the coordinateFrame with:

        <yawx>0.00000</yawx>
        <yawy>0.00000</yawy>
        <yawz>-1.00000</yawz>  // Yaw arrow points down, -Z is down, so +Z is up
        <pitchx>1.00000</pitchx>
        <pitchy>0.00000</pitchy>
        <pitchz>0.00000</pitchz>
        <rollx>0.00000</rollx>
        <rolly>1.00000</rolly>  // Roll ie forward is Y
        <rollz>0.00000</rollz>

ie the tip of Yaw axis arrow is at (0,0,-1) ie -1 in Z, ie positive Z is up
the tip of the Pitch axis arrow is at (1,0,0) ie 1 in X, as X is to the right (as always)
the tip of the Roll axis (ie forward) is at (0,1,0) ie Y is forward

So now we know that the position:

        <posx>1.76840</posx>
        <posy>-2.25391</posy>
        <posz>3.21997</posz>

means

        <posx>1.76840</posx> // to the right
        <posy>-2.25391</posy> // negative, so step back 2.25391 units (meters typically) from screen
        <posz>3.21997</posz> // up


To convert to a Y-up, -Z-forward system, you need to remap based on that coordinateFrame.
ie from Mystique to Y-up, -Z-forward you could do:

    tmp = Z;
    Z = -Y;
    Y = tmp;

But in general, you form a matrix from the MPDCI coordinateFrame to your coordinate frame.

Another way to look at it:

Since the Roll axis determines forward, then:

    forward = posx * rollx  +  posy * rolly  +  posz + rollz;

Similarly:

    up = -(posx * yawx  +  posy * yawy  +  posz + yawz); // neg - Yaw points down
    right = posx * pitchx  +  posy * pitchy  +  posz + pitchz;

which you can then move into your x,y,z system.



`<frustum>`
-------

`<frustum>` describes which way the projector is facing, as well as its field of view.

                <frustum>
                    <yaw>-54.649008076582298</yaw>
                    <pitch>30.086445865802251</pitch>
                    <roll>-35.258246325136348</roll>
                    <rightAngle>26.536123041449411</rightAngle>
                    <leftAngle>-26.579532980277939</leftAngle>
                    <upAngle>20.556835397199524</upAngle>
                    <downAngle>-20.543387795160175</downAngle>
                </frustum>
                
The `yaw`, `pitch`, and `roll` are angles (in degrees) around the axes as shown in the airplane diagram.  Note the direction of the arrows.  Also note the order: from the point of view of the _pilot_, 

1. first the plane (or projector) is turned left/right by `yaw` degrees, where positive is a turn to the right.
2. then the plane/projector is tilted up by `pitch` degrees
3. then the plane/projector is rolled along its axis - ie a positive roll is raising the left wing while lowering the right wing

(if all 3 of `(yaw, pitch, roll)` are 0, the plane/projector is facing forward along the roll axis.  ie "into" the screen of the computer, or towards the surface being projected on)


### `<rightAngle>` et al

The 4 angles form a field of view of the projected light.  Positive angles are to the right and up.

Looking down at a planar projection from above,

`L` is the `<leftAngle>`  - note this is likely negative
`R` is the `<rightAngle>`  
`t` is the throw distance  
`e` is the distance to the left (resulting from `leftAngle`)  
`r` is the distance to the right (resulting from `rightAngle`)  
`h` is the horizontal offset distance  
`w` is the width of the screen  



![alt text][frustum]

Let's do some math:

```
tan(-L) = e/t  (as L in the diagram would be negative)
tan(R) = r/t

e = t * tan(-L)
r = t * tan(R)

e + r = w
t*tan(-L) + t*tan(R) = w
tan(-L) + tan(R) = w/t

let T = throw-ratio = t/w 

T = 1 / (tan(-L) + tan(R))
T = 1 / (tan(R) - tan(L))
```

OK, from left and right angles, we have the throw ratio.  We also need the lens offset:

```
h = horz offset (as distance, not percent)
From the diagram:
	r - h = w/2
	e + h = w/2
Thus:
        r - h = e + h
Recall r = t*tanR and e = -t*tanL
Sub:
	t*tanR – h = -t*tanL + h
Rearrange:
	h = t(tanR+tanL)/2
Recall: T = t/w  ie  t = Tw
Sub:
	h = Tw(tanR+tanL)/2
And (from above): T = 1/(tanR-tanL)
Sub:
	h = (1/(tanR-tanL))w(tanR+tanL)/2
ie
	        w(tanR + tanL)
	h =  -------------------
	        2(tanR - tanL)

Typically lens offset is represented as a percent of half-width:
	h% = h/(w/2)
	h% = (tanR + tanL)/(tanR - tanL)
    
Similarly, vertical offset:
    v% = (tan(Up) + tan(Down)/(tan(Up) - tan(Down))
```


### Other math, everyone likes math

FOV (Field of View) is not just left + right, because FOV is measured from the middle of the screen:

![alt text][fov]

Thus `FOV = 2atan((tanR-tanL)/2)` (remember `leftAngle` is typically negative)  

Aspect Ratio = `(tan(rightAngle) - tan(leftAngle)) / (tan(topAngle) - tan(bottomAngle))`



## Extensions

The MPCDI spec allows for extensions.  All `<extension>` nodes are found within the top-level `<extensionSet>` node.

Mystique writes two extensions, `<MystiqueExtension>` and `<BufferExtension>`

### MystiqueExtension

This extension includes a `<GeneratedBy>` node that has a string that somehow identifies the application that wrote the file.  There is no defined format for this string, but it should be enough to uniquely identify the application (including version, etc).

    <MPCDI>
        ...
        <extensionSet>
	    ...
            <extension Name="MystiqueExtension" hasExternalFiles="false">
                <GeneratedBy>Mystique Install 2.1.0.0</GeneratedBy>
            </extension>
	    ...
        </extensionSet>
	...
    </MPCDI>

### BufferExtension

The definition of `region` in MPCDI lacks the ability to specify lens info such as fisheye coefficients and/or anamorphic parameters. Instead of hacking these directly onto the `region` node, a `<BufferExtension>` node is added, that parallels the `buffer` node.  For each `buffer` and each `region` that needs extra data, the extra data can be found in the `BufferExtension`.

A full explanation of the fisheye and anamorphic extensions can be found in the [fisheye.md](fisheye.md) document.


[airplane]: MPCDI_plane.png "secret <coordinateframe> decoder ring"
[frustum]: ThrowRatioAndOffset.png "secret <frustum> decoder ring"
[fov]: fov.PNG "fov != L + R"



