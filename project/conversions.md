### Maya

To convert Mystique's MPCDI rotations to Maya rotations:

1. In Maya, from the menu, select **Modify** -> **Reorder Rotation**  
In the dialog, select **zxy** as the new rotation order

2. Assign MPCDI yaw/pitch/roll from `<frustum>` to Maya values like so:

| Maya | MPCDI |
|:---:|:---:|
| X | pitch |
| Y | -yaw |
| Z | -roll |

3. if desired, after settings these values, reset **Rotation Order** back to xyz or whatever the user wants (Maya should recalculate the numbers to be correct for xyz order, and the object should not change orientation)
