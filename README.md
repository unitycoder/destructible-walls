# Unity Fracture
https://user-images.githubusercontent.com/36990593/118142411-a53f6500-b40a-11eb-9942-4a6b0c1c1a42.mp4
### In order to have destructible geometry, what do we actually need?
1) Cut the mesh into smaller meshes chunks
2) Add rigidbody component to each chunk
3) Connect chunks with fixed joints that break with force

#### 1) Cut the mesh into smaller meshes chunks
I started reading up on Boolean operations and Voronoi. It got complicated pretty quickly, there are a bunch of assets on the store, but either they are slow, buggy or don't work at all. Luckily I stumbled upon this forum thread https://forum.unity.com/threads/nvidia-blast.472623. Someone figured out how to use Nvidia blast library in the Unity. It gets pretty straightforward after this. You just feed the mesh to this library and receive chunks.

#### 2) Add rigidbody component to each chunk
Convert each mesh chunk into a gameobject with rigidbody. Without anything holding the chunks together they crumble to the ground. Connect the chunks with fixed joints, so they stay in place. Take each chunk and its neighbors (chunks that are in close proximity or in touch) and connect them with fixed joints.

##### Issue #1 - Structure is wobbly 
This is not ideal, the joints are not 100% fixed in place due to how PhysX resolves collisions. There is a big deal of springiness within the joints. The structure ripples when force is applied and acts like a it is made out of jello.

##### Solution #1 - Freeze rigidbodies
https://answers.unity.com/questions/230995/fixed-joint-not-really-fixed.html advised to freeze the rigidbody. Freezing the rigidbodies makes them stay in place (they drift apart after some time [#1](https://github.com/ElasticSea/unity-fracture/pull/8)) and the rigidbodies still register impact and the joints can be broken.

##### Issue #2 - Rigidbodies float in the air
Rigidbodies will stay in place even though there is no support under them. They will only resume movement when all joints are broken.

##### Solution #2 - Introduce anchors
Let's create a graph of connected chunks. Traverse graph each frame and unfreeze chunks disconnected from anchors (kinematic body)

![Chunk Graph](https://user-images.githubusercontent.com/36990593/118148705-21d54200-b411-11eb-94f6-5c654f420df9.png)
*Anchored chunks are red, frozen chunks are black, other colors represent disconnected groups*
