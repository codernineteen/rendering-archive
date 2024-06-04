
# Algorithm

1. construct a ray as origin of camera position and direction from camera position to current pixel coordinates on screen
2. Using the inverse of projection and view matrices, we convert the ray into world space representation.
3. check intersection with the scene objects or check if it is missed
4. do shading on the surface
5. reflect or refract the light directions
6. Recursively iterate the algorithm following our predefined depth of recursion.


![](../../../../images/Pasted%20image%2020240530121054.png)
