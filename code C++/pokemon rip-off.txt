#include "DarkGDK.h"
#include "sncIOstream.h"
#include <string>
#include <fstream>
#include <iomanip>
using namespace std;

// CONSTANTS
const unsigned int BLACK    = dbRGB(0,0,0);
const unsigned int WHITE    = dbRGB(255,255,255);
const unsigned int BROWN    = dbRGB(123,75,0);
const unsigned int BLUE		= dbRGB(0,174,255);
const unsigned int DARKBLUE	= dbRGB(0,0,100);
const unsigned int GREEN	= dbRGB(29,152,26);

const int glumby			= 1;
const int fondy				= 2;
const int sippy				= 3;
const int glumby_mouseover	= 4;
const int fondy_mouseover	= 5;
const int sippy_mouseover	= 6;
const int char_background	= 7;

// all backgrounds
const int battleID00		= 10;
const int battleID01		= 11;
const int background00		= 20;
const int background01		= 21;
const int background_grass	= 22;
const int background_water	= 23;
const int intro_back0		= 24;
const int intro_back1		= 25;
const int intro_back2		= 26;
const int mapUR = 30;
const int mapUL = 31;
const int mapLL = 32;
const int mapLR = 33;

//LL
const int ll_grass			= 40;
const int ll_lake00			= 41;
const int ll_lake01			= 42;
const int ll_ai01_up		= 43;
const int ll_ai01_down		= 44;
//UL
const int ul_grass			= 50;
const int ul_tree_leaves	= 51;
//UR
// 60's
//LR
const int lr_water_splashes	= 70;

// buttons for intro
const int yesUp				= 80;
const int yesDown			= 81;
const int noUp				= 82;
const int noDown			= 83;

const int water00			= 200;
const int water01			= 201;
const int water02			= 202;
const int grass00			= 210;

// buttons for battle
const int button_up0		= 240;
const int button_down0		= 241;
const int button_up1		= 242;
const int button_down1		= 243;
const int button_up2		= 244;
const int button_down2		= 245;
const int button_up3		= 246;
const int button_down3		= 247;

const int test				= 300;
const int test2				= 301;

// Restricts the whole map to specific amount
const int MAX_OVER = 1;
int UP = 0;
int DOWN = 1;
int LEFT = 1;
int RIGHT = 0;
int prevx = 0;
int prevy = 0;
int initBattle = 0;
int prevCharacter;
int longg = 0;

// variables that need to be universal
string name				;
int character			= 0;
int ID					= 10;
int tempDrtn			= 2;
int univBack			= mapLL;
int tempUnivBack		= mapLL;
int battleUnivBack		= 210;
bool screenSwitch		= true;
bool onLake				= true;
bool onLake2			= true;
int battleMonster		= 0;
string battleMonsterName= "junk";
float monsterHealth		= -1;
float playerHealth		= -1;

	// struct setup for identity and monster
	struct players
	{
		int level;
		float health;
		string pName;
		int experience;
	};
	struct monster
	{
		string type;
		int level;
		float health;
		string name; 
	};
	struct moves
	{
		string name;
		int damage;
		int changeDamage;
	};

	players player;
	monster monData;

	// All Moves
	moves riptide;
	moves half_slash;
	moves Hay_Wire;
	moves powerThirst;

void displayCurrent(int x , int y);
void AI(int &count , int &AIx , int &AIy , int &x , int &y);
void setupMoves();
void healthBar();
void battleSceneButtons(bool & , bool & , int & , int & , bool &);
void battleSceneData(bool & , int &);
void loadCharacter();
void getMonster();
void saveCharacter();
void onScreenStats(int , int , bool&);
void setupSpriteData();
void backToMap();
void battleScene(int);
void characterChange();
void BackgroundSprites();
void setupSprites();
bool sncOnSprite(int , int , int);
void setupChar();
void ChangeBackground(int &, int &, string &);
void hittingObject(int , int , string &);
bool ObjectHit_button(int , int , int);
void SpriteMove(int , int , string &);
void ControlSprite(int &, int &);
void tutorial(bool & , int x , int y);
void introButtons(bool & , bool & , bool & , bool &);
void setup();
void DarkGDK()
{
	dbRandomize(dbTimer());
	int x = 320;
	int y = 240;
	int count = 0;
	int AIx = 0;
	int AIy = 0;
	string box = "c";
	int character = 0;
	bool stats = true;
	bool tutorialOn = true;

	setup();
	setupMoves();
	setupSpriteData();
	setupChar();

	dbSyncOn();
	dbSyncRate(60);

	while (!dbEscapeKey())
	{
		if (screenSwitch == true) // background changes
			BackgroundSprites(); // change the sprite layout of the background
		ControlSprite(x , y);

		SpriteMove(x , y , box);
		ChangeBackground(x , y , box);

		if (tutorialOn == true)
			tutorial(tutorialOn , x , y);
		
		// shows numerical values on screen
		onScreenStats(x , y , stats);

		// all moving/interactive characters on map
		AI(count , AIx , AIy , x , y);

		if (dbControlKey())
			saveCharacter();
		
		dbSync();
	}

	dbWaitKey();
	return;
}

