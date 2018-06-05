# repository
chess
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <math.h>
enum{PAWN = 'p', QUEEN = 'q', BISHOP = 'e', KING = 'k', ROOK = 'r', HORSE = 'h'};
//	  пешка      ферзь       слон         король      ладья         конь
enum{EMPTY = '0', WHITE = 'w', BLACK = 'b'};
enum{EAT = 'e', MOVE = 'm', AISLE = 'a', TAKEONTHEAISLE = 't'};
char FIGURE[6] = {HORSE, BISHOP, ROOK, QUEEN, KING, PAWN};
struct Board
{
	char figure,  color;
	int ball/*, aisle*/;
};
struct Move
{
	int x, y;
	int x_0, y_0;
	int ball;
	char act;
};
struct  vector
{
	int x, y;
};
int BALL(struct Board chess[8][8], char init);
void print_board(struct Board chess[8][8])
{
	printf("    ");
	for(int f = 0; f < 8; f++)
	{
		printf("%d  ", f+1);
	}
	putchar('\n');
	putchar('\n');
	for(int f = 0; f < 8; f++)
	{
		printf("%d   ", f+1);
		for(int q = 0; q < 8; q++)
		{
				printf("%c%c ", chess[f][q].figure, chess[f][q].color);
		}
		putchar('\n');
	}
		putchar('\n');
}
void replace(struct Board chess[8][8], int i, int j, int i_move, int j_move)
{
	struct Board t;
	t = chess[i][j];
	chess[i][j] = chess[i_move][j_move];
	chess[i_move][j_move] = t;
}
void eat(struct Board chess[8][8], int i, int j, int i_move, int j_move)
{
	replace(chess, i, j, i_move, j_move);
	chess[i][j].figure = EMPTY;
	chess[i][j].color = EMPTY;
	chess[i][j].ball = 0;
}
int confines(struct vector t, int i, int j)
{
	if( (0 <= t.x + i && t.x + i < 8) && (0 <= t.y + j && t.y + j < 8) )
	{
		return 1;
	}
	return 0;
}
int border(struct Board chess[8][8], struct vector vect, int i, int j)
{
	struct vector temp;
	struct Board t;
	for(int k = 1; k < 9; k++)
	{
		temp.x = vect.x * k;
		temp.y = vect.y * k;
		if(confines(temp, i, j))
		{
			temp.x += i;
			temp.y += j;
			t = chess[temp.x][temp.y];
			if(t.color == BLACK)
			{
				return k+1;
			}
			if(t.color == WHITE)
			{
				return k;
			}
		}
		else
		{
			return k;
		}
	}
}
void application(struct Board chess[8][8], struct Move temp)
{
	if(temp.act == EAT)
	{
		eat(chess, temp.x_0, temp.y_0, temp.x, temp.y);
	}
	if(temp.act == MOVE)
	{
		replace(chess, temp.x_0, temp.y_0, temp.x, temp.y);
	}
	if(temp.act == QUEEN)
	{
		chess[temp.x][temp.y].figure = QUEEN;
	}
	if(temp.act == TAKEONTHEAISLE)
	{
		eat(chess, temp.x_0, temp.y_0, temp.x -1, temp.y);
		replace(chess, temp.x -1, temp.y, temp.x, temp.y);
	}
}
void trans(struct Board chess[8][8])
{
	struct Board tr[8][8];
	for(int q = 0, m = 7; q < 8; q++, m--)
	{
		for(int f = 0, n = 7; f < 8; f++, n--)
		{
			if(chess[m][n].color == WHITE)
			{
				chess[m][n].color = BLACK;
			}
			else
			{
				if(chess[m][n].color == BLACK)
				{
					chess[m][n].color = WHITE;
				}
			}
			tr[q][f] = chess[m][n];
		}
	}
	for(int q = 0; q < 8; q++)
	{
		for(int f = 0; f < 8; f++)
		{
			chess[f][q] = tr[f][q];
		}
	}
}

