# Implement the Tetris with C++

## 1. Introduction

In this lab, we will analyze the train of thoughts before we design the Tetris, and we will introduce the way to use the ncurses library. Then we'll implement the design of the key functions in Tetris, and finish its basic functions. And we can run it.

#### Learning Objective

+ Basics of C++
+ Ncurses library
+ Logistic design of the Tetris
+ Draw the windows
+ Design the class cube
+ Rotation algorithm
+ Move and remove function

#### Environment Requirement 

+ xfce terminal
+ g++ compiler
+ ncurses library

#### Target users

This course is of medium difficulty, so it suits for the peopole who has basic C++ programing skills and the interest in both game design and logistic analyzing.

#### Source Code

```c++
git clone https://github.com/Gamerchen/game_zero.git
```

## 2.Build and Run

#### Install Ncurses Library

Open `Xfce Terminal`, run the commands:

```c++
$ sudo apt-get update
$ sudo apt-get install libncurses5-dev
```

#### Compile

Please add option "-l" into ncurses library while compiling your commands.

```c++
$ g++ main.cpp -l ncurses -o game
```

#### Run the Program

```c++
$ ./game
```

#### Results

![Results](https://doc.shiyanlou.com/document-uid18510labid4159timestamp1512720488161.png/wm)


## 3. Theory

Before working on the program, we should figure out what functions to implement, and what modules to divide. Next, about the Tetris, the first thing coming into our mind should be displaying the cubes, and the followings are how the cubes fall, move horizontally and rotate. And at last, the layer full of cubes should be removed. Additionally, a basic Tetris should have a prompt function to show the shape of the next cube.

So the problems to be solved while programming are:

+ *Display the cubes*
+ *Implement the movement of the cubes*
+ *Rotate the cube*
+ *Remove the layer full of cubes*
+ *Prompt the shape of the next cube*

#### 3.1 Basic Images

Every cube is made of 4 boxes, and it falls from the center of the window. It can rotate while not touching the border and other cubes.

![image](https://doc.shiyanlou.com/document-uid577835labid4158timestamp1512719069577.png/wm)

#### 3.2 How to Use NCURSES Library

Basicly, NCURSES is a clone of CURSES in System V Release 4.0 (SVr4). It's a library that can be configured freely, and it's compeletly compatible with the old version of CURSES. Besides, you can use applications to directly control the terminal window display in this library. NCURSES packages the terminal function of the infrastructure, including some functions that create windows. It is the extention of Menu, Panel, Form in the CURSES basic library. We can build an application that contains multiple windows, menus, panels and forms in the same time. Windows can be managed independently, like making it scroll or hide. Menus can let users build command options to make it easier to execute the command. And forms allow users to build some simple windows to input data and display it. Panels are the extention of the windows management function in NCURSES, you can use Panels to cover or accmulate the windows.

#### 3.3 Begin with "Hello World" 

If you invoke functions in NCURSES library, you must load the ncurses.h file in the code. (stdio.h is already contained in ncurses.h)

Example:

```c++
#include <ncurses.h>

int main()
{
	initscr();   //Initialize, enter NCURSES mode
	printw("Hello World!"); //Print Hello World in the virtual screen
	refresh(); //Write the content of the virtual screen on the display and refresh
	getch();  //Wait for the user to input
	endwin(); //Exit NCURSES mode
	return 0;
}
```

In the example above, we have introduced the way to use the most basic functions in NCURSES library. What the FUNCTIONS can do is already written in the comment, so we won't repeat.

#### 3.4 The Mechanism of Window

As NCURSES initializes, it will create a window named stdscr by default. The size of the window is usually 80 rows and 25 columns(the size defers from the  monitor or video card). Besides, you can also create your own window by using the functions in the window system.

For example, if you invoke the following function:

```c++
printw("Hi!");
refresh();
```

It would input "Hi" in the current cursor position on stdscr, invoke the **`refresh()`** function and only update the buffer area on stdscr.

If you have already built a window named win and want to output content in the `win` window, you can add M before the normal functions while the parameter is also changing.

```c++
printw(string); //Print string in the current cursor position on stdscr
                 
mvprintw(y,x,string);  //Print string in the coordinate (y,x)
wprintw(win,string); //Print string in the current cursor position in win window
mvwprintw(win,y,x,string);  //Move the cursor to the coordinate (y,x) in win window and print string
```

**I believe that after reading the example above, you can already tell the functional differences in the functions by the way we name the functions.**


#### 3.5 Newwin and the Box Function

Building a window begins with **`newin()`** function .The function will return a pointer to the the structure of the window. this pointer can be transferred to functions that need window parameter like **`wprintw()`**
However, we've built a window that can't be seen, we need to use **`box()`** function to draw a border in the identified windows.

For example

```c++
WINDOW *create_newwin(int height, int width, int starty, int startx)
{
	WINDOW *local_win;
	local_win = newin(height, width, starty, startx);
	box(local_win, 0, 0);
	wrefresh(local_win);
	return local_win;
}
```

We've finished the introduction to the basic ways to use NCURSES library. You still need to refer to the related files while running into problems in your work.

## 4. Implementation

First, contain the headfile and define a function of change and a function of random number. We'll use them later. (The former is used for the rotation of the cube and the latter is used for setting the shape of the cube)

```c++
#include <iostream>
#include <sys/time.h>
#include <sys/types.h>
#include <stdlib.h>
#include <ncurses.h>
#include <unistd.h>

/* Swap a and b */

void swap(int &a, int &b){
	int t=a;
	a = b;
	b = t;
}

/* Get a random integer in interval (min,max) */

int getrand(int min, int max)
{
	return(min+rand()%(max-min+1));
}
```

#### 4.1 Define the Class

As the program is relatively easy, we only define `class Piece` here.

```c++
class Piece
	{
	public:
		int score; 	// Score
		int shape; 	//It represents the shape of the current cube
		int next_shape; 	//It represents the shape of the next cube
		int head_x;		//The position of the first box of the current cube, and mark the position
		int head_y;
	
		int size_h;		//The size of the current cube
		int size_w;
	
		int next_size_h;	 //The size of the next cube
		int next_size_w;
	
		int box_shape[4][4];	//The shape array of the current cube 4*4
		int next_box_shape[4][4]; 	//The shape array of the next cube 4*4
		int box_map[30][45];	 //Mark every box inside the game border
		bool game_over; 	//The siginal that shows the game is over
	public:
		void initial(); 	//Initialize the function
		void set_shape(int &cshape, int box_shape[][4],int &size_w, int & size_h); 	//Set the shape of the cube
	
		void score_next(); 	//Display the shape and score of the next cube
		void judge(); 	//Determine whether the layer is full
		void move();	//Move the function, use ← → ↓ to control 
		void rotate();	//Rotate function
		bool isaggin();	//Determine whether the next move would cross the border or coincide with it
		bool exsqr(int row); //Determine if the current row is empty
		
	};
```

#### 4.2 Set the Shape of the Cube

We define the shape of 7 cubes by using `case` statement here. Every time before the next cube falls, we all should invoke this statement to set the shape and initial position of it.

```c++
void Piece::set_shape(int &cshape, int shape[][4],int &size_w,int &size_h)
{
	/*First, initialize the 4*4 array to 0*/
	
	int i,j;
	for(i=0;i<4;i++)
		for(j=0;j<4;j++)
			shape[i][j]=0;

	/*Set 7 initial shapes and set their sizes*/
	
	switch(cshape)
	{
		case 0:	
			size_h=1;
			size_w=4;	
			shape[0][0]=1;
			shape[0][1]=1;
			shape[0][2]=1;
			shape[0][3]=1;
			break;
		case 1:
			size_h=2;
			size_w=3;
			shape[0][0]=1;
			shape[1][0]=1;
			shape[1][1]=1;
			shape[1][2]=1;
			break;
		case 2:
			size_h=2;
			size_w=3;	
			shape[0][2]=1;
			shape[1][0]=1;
			shape[1][1]=1;
			shape[1][2]=1;
			break;
		case 3:
			size_h=2;
			size_w=3;
			shape[0][1]=1;
			shape[0][2]=1;
			shape[1][0]=1;
			shape[1][1]=1;
			break;

		case 4:
			size_h=2;
			size_w=3;
			shape[0][0]=1;
			shape[0][1]=1;
			shape[1][1]=1;
			shape[1][2]=1;
			break;

		case 5:	
			size_h=2;
			size_w=2;
			shape[0][0]=1;
			shape[0][1]=1;
			shape[1][0]=1;
			shape[1][1]=1;
			break;

		case 6:	
			size_h=2;
			size_w=3;
			shape[0][1]=1;
			shape[1][0]=1;
			shape[1][1]=1;
			shape[1][2]=1;
			break;
	}

	//After setting the shape, initialize the beginning position of the cube

	head_x=game_win_width/2;
	head_y=1;

	//If it coincides with the border right after the initializing, then the game is over.
	if(isaggin())    /* GAME OVER ! */
		game_over=true;

}
```

#### 4.3 `Rotate()`

We use a relatively simple algorithm to rotate the cube here, quite like the rotation of matrix. First we make the shape array digonally symmetrical, and next make it bilaterally symmetrical, then we have finished the rotation. Pay attention here, we need to determine whether the cube is out of the border or coincides with the border after the rotation, if so, cancel the rotation.

```c++
void Piece::rotate()
	{
		int temp[4][4]={0};  //Temporary variable
		int temp_piece[4][4]={0};  //Backup array
		int i,j,tmp_size_h,tmp_size_w;
	
		tmp_size_w=size_w;
		tmp_size_h=size_h;
	
		for(int i=0; i<4;i++)
			for(int j=0;j<4;j++)
				temp_piece[i][j]=box_shape[i][j];	
   /*Back up the current cube. If the rotation fails, return to the current shape.*/
	
		for(i=0;i<4;i++)
			for(j=0;j<4;j++)
				temp[j][i]=box_shape[i][j];	//Digonal symmetry
		i=size_h;
		size_h=size_w;
		size_w=i;
		for(i=0;i<size_h;i++)
			for(j=0;j<size_w;j++)
				box_shape[i][size_w-1-j]=temp[i][j];	//Bilateral symmetry
	
		/* it coincides with the border, return to the shape of the backup array */
		
		if(isaggin()){
			for(int i=0; i<4;i++)
				for(int j=0;j<4;j++)
					box_shape[i][j]=temp_piece[i][j];
			size_w=tmp_size_w;	//Remember, the size should return to the formal size
			size_h=tmp_size_h;
		}

		/* If the rotation succeeds, display it on the screen */
		
		else{
			for(int i=0; i<4;i++)
				for(int j=0;j<4;j++){
					if(temp_piece[i][j]==1){
						mvwaddch(game_win,head_y+i,head_x+j,' ');	
        /* Move to some coordinate in the game_win window, and print the character */
						wrefresh(game_win);
					}
				}
			for(int i=0; i<size_h;i++)
				for(int j=0;j<size_w;j++){
					if(this->box_shape[i][j]==1){
						mvwaddch(game_win,head_y+i,head_x+j,'#');
						wrefresh(game_win);
					}
			}
	
		}
}
```

#### 4.4 `Remove()`

If the user didn't press any key, the cube should fall slowly, so we cannot stuck at **`getch()`** for waiting for the input. **`Select()`** is used here to avoid the stuck.

```c++
/* We just put part of the program here, please refer to the source code to implement. */

struct timeval timeout;
 timeout.tv_sec = 0;
 timeout.tv_usec= 500000;
      
if (select(1, &set, NULL, NULL, &timeout) == 0)
```

timeout is the longest time we would wait for the user to press the key. We set 500000us here, and won't wait for the **`getch()`** to input after waiting this long. We will move on to the next step directly.

If we detect the input within the timeout time, then the following if statement is true, and we'll get the value of the key that has been just inputed. We can make the cube go left, right, down and rotate by determining the different value of the key.

```c++
if (FD_ISSET(0, &set)) 
	while ((key = getch()) == -1) ;
```

The way to process the functions of moving left, right and down are generally the same, we only talk about the moving down function here.

```c++
/* Refer to the source code for the full program */


struct timeval timeout;
/* If te key you press is ↓ */

if(key==KEY_DOWN){
		head_y++;	// Plus 1 on the y cordinate of the cube
		if(isaggin()){	
        /* If it goes out of the border or coincides with it, then cancel this movement. */
			head_y--;

			/*Since it's been stopped, set the corresponding box on the map occpuied. 1 means occpuied, and 0 means not.*/
			for(int i=0;i<size_h;i++)
				for(int j=0;j<size_w;j++)
					if(box_shape[i][j]==1)
						box_map[head_y+i][head_x+j]=1;

			score_next(); //Show the score and prompt the next cube
			
		}

		/*If it can move down, then cancel the display of the current cube, move it to the next row and display. Pay attention here, the for loop has to go up from the bottom.*/
		else{
			for(int i=size_h-1; i>=0;i--)
				for(int j=0;j<size_w;j++){
					if(this->box_shape[i][j]==1){
						mvwaddch(game_win,head_y-1+i,head_x+j,' ');
						mvwaddch(game_win,head_y+i,head_x+j,'#');

					}
				}
			wrefresh(game_win);
}
```

#### 4.5 `Repeat()`

This function has to determine after every movement and rotation. If it returns true, then it cannot move, else it can go on the next step.

```c++
bool Piece::isaggin(){
	for(int i=0;i<size_h;i++)
		for(int j=0;j<size_w;j++){
			if(box_shape[i][j]==1){
				if(head_y+i > game_win_height-2)	//The bottom is out of the border
					return true;
				if(head_x+j > game_win_width-2 || head_x+i-1<0)	
				//It's out of the horizontal border 
					return true;
				if(box_map[head_y+i][head_x+j]==1)	
				//It coincides with the occpuied box
				return true ;
			}
		}
	return false;
}
```

#### 4.6  `Judge()`

The last function is very important, it removes the row full of cubes. When every cube stops moving down, it has to determine.

```c++
void Piece::judge(){
	int i,j;
	int line=0;	//To mark the row of the full layer
	bool full;
	for(i=1;i<game_win_height-1;i++){ // Remove the boder
		full=true;
		for(j=1;j<game_win_width-1;j++){
			if(box_map[i][j]==0) //An unoccupied box exists
				full=false; //It means this layer is not full
		}
		if(full){ //If this layer is full
			line++; //Plus 1 on this layer
			score+=50; //Add the score~
			for(j=1;j<game_win_width-1;j++)
				box_map[i][j]=0; //Remove this layer(mark it as unoccupied)
		}
	}

	/*	After determining the these, check the value of line, if it's not 0, then there's a full layer to be removed.*/

	if(line!=0){
	for(i=game_win_height-2;i>=2;i--){
		int s=i;
		if(exsqr(i)==0){
			while(s>1 && exsqr(--s)==0);	//Search the row of the exsiting cube and move it down
			for(j=1;j<game_win_width-1;j++){
				box_map[i][j]=box_map[s][j]; // Move the up layer down
				box_map[s][j]=0;	//Remove the up layer
			}
		}
	}

	/*The screen will be refreshed after removing and moving the marks. Print game_win again.*/
	for(int i=1;i<game_win_height-1;i++)
			for(int j=1;j<game_win_width-1;j++){
				if(box_map[i][j]==1){
					mvwaddch(game_win,i,j,'#');
					wrefresh(game_win);
				}
				else{
					mvwaddch(game_win,i,j,' ');
					wrefresh(game_win);
				}
			}
	}
}		
```

## 5. Summray

Now we have finished introducing the key functions. You definitely can operate the program as long as you figure out the major functions, implement them and refer to the source code to complement other functions. What's more, there are many other ways of implementing Tetris which may vary from person to person. It is possible for you to compose a more fluent and concise program. Have some fun!
