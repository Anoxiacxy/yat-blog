---
layout: post
title: CLI Greedy Snake
subtitle: ''
categories: game
tags: [c++]
---

## 总览

![](/assets/images/20171210-GreedySnake.png)

以上已经给出了有关此程序的效果图和流程图，虽然与实际程序稍有差异，但也八九不离十了。
关于此次贪吃蛇的制作，主要还是为了锻炼一下代码能力，也就相当于一个超级大模拟吧。
具体相当于要实现以下功能

### 游戏实现
游戏的主体部分，即有关地图的加载，蛇身的移动，游戏结束的判定等等，地图由文件形式存储，可以自己DIY哈
### 菜单界面
其实，这个玩意才最难搞，我写了个 Button 类，专门来处理有关这些菜单按钮的界面，然后疯狂调参，把按钮们放在一个合适的位置
### 分数统计
为永久保存历史记录与排行榜，可以开一个文件，把战绩都保存在里面，然后每次更新文件里的内容即可
### 操作设置
有的人习惯按转向来操作，即控制蛇的左拐和右拐，有的人习惯直接用上下左右控制蛇的方向，考虑到这一点，我添加了这个设置，并且为了永久保存，依旧将设置的结果保存在文件里面

## 介绍
首先进入的便是主菜单，截图如下



这个时候可以上下键选择，空格或者回车键确认，即可进入相应的选项，先直接开始游戏



选择一个模式，选择的方法同上



注意，游戏时请将输入法切换成英文，同时尽可能使用 WASD 而不是键盘右侧的上下左右，减少一些不明原因带来的延迟现象

同时，空格键还有暂停的功能，再次点击即可继续游戏（本来打算再写个暂停的界面的，但因问没 (bo) 时 (zhu) 间 (lan) ，就放弃了）



然后是游戏结束时的界面，依旧上下键选择，空格回车键确认，此界面与分数统计一起食用效果更佳




历史最佳，即分数统计系统，可用上下键翻页，空格回车键返回上一级




以上就是设置里面两种操作模式的选择，可以根据个人的喜好自行设置，关于那个动画的实现，我比较蠢。。。就直接一个一个字慢慢输出的，中文汉字还真不知道有没有什么好的输出办法。。。



最后打个自己博客的广告，溜咯

## 高级
### 自定义地图
在 data/mode/ 文件夹下有 4 个地图包，点开之后里面大概长这个样子，然后我们就可以愉快的修改地图啦，注意不要让蛇一开始就撞到墙上，地图的大小也必须是 15×20 的，虽然你也可以直接在源代码里面强行修改地图的大小，但那样的话可能会造成文字显示的位置不对等问题


可以看到，你还可以在里面修改颜色，以及有关蛇的参数，注意蛇身的坐标必须按顺序给出，不然后果会很严重
另外，因为文件读入的识别功能不那么智能，所以请不要随意调整参数的位置与空格的格式。。。

## 游戏下载
注意，此下载链接仅供娱乐，内含数据包及可执行程序，解压后无需安装即可开始游戏，但不包含源代码

## 源代码
没写什么注释，只把关于 windows API 里面自己不熟悉的函数写了下注释，大家将就看下