void setup()
{
	dbSetImageColorKey(0 , 255 , 0); // green

	dbLoadImage("glumby.bmp", glumby);
	dbLoadImage("glumby_mouseover.bmp", glumby_mouseover);
	dbLoadImage("fondy.bmp", fondy);
	dbLoadImage("fondy_mouseover.bmp", fondy_mouseover);
	dbLoadImage("sippy.bmp", sippy);
	dbLoadImage("sippy_mouseover.bmp", sippy_mouseover);
	dbLoadImage("char_background.bmp", char_background);
	//dbLoadImage("background00.bmp", background00);
	//dbLoadImage("background01.bmp", background01);
	dbLoadImage("battle_grass.bmp", background_grass);
	dbLoadImage("battle_water.bmp", background_water);
	dbLoadImage("intro_back0.bmp", intro_back0);
	dbLoadImage("intro_back1.bmp", intro_back1);
	dbLoadImage("intro_back2.bmp", intro_back2);
	dbLoadImage("UR.bmp", mapUR);
	dbLoadImage("UL.bmp", mapUL);
	dbLoadImage("LL.bmp", mapLL);
	dbLoadImage("LR.bmp", mapLR);

	// buttons for intro
	dbLoadImage("yes_up.bmp", yesUp);
	dbLoadImage("yes_down.bmp", yesDown);
	dbLoadImage("no_up.bmp", noUp);
	dbLoadImage("no_down.bmp", noDown);

	// monsters
	dbLoadImage("loo.bmp", water00);
	dbLoadImage("loosoo.bmp", water01);
	dbLoadImage("loosoofi.bmp", water02);
	dbLoadImage("grint.bmp", grass00);

	// map sprites
	dbLoadImage("LL_grass.bmp", ll_grass);
	dbLoadImage("LL_lake_sprite0.bmp", ll_lake00);
	dbLoadImage("LL_lake_sprite1.bmp", ll_lake01);
	dbLoadImage("LLai_up.bmp", ll_ai01_up);
	dbLoadImage("LLai_down.bmp", ll_ai01_down);

	dbLoadImage("UL_grass.bmp", ul_grass);
	dbLoadImage("UL_tree_leaves.bmp", ul_tree_leaves);

	dbLoadImage("LR_water_splashes.bmp", lr_water_splashes);
	
	// buttons for battles
	dbLoadImage("button_up.bmp", button_up0);
	dbLoadImage("button_down.bmp", button_down0);
	dbLoadImage("button_up.bmp", button_up1);
	dbLoadImage("button_down.bmp", button_down1);
	dbLoadImage("button_up.bmp", button_up2);
	dbLoadImage("button_down.bmp", button_down2);
	dbLoadImage("button_up.bmp", button_up3);
	dbLoadImage("button_down.bmp", button_down3);


	dbLoadImage("box.bmp", test);
	dbLoadImage("Apple.bmp", test2);

	return;
}

void setupChar()
{
	string newCharacter;
	bool save = true;
	bool newChar = true;
	bool clicked1 = false;
	bool clicked2 = false;
	// background image
	dbPasteImage(intro_back2 , 0 , 0 , intro_back2);
	dbSetTextSize(60);

	// gets user to input 'newChar' and 'save' using buttons
	introButtons(newChar , save , clicked1 , clicked2);

	dbCLS();

	if (newChar == true)
	{
		int mouseX = dbMouseX();
		int mouseY = dbMouseY();

		bool clicked = false;
		while (clicked == false && !dbEscapeKey())
		{
			dbSprite(intro_back1 , 0 , 0 , intro_back1);

			// get mouse coordinates
			mouseX = dbMouseX();
			mouseY = dbMouseY();

			// standalone character images
			if (ObjectHit_button(mouseX , mouseY , glumby) == false){
				dbDeleteSprite(glumby_mouseover);
				dbSprite(glumby , 110 , 310 , glumby);}
			// rollover buttons
			if (ObjectHit_button(mouseX , mouseY , glumby) == true){
				dbDeleteSprite(glumby);
				dbSprite(glumby_mouseover , 110 , 310 , glumby_mouseover);
				if (dbMouseClick()){ // if you click on the image, exit while loop
					clicked = true;
					prevCharacter = glumby;
					character = glumby;}
			}

			if (ObjectHit_button(mouseX , mouseY , fondy) == false){
				dbDeleteSprite(fondy_mouseover);
				dbSprite(fondy , 265 , 310 , fondy);}
			if (ObjectHit_button(mouseX , mouseY , fondy) == true){
				dbDeleteSprite(fondy);
				dbSprite(fondy_mouseover , 265 , 310 , fondy_mouseover);
				if (dbMouseClick()){
					clicked = true;
					prevCharacter = fondy;
					character = fondy;}
			}

			if (ObjectHit_button(mouseX , mouseY , sippy) == false){
				dbDeleteSprite(sippy_mouseover);
				dbSprite(sippy , 420 , 310 , sippy);}
			if (ObjectHit_button(mouseX , mouseY , sippy) == true){
				dbDeleteSprite(sippy);
				dbSprite(sippy_mouseover , 420 , 310 , sippy_mouseover);
				if (dbMouseClick()){
					clicked = true;
					prevCharacter = sippy;
					character = sippy;}
			}
		
			// Next 5 lines assign priorities to needed sprites
			int i;
			const int max = 11;
			int tempArray[max] = {intro_back1, char_background, ll_grass, ll_lake00, ll_lake01, glumby, fondy, sippy, glumby_mouseover, fondy_mouseover, sippy_mouseover};
			for (i=0 ; i<max ; i++)
				dbSetSpritePriority(tempArray[i], i+1);
		
			dbSync();
		}
	}
	
	if (newChar == false)
		loadCharacter();

	dbSetTextSize(11);

	//clear screen
	dbDeleteSprite(intro_back1);
	dbDeleteSprite(glumby);
	dbDeleteSprite(fondy);
	dbDeleteSprite(sippy);
	dbDeleteSprite(glumby_mouseover);
	dbDeleteSprite(fondy_mouseover);
	dbDeleteSprite(sippy_mouseover);
	dbCLS();

	// to save or not save sprites
	if (save == true)
		saveCharacter();

	return;
}