struct Move bishop_and_rook(struct Board chess[8][8], int i, int j, struct vector vect[4], int level);
struct Move pawn(struct Board chess[8][8], int i, int j, struct vector t[4], int level);
struct Move horse_and_king(struct Board chess[8][8], int i, int j, struct vector t[8], int level);
struct Move method_figure(struct Board chess[8][8], int i, int j, int level)
{
	struct vector vect_1[4] = {{ 1,  1}, { 1, -1}, {-1,  1}, {-1, -1}};
	struct vector vect_2[4] = {{-1,  0}, { 1,  0}, { 0,  1}, { 0, -1}};
	struct vector vect_3[8] = {{-1,  0}, { 1,  0}, { 0, -1}, { 0,  1}, { 1, -1}, { 1,  1}, {-1,  1}, {-1, -1}};
	struct vector vect_4[8] = {{-2,  1}, {-2, -1}, { 2,  1}, { 2, -1}, { 1,  2}, { 1, -2}, {-1,  2}, {-1, -2}};
	struct vector vect_5[4] = {{ 1,  0}, { 2,  0}, { 1,  1}, { 1, -1}};
	if(chess[i][j].figure == KING)
	{
		return horse_and_king(chess, i, j, vect_3, level);
	}
	if(chess[i][j].figure == HORSE)
	{
		return horse_and_king(chess, i, j, vect_4, level);
	}
	if(chess[i][j].figure == BISHOP)
	{
		return bishop_and_rook(chess, i, j, vect_1, level);
	}
	if(chess[i][j].figure == ROOK)
	{
		return bishop_and_rook(chess, i, j, vect_2, level);
	}
	if(chess[i][j].figure == QUEEN)
	{
		int f_1 = 0, f_2 = 0;
		struct Move t_1 = bishop_and_rook(chess, i, j, vect_1, level);
		struct Move t_2 = bishop_and_rook(chess, i, j, vect_2, level);
		if(t_1.x == t_1.x_0 && t_1.y == t_1.y_0)
		{
			f_1 = 1;
		}
		if(t_2.x == t_2.x_0 && t_2.y == t_2.y_0)
		{
			f_2 = 1;
		}
		if(f_1 == 0 && f_2 == 0)
		{
			if(t_1.ball < t_2.ball)
			{
				return t_2;
			}
			return t_1;
		}
		if(f_1 == 1)
		{
			return t_2;
		}
		if(f_2 == 1)
		{
			return t_1;
		}
	}
	if(chess[i][j].figure == PAWN)
	{
		return pawn(chess, i, j, vect_5, level);
	}
}
struct Move white(struct Board chess[8][8], int level)
{
	int kol_F = 0, k, flag[16];
	struct Board t;
	struct Move  temp[16], mnim;
	for(int k = 0; k < 16; k++)
	{
		flag[k] = 1;
	}
	for(int k = 0; k < 6; k++)
	{
		for(int i = 0; i < 8; i++)
		{
			for(int j = 0; j < 8; j++)
			{
				t = chess[i][j];
				if(t.figure != EMPTY && t.color == WHITE)
				{
					if(t.figure == FIGURE[k] && t.color == WHITE)
					{
						temp[kol_F] = method_figure(chess, i, j, level);
						if(temp[kol_F].x == i && temp[kol_F].y == j)
						{
							flag[kol_F] = 0;
						}
//						printf("(%d %d)(%d %d) %d %c %d\n", i+1, j+1, temp[kol_F].x+1, temp[kol_F].y+1, temp[kol_F].ball, FIGURE[k], flag[kol_F]);
						kol_F++;
					}
				}
			}
		}
	}
	for(k = 0; k < kol_F; k++)
	{
		if(flag[k] == 1)
		{
			mnim = temp[k];
			break;
		}
	}
	for(k = 0; k < kol_F; k++)
	{
		if(flag[k] == 1)
		{
			if(mnim.ball <= temp[k].ball)
			{
				mnim = temp[k];
			}
		}
	}
	if(kol_F == 0)
	{
		mnim.x = mnim.x_0;
		mnim.y = mnim.y_0;
	}
//	printf("(%d %d)(%d %d) %d  v WHITE\n", mnim.x_0+1, mnim.y_0+1, mnim.x+1, mnim.y+1, mnim.ball);
	return mnim;
}
struct Move solve(struct Board chess[8][8], int level)
{
	struct Move t;
	t = white(chess, level);
	return t;
}
int main(int argc, char **argv)
{
	int t, a, level;
	struct Board chess[8][8];
	struct Move temp;
	FILE *f = fopen(argv[1], "r");
	for(int i = 0; i < 8; i++)
	{
		for(int j = 0; j < 8; j++)
		{
			chess[i][j].figure = fgetc(f);
			chess[i][j].color = fgetc(f);
			if(chess[i][j].figure == PAWN)
			{
				chess[i][j].ball = 1;
//				chess[i][j].aisle = 1;
			}
			if(chess[i][j].figure == HORSE)
			{
				chess[i][j].ball = 3;
			}
			if(chess[i][j].figure == KING)
			{
				chess[i][j].ball = 100;
			}
			if(chess[i][j].figure == QUEEN)
			{
				chess[i][j].ball = 10;
			}
			if(chess[i][j].figure == BISHOP)
			{
				chess[i][j].ball = 3;
			}
			if(chess[i][j].figure == ROOK)
			{
				chess[i][j].ball = 5;
			}
			if(chess[i][j].figure == EMPTY)
			{
				chess[i][j].ball = 0;
			}
		}
		fgetc(f);
	}
	printf("Выберите уровень: 1, 2, 3.\n");
	scanf("%d", &level);
	if(level == 2)
	{
		level = 5;
	}
	if(level == 3)
	{
		level = 7;
	}
	printf("Хотите начать первым?\n");
	printf("1 - да, 0 -нет.\n");
	scanf("%d", &a);
	if(a == 0)
	{
		print_board(chess);
		temp = solve(chess, level);
		application(chess, temp);
		printf(" (%d %d)->(%d %d)\n", temp.x_0+1, temp.y_0+1, temp.x+1, temp.y+1);
	}
	print_board(chess);
	printf("Ваш ход.\n");
	while(scanf("%d", &a) != EOF)
	{
		temp.x_0 = a / 1000 - 1;
		temp.y_0 = a / 100 % 10 - 1;
		temp.x = a / 10 % 10 - 1;
		temp.y = a % 10 - 1;
		if(chess[temp.x][temp.y].color == WHITE)
		{
			temp.act = EAT;
		}
		if(chess[temp.x][temp.y].color == EMPTY)
		{
			temp.act = MOVE;
		}
		if(temp.x == temp.x_0 && temp.y == temp.y_0 && chess[temp.x][temp.y].figure == PAWN)
		{
			temp.act = QUEEN;
		}
		application(chess, temp);
		print_board(chess);
		temp = solve(chess, level);
		application(chess, temp);
		printf(" (%d %d)->(%d %d)\n", temp.x_0+1, temp.y_0+1, temp.x+1, temp.y+1);
		print_board(chess);
		printf("Ваш ход.\n");
	}
	return 0;
}
int BALL(struct Board chess[8][8], char init)
{
	int sum_1 = 0, sum_2 = 0;
	for(int i = 0; i < 8; i++)
	{
		for(int j = 0; j < 8; j++)
		{
			struct Board t = chess[i][j];
			if(t.color == init)
			{
				sum_1 += t.ball;
			}
			if(t.color != EMPTY && t.color != init)
			{
				sum_2 += t.ball;
			}
		}
	}
	return sum_1 - sum_2;
}
struct Move bishop_and_rook(struct Board chess[8][8], int i, int j, struct vector vect[4], int level)
{
	int k, flag[28], mnim, l, s = 0, way[4], m;
	struct Move move[28], move_0;
	move_0.x = i;
	move_0.y = j;
	move_0.ball = BALL(chess, WHITE);
	struct Board food, temp;
	struct vector t, b[4][7];
	for(k = 0; k < 4; k++)
	{
		flag[28] = 0;
	}
	for(k = 0; k < 4; k++)
	{
		for(l = 0, m = 1; l < 7; l++, m++)
		{
			b[k][l].x = vect[k].x * m;
			b[k][l].y = vect[k].y * m;
		}
	}
	for(k = 0; k < 4; k++)
	{
		way[k] = border(chess, vect[k], i, j) - 1;
//		printf("gran(%d)\n", way[k]);
	}
	for(k = 0; k < 4; k++)
	{
		for(l = 0; l < way[k]; l++)
		{
			t = b[k][l];
			temp = chess[i + t.x][j + t.y];
//			printf("%d %d \n", i + t.x +1, j + t.y +1);
			if(temp.color == BLACK)
			{
				food = temp;
				eat(chess, i, j, i + t.x, j + t.y);
				move[s].x = i + t.x;
				move[s].y = j + t.y;
				move[s].ball = BALL(chess, WHITE);
//				printf("(%d %d)(%d %d) %d\n", i+1, j+1, move[s].x + 1, move[s].y + 1, move[s].ball);
				move[s].act = EAT;
				chess[i][j] = food;
				replace(chess, i, j, i + t.x, j + t.y);
				flag[s] = 1;
				s++;
			}
			if(temp.color == EMPTY)
			{
				replace(chess, i, j, i + t.x, j + t.y);
				move[s].x = i + t.x;
				move[s].y = j + t.y;
				move[s].ball = BALL(chess, WHITE);
//				printf("(%d %d)(%d %d) %d\n", i+1, j+1, move[s].x+1, move[s].y+1, move[s].ball);
				replace(chess, i, j, i + t.x, j + t.y);
				flag[s] = 1;
				move[s].act = MOVE;
				s++;
			}
		}
	}
	if(level == 1)
	{
		for(k = 0; k < s; k++)
		{
			if(flag[k] == 1)
			{
				move_0 = move[k];
				break;
			}
		}
		for(k = 0; k < s; k++)
		{
			if(flag[k] == 1)
			{
				if(move_0.ball <= move[k].ball)
				{
					move_0 = move[k];
				}
			}
		}
	}
	if(level > 1)
	{
		for(k = 0; k < s; k++)
		{
			if(flag[k] == 1)
			{
				move_0 = move[k];
				break;
			}
		}
		for(k = 0; k < s; k++)
		{
			if(flag[k] == 1)
			{
				move[k].x_0 = i;
				move[k].y_0 = j;
				if(move[k].act == EAT)
				{
					food = chess[move[k].x][move[k].y];
					eat(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
				if(move[k].act == MOVE)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
				level--;
				trans(chess);
				struct Move h = white(chess, level);
				move[k].ball = h.ball;
				trans(chess);
				level++;
				if(move[k].act == EAT)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
					chess[move[k].x][move[k].y] = food;
				}
				if(move[k].act == MOVE)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
			}
		}
		for(k = 0; k < s; k++)
		{
			if(flag[k] == 1)
			{
				if(move_0.ball <= move[k].ball)
				{
					move_0 = move[k];
				}
			}
		}
	}
	move_0.x_0 = i;
	move_0.y_0 = j;
	return move_0;
}
struct Move pawn(struct Board chess[8][8], int i, int j, struct vector t[4], int level)
{
	int k, flag[4], mnim;
	struct Move move[4], move_0;
	move_0.x = i;
	move_0.y = j;
	move_0.ball = BALL(chess, WHITE);
	struct Board food, temp;
//	struct vector t[6] = {{1, 0}, {2, 0}, {1, 1}, {1, -1}/*, {0, 1}, {0, -1}*/};
//	memset(flag, 0, sizeof(int));
	for(int k = 0; k < 4; k++)
	{
		flag[k] = 0;
	}
/*	if(i == 7)
	{
		chess[i][j].figure = QUEEN;
		chess[i][j].ball = 10;
		move[4].x = i;
		move[4].y = j;
		move[4].act = QUEEN;
		move[4].ball = BALL(chess, WHITE);
		flag[4] = 1;
//		printf("(%d %d) %d\n", move[4].x+1, move[4].y+1, move[4].ball);
		chess[i][j].figure == PAWN;
		chess[i][j].ball == 1;
	}*/
	if(confines(t[0], i, j))
	{
		temp = chess[i + t[0].x][j + t[0].y];
		if(temp.color == EMPTY)
		{
			replace(chess, i, j, i + t[0].x, j + t[0].y);
			move[0].x = i + t[0].x;
			move[0].y = j + t[0].y;
			move[0].ball = BALL(chess, WHITE);
//			printf("(%d %d)(%d %d) %d\n", i+1, j+1,move[0].x+1, move[0].y+1, move[0].ball);
			replace(chess, i, j, i + t[0].x, j + t[1].y);
			flag[0] = 1;
			move[0].act = MOVE;
			if(confines(t[1], i, j))
			{
				temp = chess[i + t[1].x][j + t[1].y];
				if(temp.color == EMPTY && i == 1)
				{
					replace(chess, i, j, i + t[1].x, j + t[1].y);
					move[1].x = i + t[1].x;
					move[1].y = j + t[1].y;
					move[1].ball = BALL(chess, WHITE);
					flag[1] = 1;
//					printf("(%d %d)(%d %d) %d\n", i+1, j+1,move[1].x+1, move[1].y+1, move[1].ball);
					replace(chess, i, j, i + t[1].x, j + t[1].y);
					move[1].act = MOVE;
				}
			}
		}
	}
	for(k = 2; k < 4; k++)
	{
		if(confines(t[k], i, j))
		{
			temp = chess[i + t[k].x][j + t[k].y];
			if(temp.color == BLACK)
			{
				food = temp;
				eat(chess, i, j, i + t[k].x, j + t[k].y);
				move[k].x = i + t[k].x;
				move[k].y = j + t[k].y;
				move[k].ball = BALL(chess, WHITE);
//				printf("(%d %d)(%d %d) %d\n", i+1, j+1,move[k].x+1, move[k].y+1, move[k].ball);
				move[k].act = EAT;
				chess[i][j] = food;
				replace(chess, i, j, i + t[k].x, j + t[k].y);
				flag[k] = 1;
			}
/*			if(temp.color == EMPTY && chess[i + t[k+2].x][j + t[k+2].y].color == BLACK)
			if(i + t[k+2].x == 4 && chess[i + t[k+2].x][j + t[k+2].y].aisle)
			if(chess[i + t[k+2].x][j + t[k+2].y].figure == PAWN)
			{
				food = chess[i + t[k+2].x][j + t[k+2].y];
				eat(chess, i, j, i + t[k+2].x, j + t[k+2].y);
				replace(chess, i + t[k+2].x, j + t[k+2].y, i + t[k].x, j + t[k].y);
				move[k+2].x = i + t[k].x;
				move[k+2].y = j + t[k].y;
				move[k+2].ball = BALL(chess, WHITE);
//				printf("(%d %d) %d\n", move[k+2].x+1, move[k+2].y+1, move[k+2].ball);
				move[k+2].act = TAKEONTHEAISLE;
				chess[i + t[k+2].x][j + t[k+2].y] = food;
				replace(chess, i, j, i + t[k].x, j + t[k].y);
				flag[k+2] = 1;
			}
*/		}
	}
	if(level == 1)
	{
		for(int k = 0; k < 4; k++)
		{
			if(flag[k] == 1)
			{
				move_0 = move[k];
				break;
			}
		}
		for(int k = 0; k < 4; k++)
		{
			if(flag[k] == 1)
			{
				if(move_0.ball < move[k].ball)
				{
					move_0 = move[k];
				}
			}
		}
	}
	if(level > 1)
	{
		for(k = 0; k < 4; k++)
		{
			if(flag[k] == 1)
			{
				move_0 = move[k];
				break;
			}
		}
		for(k = 0; k < 4; k++)
		{
			if(flag[k] == 1)
			{
				move[k].x_0 = i;
				move[k].y_0 = j;
				if(move[k].act == EAT)
				{
					food = chess[move[k].x][move[k].y];
					eat(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
				if(move[k].act == MOVE)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
				level--;
				trans(chess);
//				print_board(chess);
//				printf("(%d %d)->(%d %d) %d black\n", move[k].x_0+1, j+1, move[k].x+1, move[k].y+1, move[k].ball);
				struct Move h = white(chess, level);
				move[k].ball = h.ball;
				trans(chess);
				level++;
				if(move[k].act == EAT)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
					chess[move[k].x][move[k].y] = food;
				}
				if(move[k].act == MOVE)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
//				print_board(chess);
			}
		}
		for(k = 0; k < 4; k++)
		{
			if(flag[k] == 1)
			{
				if(move_0.ball < move[k].ball)
				{
					move_0 = move[k];
				}
			}
		}
	}
	move_0.x_0 = i;
	move_0.y_0 = j;
	return move_0;
}

struct Move horse_and_king(struct Board chess[8][8], int i, int j, struct vector t[8], int level)
{
	int k, flag[8];
	struct Move move[8], move_0;
	move_0.x = i;
	move_0.y = j;
	move_0.ball = BALL(chess, WHITE);
	struct Board food, temp;
	for( k = 0; k < 8; k++)
	{
		flag[k] = 0;
	}
//	print_board(chess);
	for( k = 0; k < 8; k++)
	{
		if(confines(t[k], i, j))
		{
			temp = chess[i + t[k].x][j + t[k].y];
			if(temp.color == BLACK)
			{
				food = temp;
				eat(chess, i, j, i + t[k].x, j + t[k].y);
				move[k].x = i + t[k].x;
				move[k].y = j + t[k].y;
				move[k].ball = BALL(chess, WHITE);
//				printf("(%d %d)(%d %d) %d\n", i+1, j+1,move[k].x+1, move[k].y+1, move[k].ball);
				move[k].act = EAT;
				chess[i][j] = food;
				replace(chess, i, j, i + t[k].x, j + t[k].y);
				flag[k] = 1;
			}
			if(temp.color == EMPTY)
			{
				replace(chess, i, j, i + t[k].x, j + t[k].y);
				move[k].x = i + t[k].x;
				move[k].y = j + t[k].y;
				move[k].ball = BALL(chess, WHITE);
//				printf("(%d %d)(%d %d) %d\n", i+1, j+1,move[k].x+1, move[k].y+1, move[k].ball);
				replace(chess, i, j, i + t[k].x, j + t[k].y);
				flag[k] = 1;
				move[k].act = MOVE;
			}
		}
	}
	if(level == 1)
	{
		for(k = 0; k < 8; k++)
		{
			if(flag[k] == 1)
			{
				move_0 = move[k];
				break;
			}
		}
		for(k = 0; k < 8; k++)
		{
			if(flag[k] == 1)
			{
				if(move_0.ball <= move[k].ball)
				{
					move_0 = move[k];
				}
			}
		}
	}
	if(level > 1)
	{
		for(k = 0; k < 8; k++)
		{
			if(flag[k] == 1)
			{
				move_0 = move[k];
				break;
			}
		}
		for(k = 0; k < 8; k++)
		{
			if(flag[k] == 1)
			{
				move[k].x_0 = i;
				move[k].y_0 = j;
				if(move[k].act == EAT)
				{
					food = chess[move[k].x][move[k].y];
					eat(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
				if(move[k].act == MOVE)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
				level--;
				trans(chess);
//				print_board(chess);
//				printf("(%d %d)->(%d %d) %d black\n", move[k].x_0+1, j+1, move[k].x+1, move[k].y+1, move[k].ball);
				struct Move h = white(chess, level);
				move[k].ball = h.ball;
				trans(chess);
				level++;
				if(move[k].act == EAT)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
					chess[move[k].x][move[k].y] = food;
				}
				if(move[k].act == MOVE)
				{
					replace(chess, move[k].x_0, move[k].y_0, move[k].x, move[k].y);
				}
			}
		}
		for(k = 0; k < 8; k++)
		{
			if(flag[k] == 1)
			{
				if(move_0.ball < move[k].ball)
				{
					move_0 = move[k];
				}
			}
		}
	}
	move_0.x_0 = i;
	move_0.y_0 = j;
	return move_0;
}
