---
layout: page
title: All your review are belong to us
cover-img: "img/steam_big.jpg"
subtitle: An EPFL-Unil project on Steam reviews
classes: wide
---

{% include konami.html %}

## Video Gamez R kool

The number and variety of video games is humongous. As a relatively new medium, video game’s impressive growth attracts numerous scholars from different fields, though in a sometimes frustrating way. Many studies are focused on small scales and lack representativity, while others stay on an abstract level with their data handling.

**So, what better way to make an interdisciplinary study of video games than to bring humanities, game studies and machine learning together?**

With an enormous dataset of Steam reviews, we question the relevance of game genres, explore the possibility of classifying games based on their reviews, and study the resemblance between this classification and genres as well as user-defined tags.
Continue reading for more details and other interesting statistics on Steam reviews!


## Table of Contents

- [Dataset](#dataset)
- [Game embeddings](#game-embeddings)
- [Classification with K-means clustering](#classification-with-k-means-clustering)
- [Clusters and tags comparison](#clusters-and-tags-comparison)
- [A closer look at the reviews](#a-closer-look-at-the-reviews)
- [PCA and classification](#pca-and-classification)
- [What about the users?](#what-about-the-users)
- [Conclusion](#conclusion)
- [References](#references)



## Dataset

For this project, we used a large-scale dataset of Steam reviews compiled in 2021, which is openly available on Kaggle[^1].

#### Contents

This dataset consists of over reviews published on Steam, an online video game marketplace. In total, there are 21 million reviews, 10 million of which are in English, from over 300 different games. For each of these reviews, additional metadata is provided, such as the timestamp, the number of users who found that review helpful or funny, whether that game was received for free or during early-access, and other user-related information.

#### Source caveats

Because only a subset of all reviews from Steam were kept for this collection, potential biases were introduced.
We asked the author for more information on the selection of the games. They said: *"The games were picked by hand without stringent criteria. I did, however, pay attention to a few aspects and made an effort to include: both mainstream titles as well as indie titles, largely positively reviewed games as well as largely negatively reviewed games, and some older titles alongside newer titles, though you'll find most of the dataset is comprised of games released in 2017 and later."*.
They also added regarding the proportion of reviews collected for each selected game: *"I attempted to collect all the reviews at the time (Jan. 2021) for every title, and as such, no reviews were left out intentionally. I did observe there were some reviews missing but I do not have an exact number. If I remember correctly it was usually something like 1% of a game's reviews that was missing."*

#### Game tags

To complete our analysis, we gathered the official tags from Steam (also called genres) and the most popular user-defined tags. This procedure was made manually and some tags were either broken down into subparts or ignored to avoid synonyms while staying general.
For example, FPS was broken down into 'First person' and 'Shooter', and 'Tactics' was ignored for the more frequent 'Strategy' was already taken. The decision to avoid/break the different tags or not was highly subjective but mostly done by one person to stay consistent. The games and their tags are publicly accessible [here](https://www.notion.so/ae461477418242858455d878c7647f5f?v=da64bc60b507481483510043f76169c3).


## Game embeddings

Given the massive quantity of reviews, one key challenge is to condense all the information contained in a useful way. To do so, we tried to obtain numerical representations for each game, which we call **game embeddings**. Ideally, these embeddings would capture key information about the games and place conceptually similar games close together in the embedding space.

The process to obtain these embeddings is quite involved, although we can split it in two key steps: First, we take the textual content from all the English reviews and use it to train a Sent2vec[^2] model to obtain 100-dimensional sentence embeddings. Then, we take the 1000 most helpful reviews for each of the 200 most reviewed games, feed them to the trained Sent2vec model, and average the resulting sentence embeddings for each game. This gives us a 100-dimensional embedding for each of the top 200 most reviewed games in our dataset. This process is summarized in the figure below.

<figure>
    <img src="img/Embeddings_method.jpg">
</figure>

The final game embeddings, and similarly obtained genre and tag embeddings, are publicly available, and can easily be visualized using the [Tensorflow Embedding Projector](http://projector.tensorflow.org/?config=https://gist.githubusercontent.com/dmizr/6ed0d83d738a86a3d57e7a8455efe83f/raw/6b7aed45e8d7d5eec7d4f5fb0f71d9c74f0423e8/projector_config_all.json). We heavily recommend taking a look at [**this dedicated page**]({% link embeddings.md %}) for more info.

When looking at these embeddings, we notice that similar games are usually very close together in the embedding space, and that games with specific tags are often grouped together. We find these embeddings to be quite satisfying, and use them throughout our analysis.


## Classification with K-means clustering

We now try to see if we can use clustering on these embeddings to group similar games together. To do so, we pick K-Means, a simple yet effective technique for distance-based clustering. We used the [Silhouette Coefficient](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.silhouette_score.html) to determine a good number of clusters, but we also chose a high number in order to get more specific clusters, which would be more interesting to analyze. After a bit of trial and error with different seeds and cluster numbers, we settled on a 20-cluster classification that produced fairly satisfying results.


<details>
<summary><b>Click to view clusters</b> (games ordered alphabetically)</summary>
<br>
<ul>

<li><b>Cluster 1:</b> Banished, Factorio, Frostpunk, Kenshi, No Man's Sky, Oxygen Not Included, RimWorld, Satisfactory, Subnautica, Super Hexagon, Surviving Mars</li>

<li><b>Cluster 2:</b> Assassin's Creed Odyssey, Assassin's Creed Origins, Batman: Arkham Asylum GOTY Edition, BioShock Infinite, Dishonored, Dying Light, Rise of the Tomb Raider, Tomb Raider</li>

<li><b>Cluster 3:</b> Age of Empires II (2013), BATTLETECH, Crusader Kings III, Europa Universalis IV, Hearts of Iron IV, Mount & Blade: Warband, Northgard, STAR WARS™ Empire at War: Gold Pack, Sid Meier's Civilization V, Sid Meier's Civilization VI, Stellaris, They Are Billions, Total War Saga: Thrones of Britannia, Total War: ROME II - Emperor Edition, Total War: THREE KINGDOMS, Total War: WARHAMMER, Total War: WARHAMMER II, Warhammer 40,000: Dawn of War III, XCOM 2</li>

<li><b>Cluster 4:</b> A Hat in Time, Celeste, DEATH STRANDING, GRIS, Hollow Knight, Little Nightmares, Ori and the Will of the Wisps, To the Moon</li>

<li><b>Cluster 5:</b> ARK: Survival Evolved, ATLAS, BATTALION 1944, Black Desert Online, Blackwake, Bless Online, Conan Exiles, Dead by Daylight, For Honor, Hunt: Showdown, Insurgency: Sandstorm, MORDHAU, Nether, PLAYERUNKNOWN'S BATTLEGROUNDS, Rust, SCUM, Tom Clancy's Rainbow Six Siege, Warhammer: Vermintide 2</li>

<li><b>Cluster 6:</b> Darkest Dungeon®, Dead Cells, Deep Rock Galactic, Enter the Gungeon, FTL: Faster Than Light, For The King, Hades, Into the Breach, Slay the Spire, Streets of Rogue, Warhammer 40,000: Mechanicus</li>

<li><b>Cluster 7:</b> Among Us, BattleBlock Theater, Broforce, Castle Crashers, Duck Game, Golf It!, Human: Fall Flat, Keep Talking and Nobody Explodes, Overcooked! 2, Phasmophobia, Tabletop Simulator</li>

<li><b>Cluster 8:</b> Danganronpa 2: Goodbye Despair, Danganronpa: Trigger Happy Havoc, Doki Doki Literature Club, Helltaker, HuniePop, Mirror, Monster Prom, Night in the Woods, OneShot, Papers, Please, South Park™: The Stick of Truth™, The Henry Stickmin Collection, The Walking Dead, The Wolf Among Us, Undertale, VA-11 Hall-A: Cyberpunk Bartender Action</li>

<li><b>Cluster 9:</b> Borderlands 3, Call of Duty: Infinite Warfare, Call of Duty: WWII, Far Cry 5, Just Cause 3, Just Cause 4, Saints Row: The Third, Sniper Elite 4, Tom Clancy's Ghost Recon® Wildlands, Watch_Dogs 2</li>

<li><b>Cluster 10:</b> FINAL FANTASY XV WINDOWS EDITION, NieR:Automata™, Persona 4 Golden, Tales of Berseria, The Witcher 3: Wild Hunt, Yakuza 0</li>

<li><b>Cluster 11:</b> Black Mesa, Crash Bandicoot™ N. Sane Trilogy, DOOM, DOOM Eternal, DUSK, Half-Life, Half-Life 2: Episode Two, Half-Life: Alyx, Outlast, Resident Evil 2, Resident Evil 7 Biohazard</li>

<li><b>Cluster 12:</b> Arma 3, Beat Saber, Counter-Strike: Source, Day of Infamy, Garry's Mod, People Playground, Ravenfield, Totally Accurate Battle Simulator, Totally Accurate Battlegrounds, Wallpaper Engine</li>

<li><b>Cluster 13:</b> Cube World, Divinity: Original Sin 2, FINAL FANTASY XIV Online, Fallout 4, Kingdom Come: Deliverance, Middle-earth™: Shadow of War™, Mutant Year Zero: Road to Eden, Pathfinder: Kingmaker, Pillars of Eternity II: Deadfire, The Elder Scrolls Online, Vampyr</li>

<li><b>Cluster 14:</b> Artifact, Bloons TD 6, Grand Theft Auto V, HITMAN™ 2, PAYDAY 2, The Elder Scrolls V: Skyrim, The Elder Scrolls V: Skyrim Special Edition</li>

<li><b>Cluster 15:</b> American Truck Simulator, BeamNG.drive, Euro Truck Simulator 2, Rocket League, The Crew 2</li>

<li><b>Cluster 16:</b> DARK SOULS™ III, DARK SOULS™: REMASTERED, DRAGON BALL FighterZ, Monster Hunter: World, Nioh: Complete Edition, Sekiro™: Shadows Die Twice</li>

<li><b>Cluster 17:</b> Don't Starve, Don't Starve Together, My Time At Portia, Raft, Slime Rancher, Stardew Valley, Terraria, The Forest</li>

<li><b>Cluster 18:</b> Cuphead, Getting Over It with Bennett Foddy, Hotline Miami, Mark of the Ninja, One Finger Death Punch, Shovel Knight: Treasure Trove, Super Meat Boy, The Binding of Isaac, The Binding of Isaac: Rebirth</li>

<li><b>Cluster 19:</b> Cities: Skylines, Farming Simulator 19, House Flipper, Jurassic World Evolution, PC Building Simulator, Planet Coaster, The Sims(TM) 3, Two Point Hospital, Youtubers Life, theHunter: Call of the Wild™</li>

<li><b>Cluster 20:</b> Baba Is You, Gunpoint, Portal 2, The Room, The Room Two</li>
</ul>
<br>
</details>

The figure below shows the games which are part of each cluster. Use the menu to navigate to the cluster of your choice.
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="500" src="/html/clusters.html"></iframe>
If you know your games well, you'll see that the clustering worked quite well, aside from a few oddities.

Here are some clusters we found particularly satisfying:
- Cluster 3, which groups grand strategy wargames such as the _Civilization_ and _Total War_ series
- Cluster 19, which contains simulation games of all sorts
- Cluster 7, which comprises many games of different genres and visual styles, but which are all well-known for their strong co-op / multiplayer aspect

Cluster 5 is interesting as well, as it contains a lot of games, including several that don't seem that similar, apart from the fact that they are linked by being multiplayer games.
Maybe their belonging to the same cluster is a sign that the multiplayer component is the most salient part of the game, superseding the other aesthetic and gameplay features, in the players' reviews. Alternatively, it could simply be that these games were hard to group with anything else, and their relative proximity in the embedding space resulted in a cluster (as with K-Means clustering, all games must belong to a cluster).

So, hypotheses can be found to explain the regrouping of games when more obvious aspects are lacking. It appears that we can effectively find a coherent classification using only the users' reviews. To confidently verify the hypothesis that the 5th cluster gathers games because players pay more attention to the multiplayer feature of these games than the others, a thorough linguistic analysis of the reviews' content is necessary.

Although this grouping in 20 clusters is slightly uncertain and not without outliers, it seems that the model was overall able to capture enough information from the users' discourse to form a coherent understanding of each game.


## Clusters and tags comparison

The classical criteria to classify games is the use of genres that describe part of their aesthetic or some gameplay features. Steam also uses tags that users can assign to games, and the more popular ones are displayed on the games' pages on the platform. Do our clusters connect to the corresponding genres and tags of each game?
To answer this question, we isolated for each cluster the genres and tags that were present in all of their games. One interesting thing to keep in mind is that the model for game embeddings had no information about the related tags during training outside the direct use of those words in the reviews, which is not highly significant. For example, the word "indie" does not even show up in the top 1000 most frequently used words, appearing in less than 0.3% of reviews.

| Cluster | Size | Steam Genres | User-defined Tags | Prototype |
|-------|--------|---------|---------|---------|
| 1 | 11 | - | User Singleplayer | _Oxygen Not Included_ |
| 2 | 8 | Action | Action, Story Rich | _Rise of the Tomb Raider_ |
| 3 | 19 | - | Multiplayer, Singleplayer, Strategy | _Total War Saga: Thrones of Britannia_ |
| 4 | 8 | - | Adventure, Great Soundtrack, Singleplayer | _GRIS_ |
| 5 | 18 | Action | Action, Multiplayer | _SCUM_ |
| 6 | 11 | - | - | _Slay the Spire_ |
| 7 | 11 | - | Co-op, Multiplayer | _BattleBlock Theater_ |
| 8 | 16 | - | - | _Danganronpa: Trigger Happy Havoc_ |
| 9 | 10 | Action | Action, Co-op, Shooter, Singleplayer | _Tom Clancy's Ghost Recon® Wildlands_ |
| 10 | 6 | RPG | Adventure, Great Soundtrack, RPG, Singleplayer, Story Rich | _Tales of Berseria_ |
| 11 | 11 | Action | Action, Singleplayer | _Resident Evil 2_ |
| 12 | 10 | - | Action | _Ravenfield_ |
| 13 | 11 | RPG | RPG | _Pillars of Eternity II: Deadfire_ |
| 14 | 7 | - | Great Soundtrack, Singleplayer | _PAYDAY 2_ |
| 15 | 5 | - | Singleplayer | _Euro Truck Simulator 2_ |
| 16 | 6 | Action | Action, Adventure, Difficult, Great Soundtrack | _Nioh: Complete Edition_ |
| 17 | 8 | Indie | Indie, Open World, Sandbox, Singleplayer | _Don't Starve_ |
| 18 | 9 | - | Indie, Singleplayer | _Super Meat Boy_ |
| 19 | 10 | Simulation | - | _Farming Simulator 19_ |
| 20 | 5 | - | Puzzle | _The Room_ |

The size indicates the number of games in each cluster, and the prototypes are the most central games in each cluster. Prototypes can be further analysed to try and create a classification system based on representative games, though this goes beyond the aim of our study. However, they can still be useful to better understand the composition of each cluster, as we tried to do in the previous section.

We see that most clusters effectively contain games with a couple of tags or genres in common.
It is interesting to see the contrast between the tags and the diversity of some clusters. Looking at cluster 5 again, we find the common idea of multiplayer action, while the rest of the gameplay of these games is relatively varied. This reinforces our earlier hypothesis but is insufficient to verify it.
Following it, we could interpret our clustering as a model selecting the defining feature(s) of each game to then place them in corresponding clusters. Once again, this idea should be further explored by looking at the reviews and find whether the characteristics associated with the genres and tags standing out for each cluster do in fact appear significantly more than other patterns in the players' discourse.

Such results, with a more thorough analysis, could help us determine the most defining feature(s) of a game and classify genres and tags according to the value players assign to them in a game.
This could help players orient themselves more precisely when looking for a specific type of experience, contrary to simply search for a tag or genre's presence in a game, which does not indicate how important this characteristic will be in said game.

In addition to this overall suggestion that some genres and tags are more salient than others depending on the games, this table offers an interesting perspective on the genres and user-defined tags themselves. Indeed, at the exception of cluster 19, all genres defined by Steam are appearing in the user tags as well.

Supposedly, then, Steam users relate to the official genres. However, the games from clusters 2, 5, 9, 11, and 16 all share the same genre, Action. They can be distinguished by their tags, which are more precise. The same happens with the RPG genre in clusters 10 and 13. In cluster 13, the only tag shared by all games is RPG as well, while cluster 10 has several tags complementing it. Possibly, the games from cluster 13 could be considered as the most representative of the RPG genre, while cluster 10 regroups RPG games with other salient features.
Although the official genres are generally included in the user-defined tags, they appear to be insufficient by themselves, especially the Action genre.

If we were to create a new categorisation system, then, these genres would have their share of information to give, but what is most striking here is how important and useful players’ opinions and perspectives can bring to the discussion.


## A closer look at the reviews <br>
#### Comparing four close games: _Night in the Woods_, _Undertale_, _OneShot_, and _To the Moon_

Four games appear close together in our embeddings: _Night in the Woods_, _Undertale_, _OneShot_, and _To the Moon_, with all of their distance to one another contained between 0.161 and 0.230. Although _To the Moon_ is in cluster 4 and the others in cluster 8, they all have the other games as their top 3 (4 in the case of _Night in the Woods_) closest games. Consequently, they will still be considered as an ‘alternate’ smaller cluster for this analysis.

<div class="container">
    <div class="row" style="background-color: #0b2c39;">
        <div class="col-lg-6 col-md-6 nopadding" style="padding-left: 0;">
            <img src="img/nitw.jpg" onmouseover="this.src='img/nitw_screen.jpg';" onmouseout="this.src='img/nitw.jpg';">
        </div>

        <div class="col-lg-6 col-md-6 nopadding" style="color: #F2F3F4;">
            <div class="boxtext">
                <h2> Night in the Woods </h2><br><br>
                Night in the Woods is a single-player adventure game. It is a story-focused exploration game in which we play Mae, a character coming back to her hometown that she explores, meeting and interacting with non-playable characters.
            </div>
        </div>
    </div>
</div>
<br>
<div class="container">
    <div class="row" style="background-color: #000;">
        <div class="col-lg-6 col-md-6 nopadding" style="color: #F0F0F0;">
            <div class="boxtext">
                <h2> Undertale </h2><br><br>
                In Undertale, we play a human who fell in a hole and ends up exploring an unknown world, encountering villagers and ‘enemies’ we can choose to fight or spare, with our main aim being to come back to the surface and in our own world.
            </div>
        </div>

        <div class="col-lg-6 col-md-6 nopadding" style="padding-right: 0;">
            <img src="img/under.jpg" onmouseover="this.src='img/under_screen.jpg';" onmouseout="this.src='img/under.jpg';">
        </div>
    </div>
</div>
<br>
<div class="container">
    <div class="row" style="background-color: #340b36; ">
        <div class="col-lg-6 col-md-6 nopadding" style="padding-left: 0;">
            <img src="img/oneshot.png" onmouseover="this.src='img/oneshot_screen.jpg';" onmouseout="this.src='img/oneshot.png';">
        </div>

        <div class="col-lg-6 col-md-6 nopadding" style="color: #F2F2F2;">
            <div class="boxtext">
                <h2> Oneshot </h2><br><br>
                OneShot is a story in which we control a child from an outside perspective, acting as a sort of guardian for them. The game also has an interactive part with the operating system, which is praised in the reviews for its originality.
            </div>

        </div>
    </div>
</div>
<br>
<div class="container">
    <div class="row" style="background-color: #1b3e63;">
        <div class="col-lg-6 col-md-6 nopadding" style="color: #EFEFEF;">
            <div class="boxtext">
                <h2> To the Moon </h2><br><br>
                To the Moon is a puzzle and adventure game. You play as two doctors exploring an old man’s memories to alter them, so that he can die with his dying wish completed in his memories.
            </div>
        </div>

        <div class="col-lg-6 col-md-6 nopadding" style="padding-right: 0;">
            <img src="img/ttm.jpg" onmouseover="this.src='img/moon_screen.png';" onmouseout="this.src='img/ttm.jpg';">
        </div>
    </div>
</div>

Looking at the content of these games' reviews to try and find what patterns may have been found by our embedding method, several elements are striking. The four games' reviews have frequent references to terms referring to:
- the story in a highly positive way, commenting on the quality of the writing
- emotions, with words such as 'feelings', 'cry', 'sad', 'happy'
- comments on the games not being focused on gameplay
- remarks on the high quality of the soundtrack
- the word 'experience'

The four of them have 'Story rich' and 'Great Soundtrack' in their top 3 user-defined tags. This is unsurprising: users reviewing the games and assigning them tags are most probably broadly the same people. Reading these four games' reviews, then, there seems to be lexical patterns justifying the measurements made by the program.

#### Anomaly? _Night in the Woods_ and _VA-11 Hall-A: Cyberpunk Bartender Action_

_Night in the Woods_ is the only game of this cluster having a closer neighbour than the three others in this group: _VA-11 Hall-A: Cyberpunk Bartender Action_. It is a game in which the player incarnates a bartender preparing and serving drinks to different customers while listening to them in a cyberpunk setting.

<div class="container">
    <div class="row" style="background-color: #040308; ">
        <div class="col-lg-6 col-md-6 nopadding" style="padding-left: 0;">
            <img src="img/vallhalla.jpg" onmouseover="this.src='img/vallhalla_screen.jpg';" onmouseout="this.src='img/vallhalla.jpg';">
        </div>

        <div class="col-lg-6 col-md-6 nopadding" style="color: #F2F2F2;">
            <div class="boxtext">
                <h2> VA-11 Hall-A: Cyberpunk Bartender Action </h2><br><br>
                In this game you play a bartender at the eponymous VA-11 HALL-A, a small bar in a dystopian downtown which is said to attract the "most fascinating" of people. Gameplay consists of players making and serving drinks to bar attendees whilst listening to their stories and experiences.
            </div>

        </div>
    </div>
</div>

Supposedly, this game's reviews should have more in common with _Night in the Woods_ than with the three other games.
However, 'Story Rich' is the fourth user-defined tag, while 'Great Soundtrack' is absent of the list. Looking at the ten main tags for each game, nothing strikingly separates _Night in the Woods_ and _VA-11 Hall-A: Cyberpunk Bartender Action_ from _Undertale_, _To the Moon_ and _OneShot_.

_VA-11 Hall-A: Cyberpunk Bartender Action_'s reviews contains numerous mentions of sex, sexuality and alcohol, and terms such as 'waifu'/'weeb' are recurrent. Although the reviews frequently refer to the story's depth and how it affected the reviewer, terms such as 'feelings', 'sad' or 'happy' are mainly absent.

A possible link between this game and _Night in the Woods_ could be mental health issues, as terms directly related to these appear in the latter's review, and the former has mentions of alcoholism and personal or relational conflicts (in reference to the game's content). _VA-11 Hall-A: Cyberpunk Bartender Action_'s bring more information, as there are frequent mentions of the game's music as being great, as well as comments on beautiful visuals and a focus on storytelling. The writing is pointed at, though for being shallow and boring. These elements bring the game closer to the cluster previously observed but does not give any indication of a specific similarity with *Night in the Woods* (whose negative reviews mostly mention of the developer, Alec Holowka's suicide).

_VA-11 Hall-A: Cyberpunk Bartender Action_ can be understood as an anomaly when comparing these games' reviews. While it is crucial to keep in mind that this analysis is based on a non-exhaustive overview and that striking resemblances between both games may have been overlooked, this suggests that the game embeddings are built on elements that are not as obvious to the human eye as topics and words choice, at least not only. There are most probably strong elements behind the game embeddings that are inaccessible to us.

#### Opposites: _Undertale_ vs _Insurgency: Sandstorm_ and _Arma 3_

Finally, let us compare the 4-games cluster with opposite ones, focusing on the two most distant from Undertale: _Insurgency: Sandstorm_ (1.541), an action game according to Steam, and _Arma 3_ (1.526), classified as an action, simulation and strategy game. Both are online multiplayer games in which the players engage in shooting fights against other players.

<div class="container">
    <div class="row" style="background-color: #474236;">
        <div class="col-lg-6 col-md-6 nopadding" style="color: #EFEFEF;">
            <div class="boxtext">
                <h2> Insurgency: Sandstorm </h2><br><br>
                Insurgency: Sandstorm is a multiplayer tactical first-person shooter. It is set in an unnamed fictional Middle <a href="https://en.wikipedia.org/wiki/Konami_Code" target="_blank">Easter</a>n region, the game depicts a conflict between two factions: "Security" and "Insurgents".
            </div>
        </div>

        <div class="col-lg-6 col-md-6 nopadding" style="padding-right: 0;">
            <img src="img/sandstorm.jpg" onmouseover="this.src='img/sandstorm_screen.jpg';" onmouseout="this.src='img/sandstorm.jpg';">
        </div>
    </div>
</div>
<br>
<div class="container">
    <div class="row" style="background-color: #17191d; ">
        <div class="col-lg-6 col-md-6 nopadding" style="padding-left: 0;">
            <img src="img/arma.jpg" onmouseover="this.src='img/arma_screen.jpg';" onmouseout="this.src='img/arma.jpg';">
        </div>

        <div class="col-lg-6 col-md-6 nopadding" style="color: #F2F2F2;">
            <div class="boxtext">
                <h2> Arma 3 </h2><br><br>
                ARMA 3 is an open-world, realism-based, military tactical shooter video game. The single-player campaign has the player take control of U.S. Army soldier Corporal Ben Kerry. In the single player mode, the player is placed in a variety of situations, from lone wolf infiltration missions to the commanding of large-scale armored operations. This game has a very active modding community.
            </div>

        </div>
    </div>
</div>

_Insurgency: Sandstorm_'s reviews contain recurrent comments on the game's performances, mentions of sound but as in realistic rather than beautiful/enjoyable (sound design vs soundtrack). A review stood out, describing how passionate another player was, and how this intense experience was deeply appreciated by the reviewer.
This game's reviews are obviously strikingly different from the games observed above.
Similarly, _Arma 3_'s reviews contain mainly feedback on the game's content, which is similar to _Insurgency: Sandstorm_ and quite different from the four other games. However, the reviews contain mentions of 'experience', 'depth' and intense feelings - somewhat like the standing-out review from _Insurgency: Sandstorm_ - that can be paralleled with the four-games cluster.

It is highly possible that terms such as 'experience' are way too frequent to be used as a way to distinguish games' reviews (like 'game', for example); this would explain the similarities between those highly distanced games. While some potential explanation can be found, slight similarities do exist between strongly opposed games in the embeddings.

#### Embeddings: a black box

As the program that created the embeddings does not understand English but measures the distances on grounds unknown to us, it is hard to understand _how_ the distances have been calculated and visualise _why_ two games will be close or distant. Consequently, _Night in the Woods_, _Undertale_, _OneShot_ and _To the Moon_ *may* be connected on elements that are unrelated to the meaning of their reviews. However, there is a crucial lack of information in this analysis, as it is based on an overview of only a part of the reviews, especially the ones voted the most helpful, so there is a lack of representativity here. No strong conclusions can be drawn here, only hypotheses pointing out the difficulty to pinpoint the elements determining the distance between games.

## PCA and classification

Sometimes, the Principal Component Analysis (PCA) decomposition can lead to interesting projection vectors.
Out of curiosity, we tried to see if the PCA dimensions had similarities with the tags repartition.
For each PCA dimension, we plotted the mean position of each tag
(Once all games are projected, we calculate the mean position of all games possessing the tag).
On the second plot, you can visualize each dimension and the position of all tagged games to get a better idea.

#### Repartition of tags along each PCA axes<br>

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="500" src="/html/tag_repartition.html"></iframe>
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="500" src="/html/dim_games.html"></iframe>

It was expected, but individual axes do not correspond to individual tags and cannot be properly interpreted.
Still, we can use these graphics to find the most representative axes for a specific tag and better visualize groups of games on the [Embedding Projector]({% link embeddings.md %}).
(Try components 1,2 and 4, and select color by User Story Rich to see most of the Story Rich games clumped together in a corner of the space.)

While that's all we can get out of PCA on the game embeddings themselves, what happens if we use it in on our other embeddings? To find out, we merge the embeddings of the top user tags (20 of them) and top official Steam genres (9 of them), and use PCA to project them in a 2D space. The results are shown in the figure below (click on the legend to hide / reveal the user-defined tags and Steam genres). 

#### 2D PCA projection of user tags and Steam genre embeddings<br>

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="600" src="/html/all_tag_pca.html"></iframe>

This projection gives two interesting results:
- First, we notice that identical user tags and official Steam genres are usually quite close together, which aligns with our findings in the clustering section. However, some broad genres (e.g. action) are much further apart than more objective tags (e.g. indie games). This indicates that, while the genres / tags are mostly consistent, some of them might be too broad to properly categorize a game (e.g. action might not mean exactly the same thing for everyone), and that more fine-grained game genres are needed.
- Second, the two principal components seem to represent two important concepts: whether a game is single-player or multiplayer (horizontal axis), and the game's audience: casual or serious (vertical axis). This may indicate that we can describe many aspects of a game, such as the gameplay, atmosphere and soundtrack, based on how strongly associated they are associated with each of these concepts. This may also help us understand how genres relate to each other.



## What about the users?

As a bonus, let's deep-dive into some stats obtained using the dataset's metadata! The number of reviews per category is available by hovering over them.

#### Average rating of the games in our dataset<br>

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="200" src="/html/average_all.html"></iframe>
We can see that the average score of the games in our dataset is quite high. This probably due to the selection of popular games which tend to be well received.

#### Average rating of games received for free or not<br>

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="300" src="/html/recivedForFree.html"></iframe>
This was one of the most surprising results we got. Apparently, on average (for the games in our dataset at least), receiving the game for free doesn't make you want to recommend a game more.

#### Average rating of games played during early access or not<br>
"Early access" is a term meaning that the game can be purchased when the game is still being developed, usually at a smaller price, so that the player can provide feedback on what should be fixed or improved.
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="300" src="/html/early.html"></iframe>
From this result, we can see a small tendency to be more critical when a game is released in early access.
They usually have many bugs and sometimes, a game changes so much during development that early adopters of a game will not like how it is evolving. In protest, they may write "Not recommended" reviews.
This funding model also has some issues, as some game developers might take the money and abandon development, but those incidents are quite rare and did not occur for any of the games in our dataset.

#### Average rating from users that posted a certain number of reviews<br>

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="500" src="/html/numReview.html"></iframe>
Those results show that people will be more critical when posting more reviews until a certain point. One hypothesis would be that newcomers review their favourite games first, and then only after do they review the ones they dislike. This could be differentiated from "critics" (people who review many games), which probably have a more nuanced view on video games, and write reviews on all the games they play, regardless of whether they liked or not. 

#### Average rating from users that own a certain number of games<br>

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="500" src="/html/owned.html"></iframe>
We can see that the more someone owns games, the more likely they are to rate a game negatively. We can hypothesize that people that own a high amount of games are most likely avid players with plenty of video game experience. This might make them more critical of games, as, for each of the genres they like playing, they have much more games against which to compare.

#### Average rating from reviews with a certain number of words<br>

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="500" src="/html/words.html"></iframe>
The longer the reviews, the more critical they become. We also found that positive reviews had an average of 35 words, with the median length being 10 words.
For negative reviews, we have an average of 76 words with a median length of 28 words. It seems that when users do not recommend a game, they usually provide a detailed review on what they disliked about it; this is clearly visible in the results.

## Conclusion

We hope you enjoyed diving into this ocean of Steam reviews! This is a first step that needs more inputs from other fields such as linguistics, but the results so far are promising and we hope it gave you inspiration for further studies and maybe even write some <a href="https://en.wikipedia.org/wiki/Konami_Code" target="_blank">code</a>!

## References<br>

[^1]: [Kaggle, *Steam Reviews Dataset 2021*, 2021](https://www.kaggle.com/najzeko/steam-reviews-2021)
[^2]: [Matteo Pagliardini, Prakhar Gupta, Martin Jaggi, *Unsupervised Learning of Sentence Embeddings using Compositional n-Gram Features*, NAACL 2018](https://github.com/epfml/sent2vec)


<details>
<summary><b>Images</b></summary>
<ul>
<li>2017-11-11-170105.jpg (960×544). (2017, November 11). https://pixeladventurers.com/wp-content/uploads/2017/11/2017-11-11-170105.jpg</li>
<li>696436-arma-iii-windows-screenshot-moving-in-formation.jpg (1920×1080). (n.d.). Retrieved 8 June 2021, from https://www.mobygames.com/images/shots/l/696436-arma-iii-windows-screenshot-moving-in-formation.jpg</li>
<li>7551559D53AB49AE0A1D232EA6777497395A7F94 (640×480). (n.d.). Retrieved 8 June 2021, from https://steamuserimages-a.akamaihd.net/ugc/103980755826651042/7551559D53AB49AE0A1D232EA6777497395A7F94/</li>
<li>2953937288.jpg (500×710). (n.d.). Retrieved 8 June 2021, from https://cdn.startselect.com/production/products/images/4d59f/58605/2953937288.jpg</li>
<li>Growing up is hard to do in Night in the Woods. (n.d.). VideoGamer.Com. Retrieved 8 June 2021, from https://www.videogamer.com/features/growing-up-is-hard-to-do-in-night-in-the-woods</li>
<li>Isandstorm_09.jpg (1280×720). (n.d.). Retrieved 8 June 2021, from https://www.newgamenetwork.com/images/uploads/gallery/InsurgencySandstorm/isandstorm_09.jpg</li>
<li>Latest (320×240). (n.d.). Retrieved 8 June 2021, from https://static.wikia.nocookie.net/oneshot/images/d/da/OneShot_remake_title_card.png/revision/latest?cb=20170404075741</li>
<li>Media.jpg (1200×1090). (n.d.). Retrieved 8 June 2021, from https://media.melty.fr/article-4334256-ajust_1200/media.jpg</li>
<li>Moon-Pic-3.png (500×375). (n.d.). Retrieved 8 June 2021, from https://www.gbhbl.com/wp-content/uploads/2019/03/Moon-Pic-3.png</li>
<li>Oneshot_07.jpg (640×480). (n.d.). Retrieved 8 June 2021, from https://www.newgamenetwork.com/images/uploads/gallery/Oneshot/oneshot_07.jpg</li>
<li>Sierra’s Adventures in YA Literature: Image. (n.d.). Retrieved 8 June 2021, from https://sierraisalibrarian.files.wordpress.com/2020/10/nitw.jpg?w=662</li>
<li>To-the-moon-cover.cover_large.jpg (600×600). (n.d.). Retrieved 8 June 2021, from https://images.nintendolife.com/77811ede754b2/to-the-moon-cover.cover_large.jpg</li>
<li>Valhalla.jpg (800×800). (n.d.). Retrieved 8 June 2021, from https://thegamehoard.com/wp-content/uploads/2018/11/valhalla.jpg</li>
<li>Y9pNo0aCyzSO6yztekij-JH-wI1wev90a-bG_xm8RMuYEe32PI_sS5dA2sgpZKgJ-NE8QtUOzLwvmChd6UptmTswxI5FRK1Kz5HiQcc_QafkfG8 (362×512). (n.d.). Retrieved 8 June 2021, from https://lh3.googleusercontent.com/proxy/Y9pNo0aCyzSO6yztekij-JH-wI1wev90a-bG_xm8RMuYEe32PI_sS5dA2sgpZKgJ-NE8QtUOzLwvmChd6UptmTswxI5FRK1Kz5HiQcc_QafkfG8</li>
<li><div>Icons made by <a href="https://icon54.com/" title="Pixel perfect">Pixel perfect</a> from <a href="https://www.flaticon.com/" title="Flaticon">www.flaticon.com</a></div><li>
 </ul>
<br>
</details>