void introButtons(bool &newChar , bool &save , bool &clicked1 , bool &clicked2)
{

	int mouseX = dbMouseX();
	int mouseY = dbMouseY();

	while (!dbEscapeKey() && clicked1 == false)
	{
		mouseX = dbMouseX();
		mouseY = dbMouseY();
		
		dbSetTextSize(60);
		dbSprite(yesUp , 179 , 150 , yesUp);
		if (ObjectHit_button(mouseX , mouseY , yesUp) == false){
			dbDeleteSprite(yesDown);
			dbSprite(yesUp , 179 , 150 , yesUp);}
		if (ObjectHit_button(mouseX , mouseY , yesUp) == true){
			dbDeleteSprite(yesUp);
			dbSprite(yesDown , 179 , 150 , yesDown);
			if (dbMouseClick()){
				clicked1 = true;
				newChar = true;}
		}
		dbSprite(noUp , 320 , 150 , noUp);
		if (ObjectHit_button(mouseX , mouseY , noUp) == false){
			dbDeleteSprite(noDown);
			dbSprite(noUp , 320 , 150 , noUp);}
		if (ObjectHit_button(mouseX , mouseY , noUp) == true){
			dbDeleteSprite(noUp);
			dbSprite(noDown , 320 , 150 , noDown);
			if (dbMouseClick()){
				clicked1 = true;
				newChar = false;}
		}

		dbPasteImage(intro_back2 , 0 , 0 , intro_back2);
		sncCenterText(320 , 30 , "New Character?");

		dbSync();
	}
	while(dbMouseClick()){}
	dbDeleteSprite(yesUp);
	dbDeleteSprite(yesDown);
	dbDeleteSprite(noUp);
	dbDeleteSprite(noDown);

	while (!dbEscapeKey() && clicked2 == false && newChar == true)
	{
		mouseX = dbMouseX();
		mouseY = dbMouseY();

		dbSprite(yesUp , 179 , 200 , yesUp);
		if (ObjectHit_button(mouseX , mouseY , yesUp) == false){
			dbDeleteSprite(yesDown);
			dbSprite(yesUp , 179 , 200 , yesUp);}
		if (ObjectHit_button(mouseX , mouseY , yesUp) == true){
			dbDeleteSprite(yesUp);
			dbSprite(yesDown , 179 , 200 , yesDown);
			if (dbMouseClick()){
				clicked2 = true;
				save = true;}
		}
		dbSprite(noUp , 320 , 200 , noUp);
		if (ObjectHit_button(mouseX , mouseY , noUp) == false){
			dbDeleteSprite(noDown);
			dbSprite(noUp , 320 , 200 , noUp);}
		if (ObjectHit_button(mouseX , mouseY , noUp) == true){
			dbDeleteSprite(noUp);
			dbSprite(noDown , 320 , 200 , noDown);
			if (dbMouseClick()){
				clicked2 = true;
				save = false;}
		}

		dbPasteImage(intro_back2 , 0 , 0 , intro_back2);
		sncCenterText(320 , 80 , "Save Player?");

		dbSync();
	}

	while(dbMouseClick()){}
	dbDeleteSprite(yesUp);
	dbDeleteSprite(yesDown);
	dbDeleteSprite(noUp);
	dbDeleteSprite(noDown);
	
	// Entering and displaying players name
	if (newChar == true)
	{
		dbPasteImage(intro_back0 , 0 , 0 , intro_back0);
		cout << endl << endl;
		cin >> name;
	}

	return;
}

void setupMoves()
{
	// attacks
	riptide.name = "riptide";
	riptide.damage = 10;

	half_slash.name = "half-slash";
	//half_slash.damage = 25;

	Hay_Wire.name = "Hay Wire";
	Hay_Wire.damage = 0;

	powerThirst.name = "power thirst";
	powerThirst.changeDamage = 15;

	return;
}

// starts monster and player data
void setupSpriteData()
{
	int i = 0;

	//identity player;
	player.pName = name;
	player.level = 1;
	player.health = 100;
	player.experience = 0;
	playerHealth = 100;

	monData.level = 1;
	monData.health = 1;

	return;
}

