# MPCDI Extensions for Fisheye and Anamorphic Lens

MPCDI does not allow for non-planar lenses.
We have added fish-eye and anamorphic data as an extension.

We have added an extension called `BufferExtension` which contains extra data for each buffer/region as necessary.
Note that `<fisheye>` also contains horizontal and vertical offset info - for a planar lens, this can be calculated from the `<frustum>` node, but in the case of a 180 degree fisheye, you would get divide by zero :-/ So we instead just added the numbers you really want, directly.

Example:

    <extensionSet>
        <extension Name="BufferExtension" hasExternalFiles="false">
            <buffer id="wolf_90_markerless.cal">
                <region id="Staging Projector_01">
                    <fisheye>
                        <horizontalOffset>0.20000</horizontalOffset>
                        <verticalOffset>-0.10000</verticalOffset>
                        <f1>0.29907</f1>
                        <f3>-0.0002311605</f3>
                        <f5>0.0000060483</f5>
                        <f7>-0.0000000747</f7>
                    </fisheye>
                    <anamorphic>
                        <angle>0.01000</angle>
                        <ratio>1.20000</ratio>
                    </anamorphic>
                </region>
            </buffer>
        </extension>
        <extension Name="MystiqueExtension" hasExternalFiles="false">
            <GeneratedBy>Mystique Install (Engineering Build) 2.2.0.0 Jan 14 2019</GeneratedBy>
        </extension>
    </extensionSet>


## What are those `f1,f3,f5,f7` numbers?

Those are the **fisheye coefficients**.

![alt text][fisheye]

 A typical, _equidistant_ fisheye lens follows this equation:

> r = f⋅θ


See also https://en.wikipedia.org/wiki/Fisheye_lens

So light entering a lens (ie for a camera) comes in at an angle `θ` (in radians, say). `f` is the focal length. `r` is the distance between the resulting pixel on the camera flim/sensor and the center of the film/sensor.

`f` is typically measured in `mm`.  However, if you only have the resultant image, not the actual lens, you can only measure `r` in terms of pixels, or in units relative to the width of the sensor. If we set 1 to be half the width of the DMD (to match how lens-offset is measured) then this means `f` is measured in `half-widths/radians`.

![alt text][fisheyeflat]
  
Note that is the equation of light _entering_ the lens of a camera, and landing on the sensor.  We are more concerned with light _exiting_ the lens of a projector.  The equation is the same, but we are now talking about DMD pixels, not sensor pixels.

### Generalizing Fisheye Lenses

https://en.wikipedia.org/wiki/Fisheye_lens lists a number of different fish-eye lens equations.

We can approximate all of these via a polynomial (somewhat similar to Taylor expansion), ie:

> r = f<sub>1</sub>⋅θ + f<sub>2</sub>⋅θ<sup>2</sup> + f<sub>3</sub>⋅θ<sup>3</sup> + f<sub>4</sub>⋅θ<sup>4</sup> + f<sub>5</sub>⋅θ<sup>5</sup> + f<sub>6</sub>⋅θ<sup>6</sup> + f<sub>7</sub>⋅θ<sup>7</sup>...

Now, it turns out that only the odd coefficients are needed (you want an odd curve, not even, etc, etc, ...vigourous hand waving...).  And the coefficients get smaller and smaller, so there isn't much sense going past `f7`, so:
    
> r = f<sub>1</sub>⋅θ + f<sub>3</sub>⋅θ<sup>3</sup> + f<sub>5</sub>⋅θ<sup>5</sup> + f<sub>7</sub>⋅θ<sup>7</sup>
    
So that's the general fisheye equation that we use.

### Offsets

Lens aren't centered.  
`r` is the distance from the _principal axis_, but that might not be in the center of the DMD.  So we also export the horizontal and vertical offsets (similar to regular planar lens offsets).

## Anamorphic

In addition to fisheye, a lens can be anamorphic - ie non-square pixels. ie stretched a bit vertically or horizontally.  We export this as `<ratio>` which is simply the ratio of squareness of a pixel.  An anamorphic ratio of 1.0 is normal, square pixels.  A ratio of 1.2 is _taller_ pixels.  ie y/x.  (ie anamorpihc ratio is the reciprocal of the typical TV/video x/y aspect ratio.  But things like throw-ratio are width-based, so we make anamorphic ratio relative to width as well.)

![alt text][anamorphicRatio]

Lastly there is a skew `<angle>`. Lenses are not always attached 100% aligned.  Is the lens is slightly turned to the left? right? then the skew angle will not be `0` degrees


![alt text][anamorphicAngle]






[fisheye]: fisheye.PNG "simple equidistant fisheye lens"
[fisheyeflat]: fisheyeflat.PNG "on the sensor"
[anamorphicRatio]: anamorphicRatio.PNG "anamorphicRatio of 1.2"
[anamorphicAngle]: anamorphicAngle.PNG "anamorphic skew angle"
