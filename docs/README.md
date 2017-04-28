<?xml encoding="UTF-8" ?>
# Minimaps

The first minimap ever was seen in a 1981 video game called _Defender_, a horizontally scrolling shoot 'em up arcade game. It allowed the player to see what was going to be ahead of him before the screen showed the actual scene.

![Defender (1981)](https://qph.ec.quoracdn.net/main-qimg-bea9f0dbb6b54c45586c3df161395a3b)

As games grew bigger, minimaps turned to orientation aids located mainly on corners displaying very few features. They normally include player position, objectives and surrounding terrain; although allied and enemy structures are also very common. An example of a minimap full of features is the _World of Warcraft_ minimap:

![WoW Minimap Legend](http://img8.mmo.mmo4arab.com/wow/10_31/map_improvement/minimaptrackingmenu.jpg)![WoW Minimap](https://vignette2.wikia.nocookie.net/wowwiki/images/7/79/Minimap-4_1_0.png/revision/latest?cb=20110611053708)

* Zoom In & Out
* Mail
* Date
* Time
* Current Location
* Legend (to edit which markers are to be shown)
* Overworld Button
* Dungeon Queue State
* ...

***

## Other Features

### Overworld
An overworld is the opposite of a minimap: a _"bigmap"_. They usually fill the whole screen and show the player a complete map of how all levels are connected. The Legend of Zelda was the first console game to feature an overworld map. Showing a broad area that the player can explore ensures the player that he/she will have many ours of gameplay ahead.

### Layers
Games like the Age of Empires II minimap allow players to communicate through the minimap. They are common in team based multiplayer games showing marks, signals or lines. These features require minimaps to include a layering system for every type of marking.

### Fog of War
Minimaps offer information to the player that further allows him/her to have control over the surroundings. Some games prefer to give the player the chance to explore or just hide this information to place the player at a disadvantaged position. Fog of war also allows the game to not have to load and update assets until discovered by the player; therefore it can be a means for optimization instead of a plain feature for players.

***

## Coding

For a quick minimap programming guide lets make a simple example . In order for it to work we'll need a base atlas for your game's interface grafic resources from which we will load the background and the markers. We'll also need to know where the position of the desired units to be shown. Units shold be idealy in an Entity Manager type class for easy access.

The end result will be a small rectangle at the left-bottom corner showing us the player and the enemy units.

### Header

We'll manage our minimap through an separate class. It has:
* Load function with XML node for value loading
* Update function that detects mouse input and moves the camera when clicked
* Draw function that prints our markers onto the map
* UI_Image pointer for the background
* 2 rectangles we'll use to show our units that contain their altas' section



    // Minimap.h
    #include "PugiXml\src\pugixml.hpp"
    #include "SDL\include\SDL_rect.h"
    class UI_Image;
    class Minimap
    {
     public:
      Minimap();
      ~Minimap();
      UI_Image* Load(pugi::xml_node& conf);
      void Update();
      void Draw();
     private:
      UI_Image* base;
      SDL_Rect greenMark;
      SDL_Rect redMark;
    };

### TODO 1
Load a new minimap: new Minimap(); and call the minimap's Load(pugi::xml_node& conf) to load its resources.

### TODO 2
Load the minimap's resources.
First, load a new UI_Image* base using: App->gui->Add_element(UI_TYPE TYPE, j1Module* element_module)
Let the type be IMAGE and its listener App->game, although any module will work

Then, set the markers' values. It's best to load from the xml we are given, but for practical reasons use these values for now:


    greenMark = { 169, 519, 3, 3 };
    redMark   = { 173, 519, 3, 3 };

Then, set the background image (UI_Image* base):
Use UI_Image::Set_Image_Texture(SDL_Rect tex) to set the background's atlas section and UI_element::Set_Interactive_Box(SDL_Rect new_rect) to set the screen section to print.

Use the following values:


    SDL_Rect mapImgTex = { 0, 533, 193, 132 };
    int scale = App->win->GetScale();
    SDL_Rect mapIntBox = { (552 * scale) - (190 * scale), (448 * scale) - (132 * scale), 193 * scale,132 * scale };
	
    Finally, block base by changing the UI_element::draggable value to NO_SCROLL.


### TODO 3:
The previous function will return the UI_Image* it will use for the background.
Add the returned value to the HUD's UI_element* hud_screen as a new child:
AddChild(UI_element* new_child). You should be able to see a dark rectangle at the bottom-left corner.

Call the minimap Draw() function. Make sure to only call it if minimap is NOT nullptr!

### TODO 4
Print player in the minimap. Use Player* player = App->game->em->player; to get our player. The player's position is stored in its iPoint currentPos. We'll need the player's relative position in the map. For that, we'll use an fPoint to store the values. Example for the horizontal vaue:


    horizontal relPos = (horizontal position as a float) / (map width * tile width)


This will give us a decimal value < 1. By multiplying the minimap's width by the value will know where in the minimap it we'll have to be printed. Now we just need to add the minimap's x value to find the screen position.

Once we have both, we are ready to print. Use App->render->Blit(atlas, x, y, mark) to print. Set the mark to one of the previously loaded ones.


### TODO 5
Print enemies in the minimap. Use the same method as before, but this time iterate through an Entity list. App->game->em->GetEntities() will return the enemies in the scene. Use a different mark for the enemies.

### TODO 6
Call the minimap Update() function. Make sure to only call it if minimap is NOT nullptr!

In Update(), check if the mouse is inside the minimap. Use UI_element::Mouse_is_in(const iPoint& mouse_pos) to check.

### TODO 7
Check if the mouse button is pressed: KEY_DOWN || KEY_REPEAT Use App->input->GetMouseButtonDown(SDL_BUTTON_LEFT) to get the mouse state.

### TODO 8
Once the player is clicking the minimap, move the camera. We'll use the same method as for printing markers but backguards.


    horizontal relPos = (mouse x position - minimap x as a float) / (minimap width)


Find the relative position as a decimal < 1. By multiplying the map's width (map width * tile width) by the value we'll know where in the map the camera must be. Because the camera follows the player, for practical reasons we'll just move the player


***

## Further Mods

Being a separate class we can modify it to become an **Overworld Map**. Steps:
1. Increasw the image size to fill the screen
2. Set the HUD to onlly call the map draw when needed
3. Exclude enemy unit drawing
4. Exclude camera movement functionality

We can also set it up to be **rhomb shaped**. Steps:
1. Print the relative positions from an isometric point of view
2. Once we detect the mouse is inside the map, we just need to detect the mouse's relative position inside the minimap and move the camera acordingly.
2. Change mouse detection to double triangle detection:

Example:



    float sign (fPoint p1, fPoint p2, fPoint p3)
    {
      return (p1.x - p3.x) * (p2.y - p3.y) - (p2.x - p3.x) * (p1.y - p3.y);
    }

    bool PointInTriangle (fPoint pt, fPoint v1, fPoint v2, fPoint v3)
    {
      bool b1, b2, b3;

      b1 = sign(pt, v1, v2) < 0.0f;
      b2 = sign(pt, v2, v3) < 0.0f;
      b3 = sign(pt, v3, v1) < 0.0f;

      return ((b1 == b2) && (b2 == b3));
    }


The previous example shows a basic detection method, but quite inneficient. Click [here](http://stackoverflow.com/questions/2049582/how-to-determine-if-a-point-is-in-a-2d-triangle) to see different ways to detect if the mouse is inside the map. For an extensive algebra approach click [here](https://books.google.es/books?id=fSZEAAAAQBAJ&pg=PA160&lpg=PA160&dq=c%2B%2B+triangle+point+collision&source=bl&ots=O9s1htN7f4&sig=0885vsXQ3QFYQK0-2vdLtAXvhmo&hl=en&sa=X&ved=0ahUKEwjkt-uWkLzTAhVGVhQKHQvjBPwQ6AEIXzAI#v=onepage&q=c%2B%2B%20triangle%20point%20collision&f=false).