// tutorial, calls displayCurrent
void tutorial(bool &tutorialOn , int x , int y)
{
	int waitOn = 0;
	int flash = 0;

	while (tutorialOn == true)
	{
		if (dbSpaceKey())
			tutorialOn = false;

		// shows what you would normally see on screen in normal while loop
		displayCurrent(x , y);

		// tutorial stuff
		if (flash > 15 && flash < 101)
		{
			dbInk(BLACK, 0);
			sncCenterText(480 , 2 , "<----- Controls");
			dbBox(320-25 , 240+26 , 320 - 25 + 50 , 240+27);
			sncCenterText(450 , 207 , "<---- Name");
			sncCenterText(454 , 222 , "<---- Level");
			dbSetTextSize(20);
			sncCenterText(330 , 270 , "^");
			dbSetTextSize(10);
			sncCenterText(330 , 275 , "|");
			sncCenterText(417 , 286 , "----- Experience bar");

			flash++;
		}
		if (flash > 100)
				flash = 0;
		flash++;

		
		dbInk(WHITE, 0);
		dbSetTextSize(30);
		if (waitOn > 200)
		{
			dbBox(130 , 115 , 510 , 118);
			sncCenterText(320 , 80 , "Press spacebar to start.");
		}
		dbSetTextSize(10);

		waitOn++;

		dbSync();
	}
	return;
}

void displayCurrent(int x , int y)
{
	int temp = 0;

	do
	{
		dbPasteImage(mapLL , 0 , 0 , mapLL);

		// player stats
		dbInk(DARKBLUE, 0);
		sncCenterText(x+40 , y-33 , name);
		sncCenterText(x+40 , y-18 , player.level);
		sncCenterText(630 , 0 , initBattle);
		dbInk(BLUE, 0);
		dbBox(x-25 , y+23 , x - 25 + (player.experience/2) , y+26);
		dbInk(BLACK, 0);
		dbBox(x-27 , y+23 , x-26 , y+26);
		dbBox(x+26 , y+23 , x+27 , y+26);

		// character
		dbSizeSprite(character, 40, 40);
		dbSprite(character , x , y , character);
		dbOffsetSprite(character , 20 , 20);
	
		dbInk(WHITE, 0);
		sncCenterText(220 , 2 , "STATS: spacebar    SAVE: control    EXIT: escape");

		dbSync();
	}while(temp == 1);

	return;
}

void loadCharacter()
{
	string a;
	string b;
	int count = 0;
	int i = 0;
	char ch;

	dbPasteImage(intro_back2 , 0 , 0 , intro_back2);
	dbSetTextSize(50);
	sncCenterText(320 , 80 , "Enter Character's Name");
	string fileName;
	cout << endl << endl << endl;
	cin >> fileName;
	name = fileName;
	fileName = fileName + ".txt";

	ifstream tf;

	tf.open(fileName.c_str());
	if (tf.fail())
	{
		cout << "Cannot find file" << endl;
		cout << "press any key to exit" << endl;
		dbWaitKey();
		return;
	}
	
	//char next;

	// all thanks to http://stackoverflow.com/users/596781/kerrek-sb
	string line;
	int cnt = 0;
	while (getline(tf, line))
	{
		istringstream iss(line);
		int a;
		if (!(iss >> a)) { break; } // error
		
		if (cnt == 0)
			character = a;
		if (cnt == 1)
			player.level = a;
		if (cnt == 2)
			player.experience = a;

		cnt++;
	}

	tf.close();

	return;
}

void saveCharacter()
{
	ofstream fin;
	fin.open(name + ".txt");
	if (fin.fail())
	{
		cout << "Cannot find file" << endl;
		cout << "press any key to exit" << endl;
		dbWaitKey();
		return;
	}
	fin << character;
	fin << "\n";
	fin << player.level;
	fin << "\n";
	fin << player.experience;

	sncCenterText(320 , 240 , "Saved");
	dbWait(1200);

	fin.close();
	return;
}

// changes character sprite image in grass/water/winning an award (bands on arm)
void characterChange()
{
	if (character == glumby)
	{
		dbDeleteSprite(glumby);
		character = fondy;
	}
	// need a set number of characters
	// swimming, grassing     // bands around arms after defeating an arena or something
	return;
}

// moving the character
void ControlSprite(int &x, int &y)
{
	dbSizeSprite(character, 40, 40);

	prevx = x;
	prevy = y;

	if (dbLeftKey()){
		if(tempDrtn == 2)
			dbMirrorSprite(character);
		x -= 2;
		tempDrtn = 1;
	}
	if (dbRightKey()){
		if(tempDrtn == 1)
			dbMirrorSprite(character);
		x += 2;
		tempDrtn = 2;
	}
	if (dbDownKey()){
		y += 2;
	}
	if (dbUpKey()){
		y -= 2;
	}
	
	return;
}

// moves character, calls hittingObject
void SpriteMove(int x, int y, string &box)
{
	// move glumby
	dbSprite(character , x , y , character);
	dbOffsetSprite(character , 20 , 20);

	hittingObject( x,  y, box);
	
	return;
}