```c++
//Anoxiacxy
#include <windows.h> 
#include <iostream>
#include <fstream>
#include <cstring>
#include <conio.h>
#include <vector> 
#include <cstdio>
#include <algorithm>
#include <ctime>
#define W 20 
#define H 15 
using namespace std;
//-----------------------------隐藏光标
void HideCursor(){
	HANDLE handle = GetStdHandle(STD_OUTPUT_HANDLE);
	CONSOLE_CURSOR_INFO CursorInfo;
	GetConsoleCursorInfo(handle,&CursorInfo);//获取控制台光标信息
	CursorInfo.bVisible=false;//隐藏控制台光标
	SetConsoleCursorInfo(handle,&CursorInfo);//设置控制台光标状态
}
void SetWindowSize(int cols, int lines);
void Goto(int x, int y);
void SetColor(int colorID);
void SetWindowSize(int cols, int lines){//设置窗口大小
    system("title GreedySnake");//设置窗口标题
    char cmd[30];
    sprintf(cmd, "mode con cols=%d lines=%d", cols * 2, lines);//一个图形■占两个字符，故宽度乘以2
    system(cmd);//设置窗口宽度和高度
}
void Goto(int x,int y){//设置光标位置
    COORD position;
    position.X = x * 2;
    position.Y = y;
    SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), position);
}
void SetColor(int colorID){//设置文本颜色
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), colorID);
}
//---------------------------------------------//
const string CLEAR="  ";
class Point{ // Point类 表示控制台里面的一个坐标 
private:
	int x,y; 
public:
	Point(int x = 0,int y = 0):x(x), y(y){}
	void Set(int x,int y); //将坐标设为-- 
	void Print(string Object); //在该坐标输出一个字符 
	void Print(string Object, int color); //在该坐标输出一个带颜色的字符 
	void Clear(); //清空当前坐标的字符 
	int Getx(){return this->x;}
	int Gety(){return this->y;}
	bool operator == (const Point& point)
		{return point.x == this->x && point.y == this->y;}
};
void Point::Print(string Object, int color){
	SetColor(color);
	Goto(x, y);
	cout << Object;
} 
void Point::Print(string Object){
	Goto(x, y);
	cout << Object;
} 
void Point::Clear(){
	Point::Print(CLEAR);
}
void Point::Set(int x, int y){
	this->x = x;
	this->y = y;
}
//---------------------------------------------//
#define STARTGAME 0
#define RANKING 1
#define SETTING 2
#define ABOUT 3
#define EXIT 4
#define MAINMENU_TOT 5

#define EASY 0
#define NORMAL 1
#define HARD 2
#define UNBELIEVABLE 3
#define MODEMENU_TOT 4

#define PLAY_AGAIN 0
#define SHOW_RANK 1
#define BACK_TO_MODEMENU 2
#define BACK_TO_MAINMENU 3
#define GAMEOVERMENU_TOT 4
class Button{
public:
	Button(){}
	void Set(string s, Point p, int c1, int c2);	//设置当前按钮显示字符，位置，未选中时颜色，选中时颜色 
	void Print(); 	//输出 
	bool slect; 	//是否选中 
	Point pos;		 
private:
	string discribe;
	int color[2];
	//0---unslected 1---slected 
};
void Button::Set(string s, Point p, int c1, int c2){
	discribe = s;
	pos = p;
	color[0] = c1;
	color[1] = c2; 
	slect = 0;
}
void Button::Print(){
	SetColor(color[slect]);
	Goto(pos.Getx(), pos.Gety());
	cout << discribe;
}

//---------------------------------------------//

#define EMPTY 0
#define WALL 1
#define SNAKE 2
#define FOOD 3
#define ELEMENT_TOT 4
#define ABOUT_TOT 4
void Initializatoin();//初始化，读入数据 

string element[ELEMENT_TOT];//元素符号 
Button about_button[ABOUT_TOT];

int MainMenu();	//主菜单--返回选项 
int ModeMenu();	//模式选择--返回选项 
int GameOverMenu(int); //游戏结束--传入得分--返回选项 
int SetMenu(); 	//操作设置--返回选项 
int mp[W][H]; 	//地图 
int empty_num;	//地图中空位的个数 
void NewFood(int color){ //在地图的剩余空位中随机生成一个食物 
	srand(time(NULL));
	int pos = rand() % empty_num + 1;
	for(int j = 0; j < H; j++)
		for(int i = 0; i < W; i++){
			if(mp[i][j] == EMPTY) pos--;
			if(pos == 0 && mp[i][j] == EMPTY){
				mp[i][j] = FOOD;
				Point(i, j).Print(element[FOOD], color);
				empty_num--;
				return;
			}
		}	
}
struct Snake{	 
	Point body[W * H]; //身体--由一个个点组成 
	int head, tail;	//头-尾 
	int speed;		//速度 
	int dicx, dicy; //方向 
	int color;
	Snake(){}
	void Set(Point snk[], int len, int _color, int _speed, int x, int y) {	//一个初始化函数 
		head = len; tail = 1;
		color = _color;
		speed = _speed;
		dicx = x; dicy = y;
		for(int i = 1; i <= len; i++){
			body[i] = snk[i];
			body[i].Print(element[SNAKE], color);
			mp[body[i].Getx()][body[i].Gety()] = SNAKE;	
		}
	}
	int move() { //控制蛇向当前方向移动一格--返回蛇头遇到的物品 
		int hx = body[head].Getx() + dicx;
		int hy = body[head].Gety() + dicy;
		int tmp = mp[hx][hy];
		if(mp[hx][hy] == WALL || mp[hx][hy] == SNAKE) return tmp;
		if(mp[hx][hy] != FOOD){
			int tx = body[tail].Getx();
			int ty = body[tail].Gety();
			body[tail].Clear();
			mp[tx][ty] = EMPTY;
			tail = (tail + 1) % (W * H);
		}
		mp[hx][hy] = SNAKE;
		head = (head + 1) % (W * H);
		body[head].Set(hx, hy);
		body[head].Print(element[SNAKE], color);
		return tmp;
	}
	void turn_left() {	//左转 
		dicx *= -1; 
		swap(dicx, dicy);
	} 
	void turn_right() {	//右转 
		dicy *= -1;
		swap(dicx, dicy);
	}
};
#define OPT_L_R 0 	//ad左右拐弯
#define OPT_U_D_L_R 1//直接转向xx 
#define OPT_TOT 2
#define DISCRIBE_LEN 100 //选项描述的最大长度（其实用不着这么多） 

int set_opt=1;

void DealOpt(char press, Snake& snake){	//处理游戏时的按键操作，强烈建议仅使用 WASD 
	if(press == ' ')while(getch() != ' ');
	else if(set_opt == OPT_L_R)
		switch(press){
			case 75://left
			case 'A':
			case 'a':
				snake.turn_left(); break;
			case 77://right
			case 'D':
			case 'd':
				snake.turn_right(); break;
			default:break;
		}
	else if(set_opt == OPT_U_D_L_R)
		switch(press){
			case 'A':
			case 'a':
			case 75 :
				if(snake.dicx == 0){
					if(snake.dicy == 1) snake.turn_right();
					else snake.turn_left();
				}break;
			case 'D':
			case 'd':
			case 77 :
				if(snake.dicx == 0){
					if(snake.dicy == -1) snake.turn_right();
					else snake.turn_left();
				}break;
			case 'W':
			case 'w':
			case 72 :
				if(snake.dicy == 0){
					if(snake.dicx == -1) snake.turn_right();
					else snake.turn_left();
				}break;
			case 'S':
			case 's':
			case 80 :
				if(snake.dicy == 0){
					if(snake.dicx == 1) snake.turn_right();
					else snake.turn_left();
				}break;
		}
}
int Play(int Mode){	//游戏的过程 
	ifstream fin; string tmmp;
	switch(Mode){			//打开选择的模式地图 
		case EASY: 			fin.open("data//mode//easy.mode");			break;
		case NORMAL: 		fin.open("data//mode//normal.mode");		break;
		case HARD:			fin.open("data//mode//hard.mode");			break;
		case UNBELIEVABLE:	fin.open("data//mode//unbelievable.mode");	break;
		default:return 0;
	}
	if(!fin.is_open()){
		system("cls");
		cout << "数据丢失，请检查数据文件后重新启动游戏";
		exit(0);
	}
	empty_num=0;	//载入地图 
	int wall_color, snake_color, food_color;
	fin >> tmmp >> wall_color >> tmmp >> snake_color >> tmmp >> food_color >> tmmp;
	for(int j = 0; j < H; j++)
		for(int i = 0; i < W; i++) {
			fin >> mp[i][j];
			if(mp[i][j] == EMPTY) empty_num++;
			Point(i, j).Print(element[mp[i][j]], wall_color);
		}
	int len;
	Point tmp[W*H];
	fin >> tmmp >> len >> tmmp;
	for(int i = 1; i <= len; i++){
		empty_num--;
		int x,y;
		fin >> tmmp >> x >> tmmp >> y >> tmmp;
		tmp[i].Set(x, y);
	}
	int dicx, dicy, speed;
	fin >> tmmp >> dicx >> dicy;
	fin >> tmmp >> speed;
	fin.close();
	Snake snake;
	snake.Set(tmp, len, snake_color, speed, dicx, dicy);
	NewFood(food_color);
	int score = 0;
	bool live = true;
	while(live) {	//游戏循环--当蛇活着时 
		if(kbhit()) {
			char press=getch();
			DealOpt(press, snake);
		}
		switch(snake.move()){
			case SNAKE:
			case WALL:  live=false; break;
			case FOOD:  NewFood(food_color);
						score++;
						break;
			default:break;
		}
		Sleep(1000 / snake.speed);
	}
	return score;
}
#define SHOW_NUM 5
#define SPEED 30 
int best_score[10086];
void UpdataScore(int score){	//游戏结束后更新分数，并及时记录在文件中 
	best_score[++best_score[0]] = score;
	for(int k = best_score[0]; best_score[k] > best_score[k-1] && k > 1; k--)
		swap(best_score[k], best_score[k-1]);
	while(best_score[0] > 10000) best_score[0]--;
	ofstream fout;
	fout.open("data//best.score");
	for(int i = 0; i <= best_score[0]; i++)
		fout << best_score[i] << endl;
	fout.close();
}
void ShowScore(int now){	//打印当前页的分数 
	system("cls");
	for(int i = 1; i <= SHOW_NUM && (now - 1) * SHOW_NUM + i <= best_score[0]; i++){
		Goto(8, i << 1 | 1); Sleep(SPEED);
		printf("%d",(now - 1) * SHOW_NUM + i);
		Goto(10,i << 1 | 1);
		printf("%4d", best_score[(now - 1) * SHOW_NUM + i]);
	}
}
void Setting() {	//就是操作的设置啦--同样及时记录 
	set_opt = SetMenu();
	ofstream fout;
	fout.open("data//others//setting.txt");
	fout << set_opt << endl;
	fout.close();
}
void Ranking(){	//此函数可以实现分数的翻页功能
	 int cnt = (best_score[0] + SHOW_NUM - 1) / SHOW_NUM;
	 int now = 1;
	 ShowScore(now);
	 while(true){
	 	bool turn_out = false;
	 	switch(getch()){
			case 'w':
			case 72 ://up
				if(now != 1){
					now--;
					ShowScore(now);
				}
				break;
			case 's':
			case 80 ://down
				if(now < cnt){
					now++;
					ShowScore(now);
				}
				break;
			case ' ':
			case 13 ://enter
				turn_out = true;
			default:break;
		}
		if(turn_out)break;
	}
}
void StartGame(){	//开始游戏的选项 
	while(true){
		system("cls");
		int mode = ModeMenu();  
		bool changemode = false;
		while(true){
			int score = Play(mode);
			UpdataScore(score);
			while(true){
				bool turn_out = false;
				switch(GameOverMenu(score)){
					case PLAY_AGAIN:		turn_out = true; break;
					case SHOW_RANK:			Ranking(); break;
					case BACK_TO_MAINMENU:	return;
					case BACK_TO_MODEMENU:	turn_out=true; 
											changemode=true; break;		
				}
				if(turn_out)break;
			}
			if(changemode == true)break;
		}
	}
}
void About() {	//关于页面--只能在源代码里面修改 不属于外面的data文件 
	system("cls");
	for (int slect = 0; slect < ABOUT_TOT; slect++) 
		about_button[slect].Print();
	while(true){
		switch(getch()){
			case ' ':
			case 13 ://enter
				return;
			default :break;
		}
	}
} 
int main(){
	SetWindowSize(W, H);	//设置窗口大小 
	HideCursor();			//隐藏光标 
	//StartAnimation();		//开始动画--这个也懒得写了 
	Initializatoin();		//初始化 
	while(true){
		switch(MainMenu()){
			case STARTGAME:	StartGame();break;
			case RANKING:	Ranking();break;
			case SETTING:	Setting();break;
			case ABOUT:		About();break;
			case EXIT:		exit(0);
			default: 		break;
		}
	}
	return 0;
}
void Initializatoin(){	//初始化--读入一大堆文件之类的 
	ifstream fin; string tmmp;
	fin.open("data//others//element.txt");//读入元素符号
	if(!fin.is_open()){
		system("cls");
		cout << "数据丢失，请检查数据文件后重新启动游戏";
		exit(0);
	}
	element[0]="  ";  
	for(int i = WALL; i < ELEMENT_TOT; i++) 
		fin >> tmmp >> element[i];
	fin.close();
	fin.open("data//best.score");	//读入历史分数 
	
	if(fin.is_open()){
		fin >> best_score[0];
		for(int i = 1; i <= best_score[0]; i++)
			fin >> best_score[i];
	}
	fin.close();
	fin.open("data//others//setting.txt");
	if(!fin.is_open()){
		system("cls");
		cout << "数据丢失，请检查数据文件后重新启动游戏";
		exit(0);
	}
	fin >> set_opt;
	fin.close();
	
	string s; int c1, c2, xpos, ypos;	//防伪--广告 
	s = "code by"; c1 = c2 = 8; xpos = 5, ypos = 6;
	about_button[0].Set(s, Point(xpos,ypos), c1, c2);
	s = "anoxiacxy"; c1 = c2 = 15; xpos = 10, ypos = 6;
	about_button[1].Set(s, Point(xpos,ypos), c1, c2);
	s = "anoxiacxy.github.io"; c1 = c2 = 8; xpos = 5, ypos = 7;
	about_button[2].Set(s, Point(xpos,ypos), c1, c2);
}
int MainMenu(){
	Button main_button[MAINMENU_TOT];//主菜单按钮 
	ifstream fin; string tmmp;
	fin.open("data//button//mainmenu.button");//载入 
	if(!fin.is_open()){
		system("cls");
		cout << "数据丢失，请检查数据文件后重新启动游戏";
		exit(0);
	}
	for(int slect = STARTGAME; slect < MAINMENU_TOT; slect++){
		string s; int c1, c2, xpos, ypos;
		fin >> tmmp >> s >> tmmp;
		fin >> tmmp >> xpos >> tmmp >> ypos >> tmmp;
		fin >> tmmp >> c1 >> c2;
		main_button[slect].Set(s, Point(xpos,ypos), c1, c2);
	}
	fin.close();
	system("cls");
	for(int slect = STARTGAME; slect < MAINMENU_TOT; slect++){
		main_button[slect].slect = false;
		main_button[slect].Print();
	} 
	int slect = STARTGAME;
	main_button[slect].slect = true;
	main_button[slect].Print();
	while(true){
		switch(getch()){
			case 'W':
			case 'w':
			case 72://up
				main_button[slect].slect = false;
				main_button[slect].Print();
				slect = (slect - 1 + MAINMENU_TOT) % MAINMENU_TOT;
				main_button[slect].slect = true;
				main_button[slect].Print();
				break;
			case 'S':
			case 's':
			case 80://down
				main_button[slect].slect = false;
				main_button[slect].Print();
				slect=(slect+1)%MAINMENU_TOT;
				main_button[slect].slect = true;
				main_button[slect].Print();
				break;
			case ' ':
			case 13://enter
				return slect;
			default:break;
		}
	}
}
int ModeMenu(){
	Button mode_button[MODEMENU_TOT];//模式选择按钮 
	ifstream fin; string tmmp;
	fin.open("data//button//modemenu.button");//载入  
	if(!fin.is_open()){
		system("cls");
		cout<<"数据丢失，请检查数据文件后重新启动游戏";
		exit(0);
	}
	for(int slect=EASY;slect<MODEMENU_TOT;slect++){
		string s;int c1,c2,xpos,ypos;
		fin >> tmmp >> s >> tmmp;
		fin >> tmmp >> xpos >> tmmp >> ypos >> tmmp;
		fin >> tmmp >> c1 >> c2;
		mode_button[slect].Set(s, Point(xpos,ypos), c1, c2);
	}
	fin.close();
	system("cls");
	for(int slect = EASY; slect < MODEMENU_TOT; slect++){
		mode_button[slect].slect = false;
		mode_button[slect].Print();
	}
	int slect = EASY;
	mode_button[slect].slect = true;
	mode_button[slect].Print();
	while(true){
		switch(getch()){
			case 'W':
			case 'w':
			case 72://up
				mode_button[slect].slect = false;
				mode_button[slect].Print();
				slect=(slect-1+MODEMENU_TOT)%MODEMENU_TOT;
				mode_button[slect].slect = true;
				mode_button[slect].Print();
				break;
			case 'S':
			case 's':
			case 80://down
				mode_button[slect].slect = false;
				mode_button[slect].Print();
				slect=(slect+1)%MODEMENU_TOT;
				mode_button[slect].slect = true;
				mode_button[slect].Print();
				break;
			case ' ':
			case 13://enter
				return slect;
			default:break;
		}
	}	
}
int GameOverMenu(int score){
	Button gameover_button[GAMEOVERMENU_TOT];	//游戏结束的按钮 
	ifstream fin; string tmmp;
	fin.open("data//button//gameovermenu.button");//载入 
	if(!fin.is_open()){
		system("cls");
		cout<<"数据丢失，请检查数据文件后重新启动游戏";
		exit(0);
	}
	for(int slect = PLAY_AGAIN; slect < GAMEOVERMENU_TOT; slect++){
		string s;int c1, c2, xpos, ypos;
		fin >> tmmp >> s >> tmmp;
		fin >> tmmp >> xpos >> tmmp >> ypos >> tmmp;
		fin >> tmmp >> c1 >> c2;
		gameover_button[slect].Set(s, Point(xpos,ypos), c1, c2);
	}
	fin.close();	
	system("cls");
	SetColor(15);
	Goto(8, 3); printf("分数%4d", score);
	for(int slect = PLAY_AGAIN; slect < GAMEOVERMENU_TOT; slect++){
		gameover_button[slect].slect = false;
		gameover_button[slect].Print();
	}
	int slect = PLAY_AGAIN;
	gameover_button[slect].slect = true;
	gameover_button[slect].Print();
	while(true){
		switch(getch()){
			case 'W':
			case 'w':
			case 72 ://up
				gameover_button[slect].slect = false;
				gameover_button[slect].Print();
				slect = (slect - 1 + GAMEOVERMENU_TOT) % GAMEOVERMENU_TOT;
				gameover_button[slect].slect = true;
				gameover_button[slect].Print();
				break;
			case 'S':
			case 's':
			case 80 ://down
				gameover_button[slect].slect = false;
				gameover_button[slect].Print();
				slect = (slect + 1) % GAMEOVERMENU_TOT;
				gameover_button[slect].slect = true;
				gameover_button[slect].Print();
				break;
			case ' ':
			case 13 ://enter
				return slect;
			default:break;
		}
	}	
}
int SetMenu(){
	Button set_button[OPT_TOT]; //设置的按钮 
	ifstream fin; string tmmp;
	fin.open("data//button//setmenu.button");//载入 
	if(!fin.is_open()){
		system("cls");
		cout<<"数据丢失，请检查数据文件后重新启动游戏";
		exit(0);
	}
	for(int slect = OPT_L_R; slect < OPT_TOT; slect++){
		string s; int c1, c2, xpos, ypos;
		fin >> tmmp >> s >> tmmp;
		fin >> tmmp >> xpos >> tmmp >> ypos >> tmmp;
		fin >> tmmp >> c1 >> c2;
		set_button[slect].Set(s, Point(xpos,ypos), c1, c2);
	}
	fin.close();
	system("cls");
	for(int slect = OPT_L_R; slect < OPT_TOT; slect++){
		set_button[slect].slect = false;
		set_button[slect].Print();
	}
	int slect = set_opt;
	set_button[slect].slect = true;
	set_button[slect].Print();
	Button discribe[OPT_TOT][DISCRIBE_LEN]; int len[OPT_TOT];
	fin.open("data//others//discribe.txt");
	if(!fin.is_open()){
		system("cls");
		cout<<"数据丢失，请检查数据文件后重新启动游戏";
		exit(0);
	}
	for (int i = OPT_L_R; i < OPT_TOT; i++) {
		fin >> tmmp >> len[i];
		for (int j = 1; j <= len[i]; j++) {
			string s; int c1, c2, xpos, ypos;
			fin >> tmmp >> s >> tmmp;
			fin >> tmmp >> xpos >> tmmp >> ypos >> tmmp;
			fin >> tmmp >> c1 >> c2;
			discribe[i][j].Set(s, Point(xpos,ypos), c1, c2);
		}
	}
	fin.close();
	for(int i = 1; i <= len[slect]; i++){
		discribe[slect][i].Print();
		Sleep(SPEED);
	}
	while(true){
		switch(getch()){
			case 'A':
			case 'a':
			case 75 ://left
				set_button[slect].slect = false;
				set_button[slect].Print();
				
				for(int i=1;i<=len[slect];i++)
					discribe[slect][i].pos.Clear();
				
				slect = (slect - 1 + OPT_TOT) % OPT_TOT;
				set_button[slect].slect = true;
				set_button[slect].Print();
				
				for(int i=1;i<=len[slect];i++){
					discribe[slect][i].Print();
					Sleep(SPEED);
				}
				
				break;
			case 'D':
			case 'd':
			case 77 ://right
				set_button[slect].slect = false;
				set_button[slect].Print();
				
				for(int i=1;i<=len[slect];i++)
					discribe[slect][i].pos.Clear();
				
				slect=(slect + 1) % OPT_TOT;
				set_button[slect].slect = true;
				set_button[slect].Print();
				
				for(int i=1;i<=len[slect];i++){
					discribe[slect][i].Print();
					Sleep(SPEED);
				}
				
				break;
			case ' ':
			case 13 ://enter
				return slect;
			default:break;
		}
	}
}
```