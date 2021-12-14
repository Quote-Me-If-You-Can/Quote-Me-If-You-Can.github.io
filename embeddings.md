---
title: Embedding projector
layout: page
---

### Using the Tensorflow Embedding Projector <br>

This page is all about the **[Embedding Projector](http://projector.tensorflow.org/?config=https://gist.githubusercontent.com/dmizr/6ed0d83d738a86a3d57e7a8455efe83f/raw/6b7aed45e8d7d5eec7d4f5fb0f71d9c74f0423e8/projector_config_all.json)**.


Every point you see corresponds to a game embedding projected on a 3D space. 
If you click on any of the floating points you will get some information on the closest neighbors.
Keep in mind that the "closest neighbors" are calculated in the original 100 dimensional space, so it will not always correspond to the nearest points in the projection.
You can move around the projection by clicking and dragging across the central space, or by zooming in and out.

<figure>
    <img src="../img/tensorflow_1.png">
    <figcaption></figcaption>
</figure>

- 1 : By clicking on this box you can select between 3 datasets :
    - The game embeddings, which will show the point cloud of games.
    - The tag embeddings, which will show the point cloud of some of the most popular user-defined tags.
    - The genre embeddings, which will show the point cloud of the official Steam tags.

- 2 : This drop-down menu allows you to color the games based on their tag or genre. Try coloring games by a tag, and then move the projection around. You'll likely notice that games with a specific tag are often near each other, and that it's sometimes possible to find a view of the projection that nicely separates the points!

- 3 : This menu allows you to select which axes from PCA you want to show, or even switch to 2D. You can also use other dimensionality reduction techniques like UMAP or t-SNE, don't be shy!

- 4 : This field allows you to enter a game's name to locate it automatically. If you're out of inspiration, here are some varied and interesting games to look up:
    - [Among Us](https://store.steampowered.com/app/945360/Among_Us/): a wildly popular social deduction game
    - [Euro Truck Simulator 2](https://store.steampowered.com/app/227300/Euro_Truck_Simulator_2/): an open-world truck simulation game
    - [Hollow Knight](https://store.steampowered.com/app/367520/Hollow_Knight/): a highly-praised 2D indie action-adventure game
    -  [Call of Duty: Infinite Warfare](https://store.steampowered.com/app/292730/Call_of_Duty_Infinite_Warfare/): a high-budget first-person shooter game


<figure>
    <img src="../img/tensorflow_2.png">
    <figcaption></figcaption>
</figure>

- 5 : When a game is selected, the closest points are displayed in this section. Different distance metrics are available, try them both to see which one you understand the best. 

- 6 : If you click on this rectangle when a point is selected, there will be a drop-down showing which tags (from Steam or the users) are attributed to this game.

### Our data <br>

All our embeddings are openly available in TSV format. Feel free to use them for further analysis!
- [Game embeddings](https://gist.github.com/dmizr/f9de09c761f35cc5c3c8b14ad24c5eff)
- [Tag embeddings](https://gist.github.com/dmizr/8c10552beccebc44826cb3e91dae740c)
- [Genre embeddings](https://gist.github.com/dmizr/219491ae705c07d61002d2d7a5c2c03d) 