// checks if the character is supposed to do something special (i.e. go into a battle, change background)
void hittingObject(int x, int y, string &box)
{
	int i;
	int temp = 0;
	const int size = 3;
	int ID[size] = {ll_grass, ll_lake00, ll_lake01};

	// finds out what monster you should be fighting if you get in a battle
	getMonster();

	for (i=0 ; i<size ; i++)
	{
		// (prevx != x || prevy != y) makes it so the random number generator only occurs while moving
		if (dbSpriteCollision(character , ID[i]) && (prevx != x || prevy != y))
		{
			initBattle = dbRND(150);
			if (initBattle == 1)
			{
				box = 'a';
			}
			temp = 1;
		}

		//if (temp == 1)
		//{
		//	character = fondy;
		//}
		//if (temp == 2)
		//{
		//	character = glumby;
		//}

		//if (dbSpriteCollision(character , lake_sprite0) || dbSpriteCollision(character , lake_sprite1) && onLake == true)
		//{
		//	onLake = false;
		//	dbDeleteSprite(character);
		//	character = test;
		//}
		//if (!dbSpriteCollision(character , lake_sprite0) || !dbSpriteCollision(character , lake_sprite1))
		//{
		//	onLake = true;
		//	dbDeleteSprite(character);
		//	character = prevCharacter;
		//}
	/*		if ((ID[i] == lake_sprite0 || ID[i] == lake_sprite1) && onLake == true)
			{
				dbDeleteSprite(character);
				character = test;
				onLake = false;}
			if ((ID[i] == lake_sprite0 || ID[i] == lake_sprite1))
				onLake2 = true;
			else
				onLake2 = false;

			if (onLake2 == false)
			{	
				longg++;
				dbSprite(test2 , (longg) * 20 , 200 , test2);
				dbDeleteSprite(character);			
				character = prevCharacter;
			}
	*/

			//onLake = true;
			//if (ID[i] != lake_sprite0 || ID[i] != lake_sprite1)
	}

	return;
}

// check if button is hit (only used for the intro)
bool ObjectHit_button(int x, int y , int ID)
{
	// Get the coordinates of the sprite's upper-left corner.
	// The insertion point may have been moved by (OffsetX,OffsetY).
    int lowerX = dbSpriteX(ID) - dbSpriteOffsetX(ID);
    int lowerY = dbSpriteY(ID) - dbSpriteOffsetY(ID);

    // Get the coordinates of the sprite's lower-right corner.
    int upperX = lowerX + dbSpriteWidth(ID);
    int upperY = lowerY + dbSpriteHeight(ID);

    // (x,y) inside or outside sprite?
	if (x >= lowerX && x <= upperX && y >= lowerY && y <= upperY)
	{
		return true;
	}
	else
		return false;
}

void ChangeBackground(int &x , int &y , string &box)
{	
	tempUnivBack = univBack;

	// battle scene backgrounds
	if (box == "a"){
		battleScene(battleMonster);
		box = "c";

		// gets screen back to normal un-battle stuff
		backToMap();
	}

	//change map location and check to see if at edge of map
	if (x > 640 && RIGHT == 0){
		x = 0;
		screenSwitch = false;
		if (univBack == mapUL){
			univBack = mapUR;
			screenSwitch = true;}
		if (univBack == mapLL){
			univBack = mapLR;
			screenSwitch = true;}
		box = "c";
		LEFT--;
		RIGHT++;
		if (RIGHT >= 2)
			RIGHT = 1;
	}
	if (x < 0 && LEFT == 0){
		x = 640;
		if (univBack == mapUR){
			univBack = mapUL;
			screenSwitch = true;}
		if (univBack == mapLR){
			univBack = mapLL;
			screenSwitch = true;}
		box = "c";
		RIGHT--;
		LEFT++;
		if (LEFT >= 2)
			LEFT = 1;
	}
	if (y > 480 && DOWN == 0){
		y = 0;
		if (univBack == mapUL){
			univBack = mapLL;
			screenSwitch = true;}
		if (univBack == mapUR){
			univBack = mapLR;
			screenSwitch = true;}
		box = "c";
		UP--;
		DOWN++;
		if (DOWN >= 2)
			DOWN = 1;
	}
	if (y < 0 && UP == 0){
		y = 480;
		if (univBack == mapLL){
			univBack = mapUL;
			screenSwitch = true;}
		if (univBack == mapLR){
			univBack = mapUR;
			screenSwitch = true;}
		box = "c";
		DOWN--;
		UP++;
		if (UP >= 2)
			UP = 1;
	}
	if (x > 640 && RIGHT == 1)
		x = 640;
	if (x < 0 && LEFT == 1)
		x = 0;
	if (y > 480 && DOWN == 1)
		y = 480;
	if (y < 0 && UP == 1)
		y = 0;

	dbPasteImage(univBack , 0 , 0 , univBack);

	return;
}

void AI(int &count , int &AIx , int &AIy , int &x , int &y)
{
	bool collision = true;
	int touchingCount = 0;

	if (count > 50 && count < 150)
		AIx++;
	else if (count > 300 && count < 420)
		AIy--;
	else if (count > 700 && count < 800)
		AIx--;
	else if (count > 950 && count < 1070)
		AIy++;

	count++;

	// LL crossed eyed thing
	if (count > 1200)
		count = 0;
	if ((count/10)%2 == 0){
		dbDeleteSprite(ll_ai01_down);
		dbSprite(ll_ai01_up , 400 + AIx , 250 + AIy , ll_ai01_up);}
	else{
		dbDeleteSprite(ll_ai01_up);
		dbSprite(ll_ai01_down , 400 + AIx , 250 + AIy , ll_ai01_down);}

	// LL crossed eyed thing
	if (dbSpriteCollision(character , ll_ai01_down) && collision == true)
	{
		dbDeleteSprite(ll_ai01_up);
		dbDeleteSprite(ll_ai01_down);
		while (!dbSpaceKey())
		{
			displayCurrent(x , y);
			dbInk(WHITE, 0);
			dbBox (400 + AIx - 70 , 250 + AIy - 48 , 400 + AIx + 110 , 250 + AIy - 15);
			dbInk(BLACK, 0);
			dbSetTextSize(10);
			
			dbDeleteSprite(ll_ai01_down);
			dbDeleteSprite(ll_ai01_up);

			sncCenterText(420 + AIx , 250 + AIy - 47 , "So you want to know");
			sncCenterText(420 + AIx , 250 + AIy - 32 , "how to find monsters?");
			
			dbDeleteSprite(ll_ai01_up);
			dbDeleteSprite(ll_ai01_down);

			if ((touchingCount/10)%2 == 0)
			{
				dbSprite(ll_ai01_up , 400 + AIx , 250 + AIy - 3 , ll_ai01_up);
			}
			else if ((touchingCount/10)%2 != 0)
			{
				dbSprite(ll_ai01_down , 400 + AIx , 250 + AIy + 3 , ll_ai01_down);
			}

			touchingCount++;

			dbSync();
		}
		while (dbSpaceKey()){}
		while (!dbSpaceKey())
		{
			displayCurrent(x , y);
			dbInk(WHITE, 0);
			dbBox (400 + AIx - 70 , 250 + AIy - 48 , 400 + AIx + 110 , 250 + AIy - 15);
			dbInk(BLACK, 0);
			dbSetTextSize(10);
			
			dbDeleteSprite(ll_ai01_down);
			dbDeleteSprite(ll_ai01_up);

			sncCenterText(420 + AIx , 250 + AIy - 47 , "Just take a stroll in");
			sncCenterText(420 + AIx , 250 + AIy - 32 , "the grass or water!");
			
			dbDeleteSprite(ll_ai01_up);
			dbDeleteSprite(ll_ai01_down);

			if ((touchingCount/10)%2 == 0)
			{
				dbSprite(ll_ai01_up , 400 + AIx , 250 + AIy - 3 , ll_ai01_up);
			}
			else if ((touchingCount/10)%2 != 0)
			{
				dbSprite(ll_ai01_down , 400 + AIx , 250 + AIy + 3 , ll_ai01_down);
			}

			touchingCount++;

			dbSync();
		}

		while (dbSpriteCollision(character , ll_ai01_down))
		{
			dbDeleteSprite(character);
			ControlSprite(x , y);
			dbSprite(character , x , y , character);
			dbOffsetSprite(character , 20 , 20);
			dbSync();
		}
		collision = false;
	}

	return;
}

// sets up all interactive sprites on map
void BackgroundSprites()
{
	// deletes sprites
	const int size = 6;
	int ID[size] = {ll_grass, ll_lake00, ll_lake01, ul_grass, ul_tree_leaves, lr_water_splashes};

	for (int i=0 ; i<size ; i++)
		dbDeleteSprite(ID[i]);

	if (univBack == mapLL)
	{
		dbSprite(ll_grass , 320 , 342 , ll_grass);
		dbSprite(ll_lake00 , 97 , 80 , ll_lake00);
		dbSprite(ll_lake01 , 83 , 123 , ll_lake01);
	}
	if (univBack == mapUL)
	{
		dbSprite(ul_grass, 0 , 0 , ul_grass);
		dbSprite(ul_tree_leaves , 0 , 0 , ul_tree_leaves);
	}
	if (univBack == mapUR)
	{
	}
	if (univBack == mapLR)
	{
		dbSprite(lr_water_splashes, 0 , 0 , lr_water_splashes);
	}

	return;
}

// stats dipslayed while not in battle
void onScreenStats(int x , int y , bool &stats)
{
	sncCenterText(220 , 2 , "STATS: spacebar    SAVE: control    EXIT: escape");
	if (dbSpaceKey())
	{
		if (stats == false)
			stats = true;
		else
			stats = false;
	}
	while (dbSpaceKey()){}

	if (stats == true)
	{
		dbInk(DARKBLUE, 0);
		sncCenterText(x+40 , y-33 , name);
		sncCenterText(x+40 , y-18 , player.level);
		sncCenterText(630 , 0 , initBattle);
	
		dbInk(BLUE, 0);
		dbBox(x-25 , y+23 , x - 25 + (player.experience/2) , y+26);
		dbInk(BLACK, 0);
		dbBox(x-27 , y+23 , x-26 , y+26);
		dbBox(x+26 , y+23 , x+27 , y+26);

		dbInk(WHITE, 0);
	}
	return;
}

// calls battleSceneData
void battleScene(int battleMonster)
{
	// deletes any AI on screen
	dbDeleteSprite(ll_ai01_down);
	dbDeleteSprite(ll_ai01_up);

	// displays FIGHT text
	int i = 0;
	int j = 0;
	bool done = false;
	bool clicked = false;
	// movesLeft is the total remaining for all moves
	int movesLeft = 5;

	bool battle = true;
	// delete all previous sprites
	const int size = 6;
	int ID[size] = {ll_grass, ll_lake00, ll_lake01, ul_grass, ul_tree_leaves, lr_water_splashes};
	for (int i=0 ; i<size ; i++)
		dbDeleteSprite(ID[i]);
	
	dbSetTextSize(200);
	dbInk(BROWN, 0);

	while (done == false)
	{
		dbPasteImage(univBack , 0 , 0 , univBack);
		if (i%2 == 0)
			sncCenterText(320+i , 190 , "FIGHT");
		else
			sncCenterText(320-i , 190 , "FIGHT");

		if (i > 150)
			done = true;
			
		i++;
		dbSync();
	}
	dbInk(WHITE, 0);

	univBack = battleUnivBack;

	while (clicked == false)
	{
		battleSceneData(clicked , movesLeft);
		dbSync();
	}

	// checks if player and monster died
	if (player.health <= 0 && monData.health <= 0)
		player.level -= 1;
	// checks if player out of moves but killed monster
	else if (movesLeft == 0 && monData.health <= 0)
		player.experience += 25;
	// checks if player died and subtracts 1 from level if true
	else if (player.health <= 0 || movesLeft <= 0)
		player.level -= 1;
	else
		player.experience += 25;
	player.health = 100;

	if (player.experience >= 100)
	{
		player.level += 1;
		player.experience = 0;
	}

	return;
}

// finds terrain and monster and assigns struct values to monster
void getMonster()
{
	dbRandomize(dbTimer());

	int rndEnemy = dbRND(2);
	
	tempUnivBack = univBack;
	if (dbSpriteCollision(character , ll_grass))
	{
		battleUnivBack = background_grass;

		battleMonster		= grass00;
		battleMonsterName	= "grint";
		monData.type		= "grass";
		monData.health		= 50;
		monsterHealth		= 50;
		monData.level		= 5;
	}
	
	else if (dbSpriteCollision(character , ll_lake00) || dbSpriteCollision(character , ll_lake01))
	{
		battleUnivBack = background_water;
		if (rndEnemy == 0){
			battleMonsterName	= "loo";
			battleMonster		= water00;
			monData.type		= "water";
			monData.health		= 30;
			monsterHealth		= 30;
			monData.level		= 2;
		}
		if (rndEnemy == 1){
			battleMonsterName	= "loosoo";
			battleMonster		= water01;
			monData.type		= "water";
			monData.health		= 50;
			monsterHealth		= 50;
			monData.level		= 4;
		}
		if (rndEnemy == 2){
			battleMonsterName	= "loosoofi";
			battleMonster		= water02;
			monData.type		= "water";
			monData.health		= 70;
			monsterHealth		= 70;
			monData.level		= 7;
		}
	}
	return;
}

// sets all variable back to the un-battle map
void backToMap()
{
	univBack = tempUnivBack;
	if (tempUnivBack == mapLL){
		UP = 0;
		DOWN = 1;
		RIGHT = 0;
		LEFT = 1;
	}
	if (tempUnivBack == mapUL){
		UP = 1;
		DOWN = 0;
		RIGHT = 0;
		LEFT = 1;
	}
	if (tempUnivBack == mapUR){
		UP = 1;
		DOWN = 0;
		RIGHT = 1;
		LEFT = 0;
	}
	if (tempUnivBack == mapLR)
	{
		UP = 0;
		DOWN = 1;
		RIGHT = 1;
		LEFT = 0;
	}

	// delete all sprites associated with battling
	const int size = 12;
	int ID[size] = {grass00 , water00 , water01 , water02 , button_up0 , button_down0 , button_up1 , button_down1 ,
		button_up2 , button_down2 , button_up3 , button_down3};
	for (int i=0 ; i<size ; i++)
		dbDeleteSprite(ID[i]);

	return;
}

// sprites and text in battle, checks for player or monster < 0 health
// also calls battleSceneButtons and healthBar inside
void battleSceneData(bool &clicked , int &movesLeft)
{
	// checks if the specific button was pressed
	bool changeDamage = false;
	// playDamage is the variable that gets 
	int playerDamage = 0;
	// checks if you clicked on an attack
	bool attacked = false;

	int mouseX = 0;
	int mouseY = 0;

	while (monData.health > 0 && player.health > 0 && movesLeft > 0)
	{
		// displays sprites
		dbSprite(battleMonster , 425 , 45 , battleMonster);
		dbSprite(character , 10 , 240 , character);
		dbSizeSprite(battleMonster, 200, 200);
		dbSizeSprite(character, 300, 300);

		// background
		dbPasteImage(univBack , 0 , 0 , univBack);

		// displays fighting buttons
		battleSceneButtons(clicked , attacked , playerDamage , movesLeft , changeDamage);
		
		// keeping track of health for monster and player
		dbSetTextSize(20);
		if (attacked == true)
		{
			monData.health -= playerDamage;
			attacked = false;
			while (dbMouseClick()){}

			int monsterDamage = dbRND(30);
			player.health -= monsterDamage;
		}
		//player.health -= monsterDamage;
		healthBar();

		// displays the names of the moves
		dbSetTextSize(20);
		sncCenterText(425 , 309 , riptide.name);
		sncCenterText(425 , 342 , half_slash.name);
		sncCenterText(565 , 309 , Hay_Wire.name);
		sncCenterText(565 , 342 , powerThirst.name);
		dbSetTextSize(40);

		// displays color of monster type
		if (monData.type == "water")
			dbInk(BLUE, 0);
		if (monData.type == "grass")
			dbInk(GREEN, 0);
		else
			dbInk(BROWN, 0);
		dbBox(320 , 367 , 382 , 397);
		dbInk(BLACK, 0);
		sncCenterText(348 , 367 , "IDK%");

		// back to to normal white text
		dbInk(WHITE, 0);

		// displays the names and levels of monster and player
		dbSetTextSize(50);
		sncCenterText(130 , 8 , battleMonsterName);
		sncCenterText(505 , 371 , name);
		dbSetTextSize(40);
		sncCenterText(75 , 70 , monData.level);
		sncCenterText(608 , 433 , player.level);
		dbSetTextSize(11);
		
		dbSync();
	}
	return;
}

// makes the buttons for a battle interactive
void battleSceneButtons(bool &clicked , bool &attacked , int &playerDamage , int &movesLeft , bool &changeDamage)
{
	int fakePlayerDamage = 0;
	// get mouse coordinates
	int mouseX = dbMouseX();
	int mouseY = dbMouseY();
	
	// UL
	if (ObjectHit_button(mouseX , mouseY , button_up0) == false){
		dbDeleteSprite(button_down0);
		dbSprite(button_up0 , 353 , 300 , button_up0);}
	// rollover buttons
	if (ObjectHit_button(mouseX , mouseY , button_up0) == true){
		dbDeleteSprite(button_up0);
		dbSprite(button_down0 , 353 , 300 , button_down0);
		fakePlayerDamage = riptide.damage;
		if (dbMouseClick()){ // if you click on the image, exit while loop
			clicked = true;
			attacked = true;
			playerDamage = riptide.damage;
			if (changeDamage == true){
				playerDamage += powerThirst.changeDamage;
				changeDamage = false;}
			movesLeft -= 1;}
	}
	// UR
	if (ObjectHit_button(mouseX , mouseY , button_up1) == false){
		dbDeleteSprite(button_down1);
		dbSprite(button_up1 , 495 , 300 , button_up1);}
	if (ObjectHit_button(mouseX , mouseY , button_up1) == true){
		dbDeleteSprite(button_up1);
		dbSprite(button_down1 , 495 , 300 , button_down1);
		sncCenterText(492 , 265 , "Damage is random 0-20");
		if (dbMouseClick()){
			clicked = true;
			attacked = true;
			Hay_Wire.damage = dbRND(20);
			playerDamage = Hay_Wire.damage;
			if (changeDamage == true){
				playerDamage += powerThirst.changeDamage;
				changeDamage = false;}
			movesLeft -= 1;}
	}
	// LL
	if (ObjectHit_button(mouseX , mouseY , button_up2) == false){
		dbDeleteSprite(button_down2);
		dbSprite(button_up2 , 353 , 333 , button_up2);}
	if (ObjectHit_button(mouseX , mouseY , button_up2) == true){
		dbDeleteSprite(button_up2);
		dbSprite(button_down2 , 353 , 333 , button_down2);
		sncCenterText(492 , 265 , "Deals 1/4 enemies health");
		if (dbMouseClick()){
			clicked = true;
			attacked = true;
			playerDamage = (monData.health)/2;
			if (changeDamage == true){
				playerDamage += powerThirst.changeDamage;
				changeDamage = false;}
			movesLeft -= 2;}
	}
	// LR
	if (ObjectHit_button(mouseX , mouseY , button_up3) == false){
		dbDeleteSprite(button_down3);
		dbSprite(button_up3 , 495 , 333 , button_up3);
	}
	if (ObjectHit_button(mouseX , mouseY , button_up3) == true && changeDamage == true)
		sncCenterText(320 , 240 , "Only 1 per turn");
	if (ObjectHit_button(mouseX , mouseY , button_up3) == true && changeDamage == false){
		dbDeleteSprite(button_up3);
		dbSprite(button_down3 , 495 , 333 , button_down3);
		sncCenterText(492 , 265 , "+15 to next attack");
		if (dbMouseClick()){
			clicked = true;
			playerDamage = 0;
			changeDamage = true;
			movesLeft -= 1;
			attacked = true;}
	}

	sncCenterText(435 , 280 , "Damage: ");
	sncCenterText(470 , 280 , fakePlayerDamage);
	sncCenterText(543 , 280 , "Moves Left: ");
	sncCenterText(592 , 280 , movesLeft);

	//// Next 5 lines assign priorities to needed sprites
	//int i;
	//const int max = 11;
	//int tempArray[max] = {test , test2};
	//for (i=0 ; i<max ; i++)
	//	dbSetSpritePriority(tempArray[i], i+1);

	return;
}

// battle health bar for player and monster
void healthBar()
{
	float temp1 = 175 / monsterHealth;
	float temp2 = 175 / playerHealth;

	dbBox(122 , 67 , 122 + (monData.health * temp1) , 83);
	dbInk(BLACK, 0);
	sncCenterText(205 , 68 , monData.health);

	dbInk(WHITE, 0);
	dbBox(519 - (player.health * temp2), 430 , 519 , 446);
	dbInk(BLACK, 0);
	sncCenterText(427 , 431 , player.health);

	return;
}