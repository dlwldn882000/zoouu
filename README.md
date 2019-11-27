# zoouu
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <conio.h>				//getch 사용
#include <windows.h>			//gotoxy 사용

#define LEFT 75
#define RIGHT 77
#define UP 72
#define DOWN 80
#define ESC 27
#define ENTER 13
#define SPACE 32
#define BACKSPACE 8

#define COL                   GetStdHandle(STD_OUTPUT_HANDLE)        // 콘솔창의 핸들정보 받기
#define BLACK                SetConsoleTextAttribute(COL, 0x0000);        // 검정색
#define DARK_BLUE         SetConsoleTextAttribute(COL, 0x0001);        // 파란색
#define GREEN                SetConsoleTextAttribute(COL, 0x0002);        // 녹색
#define BLUE_GREEN        SetConsoleTextAttribute(COL, 0x0003);        // 청녹색
#define BLOOD               SetConsoleTextAttribute(COL, 0x0004);        // 검붉은색
#define PURPLE               SetConsoleTextAttribute(COL, 0x0005);        // 보라색
#define GOLD                 SetConsoleTextAttribute(COL, 0x0006);        // 금색
#define ORIGINAL            SetConsoleTextAttribute(COL, 0x0007);        // 밝은 회색 (ORIGINAL CONSOLE COLOR)
#define BLUE                  SetConsoleTextAttribute(COL, 0x0009);        // 파란색
#define HIGH_GREEN       SetConsoleTextAttribute(COL, 0x000a);        // 연두색
#define SKY_BLUE           SetConsoleTextAttribute(COL, 0x000b);        // 하늘색
#define RED                   SetConsoleTextAttribute(COL, 0x000c);        // 빨간색
#define PLUM                SetConsoleTextAttribute(COL, 0x000d);        // 자주색
#define YELLOW             SetConsoleTextAttribute(COL, 0x000e);        // 노란색
#define WHITE                SetConsoleTextAttribute(COL, 0x000f);        // 흰색


#define N 600        // 노드의 갯수
#define m 5000     // 최대 값

FILE* fp;
int data[N][N];

typedef struct Node {
	char numbername[N][20];
	int distance[N]; // 정점 까지의 거리
	int via[N];      // 이전 정점을 가리키는 포인터
	int startnumber;
	int destnumber;
}node;

void nosun(node subway);
void select(node subway, int ccolor, int v1, int v2);
int namecheck(char insert[], node* Subway);
void input(node *Subway, char start[], char dest[], int option);
void shortfind(node *Subway, int option);
void print(node subway, char start[], char dest[], int option);
void save_input(char start[], char dest[]);
void save_output(char start[], char dest[]);
void color(int path[], int i);
void main_print();
void gotoxy(int x, int y);

void main() {
	node subway, *Subway;	//구조체선언
	Subway = &subway;		//구조체포인터 Subway초기화

	int option;
	char start[30];
	char dest[30];

	unsigned int y = 10, y1 = 0;
	WHITE;
	main_print();
	gotoxy(15, 8);
	printf("▶");
	char cursor;
	while (1) {
		cursor = _getch();
		switch (cursor) {
		case UP:
			system("cls");
			main_print();
			y1 = ((--y) % 5 + 8);
			gotoxy(15, y1);
			printf("▶");
			break;
		case DOWN:
			system("cls");
			main_print();
			y1 = ((++y) % 5 + 8);
			gotoxy(15, y1);
			printf("▶");
			break;
		case ENTER:
			system("cls");
			switch (y1) {
			case 8:
				system("cls");
				input(Subway, start, dest, 1);
				nosun(subway);
				break;
			case 9:
				input(Subway, start, dest, 0);
				shortfind(Subway, 0);
				print(subway, start, dest, 0);
				break;
			case 10:
				input(Subway, start, dest, 0);
				shortfind(Subway, 1);
				print(subway, start, dest, 1);
				break;
			case 11:
				save_output(start, dest);
				input(Subway, start, dest, 1);
				shortfind(Subway, 0);
				print(subway, start, dest, y1);
				main_print();
				gotoxy(15, 8);
				printf("▶");
				break;
			case 12:
				exit(0);
				break;
			}
			break;
		}
	}
}

void gotoxy(int x, int y)
{
	COORD Pos = { x - 1, y - 1 };
	SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), Pos);
}

void main_print() {
	GREEN;
	printf("┌──────────────────────────┐\n");
	printf("│                                                    │\n");
	printf("│                 "); WHITE; printf("지하철  노선도"); GREEN;
	printf("                     │\n");
	printf("│                                                    │\n");
	printf("└──────────────────────────┘\n\n\n");
	BLUE;
	printf("\t\t 1. 노선도 보기\n");
	printf("\t\t 2. 최단거리 검색\n");
	printf("\t\t 3. 최소환승 검색\n");
	printf("\t\t 4. 즐겨찾기\n");
	printf("\t\t 5. 종료\n\n");

}

void input(node *Subway, char start[], char dest[], int option) {

	int weight, num1, num2, w = 0;
	int i = 0, s, e, temp = 0;

	// 데이터 초기화
	for (s = 0; s<N; s++) {
		for (e = 0; e<N; e++) {
			if (s == e)
				data[s][e] = 0;
			else
				data[s][e] = m;
		}
	}

	fp = fopen("subway.txt", "r");					  //subway파일불러오기
	while (w != EOF) {
		w = fscanf(fp, "%d %d %d", &num1, &num2, &weight);//data이차행렬에 가중치넣기
		w = fscanf(fp, "%s", &Subway->numbername[num1]);
		data[num1 - 1][num2 - 1] = weight;
		data[num2 - 1][num1 - 1] = weight;
	}
	fclose(fp);

	if (option == 0) {
		GREEN;
		printf("┌──────────────────────────┐\n");
		printf("│                                                    │\n");
		printf("│                 "); WHITE; printf("지하철  노선도"); GREEN;
		printf("                     │\n");
		printf("│                                                    │\n");
		printf("└──────────────────────────┘\n\n\n");
		BLUE;
		//출발 및 도착역 고유번호 찾기
		while (1) {
			printf("\t 출발점을 입력하시오. : ");
			scanf("%s", start);
			if (namecheck(start, Subway))
				break;
			else
				printf("\t 잘못 입력하셨습니다. \n");

		}
		while (1) {
			printf("\n\t 도착점을 입력하시오. : ");
			scanf("%s", dest);
			if (namecheck(dest, Subway))
				break;
			else
				printf("\t 잘못 입력하셨습니다. \n");

		}
	}

	for (i = 0; i<N; i++) {						//출발역이름에 따른 고유번호찾기
		if (!strcmp(Subway->numbername[temp], start)) {
			Subway->startnumber = temp;
			break;
		}
		temp++;
	}

	temp = 0;
	for (i = 0; i<N; i++) {						//도착역이름에 따른 고유번호찾기
		if (!strcmp(Subway->numbername[temp], dest)) {
			Subway->destnumber = temp;
			break;
		}
		temp++;
	}
}

void shortfind(node *Subway, int option) {		//구조체안에 배열값을 변경하기위해
	int i = 0, j, k, min;											//구조체포인터 사용(Call by reference)
	int v[N];

	//최소 환승일경우 데이터 초기화
	if (option == 1)
	{
		for (i = 0; i<N; i++) {
			for (j = 0; j<N; j++) {
				if (i == j) {
					data[i][j] = 0;
				}
				if (!strcmp(Subway->numbername[i + 1], Subway->numbername[j + 1])) {
					data[i][j] += 500;			//환승역일경우 가중치 500을 더해서 최소환승을 함
				}
			}
		}
	}

	// 시작지점으로 부터 각 지점까지의 최단거리 구하기
	for (j = 0; j<N; j++)
	{
		v[j] = 0;
		Subway->distance[j] = m;
	}
	//2.시작 지점부터 시작 하도록 지정
	Subway->distance[Subway->startnumber - 1] = 0;
	//3.정점의 수만큼 돈다.
	for (i = 0; i<N; i++)
	{
		//4. 최단거리인 정점을 찾는다.
		min = m;
		for (j = 0; j<N; j++)
		{
			if (v[j] == 0 && Subway->distance[j] < min) {
				k = j;
				min = Subway->distance[j];
			}
		}
		v[k] = 1;               // 최단거리가 확인된 정점
		if (min == m)break;        // error 그래프가 연결되어있지 않음
								   // k를 경유해서 j에 이르는 경로가 최단거리이면 갱신
		for (j = 0; j<N; j++)
		{
			if (Subway->distance[j] > Subway->distance[k] + data[k][j])
			{
				Subway->distance[j] = Subway->distance[k] + data[k][j];
				Subway->via[j] = k;
			}
		}
	}
}

void print(node subway, char start[], char dest[], int option) {		//값 변경이 없으므로 구조체를 매개변수로 사용 (call by value)
	int path[N], path_cnt = 0;
	int i = 0, k, temp = 600, j = 0;
	int count = 0;
	float cost = 0;

	//이동경로 저장
	k = subway.destnumber - 1;
	while (1)
	{
		path[path_cnt++] = k;												//path[]에 이동 경로 저장
		if (k == (subway.startnumber - 1))break;
		k = subway.via[k];
	}

	//이동 경로 출력
	BLUE; printf("\n\t 1호선"); GREEN; printf(" 2호선"); RED; printf(" 3호선"); DARK_BLUE; printf(" 4호선");
	PURPLE; printf(" 5호선"); BLOOD; printf(" 6호선"); HIGH_GREEN; printf(" 7호선"); PLUM; printf(" 8호선\n");
	GOLD; printf("\t 9호선"); YELLOW; printf(" 분당선"); SKY_BLUE; printf(" 인천1호선"); BLUE_GREEN; printf(" 중앙선");
	BLUE_GREEN; printf(" 경의선"); WHITE; printf(" 공항철도"); HIGH_GREEN; printf(" 경춘선\n");
	WHITE; printf("\n 경로는 : ");
	while (!(strcmp(start, subway.numbername[path[path_cnt - 1] + 1]))) {			//출발역이 환승역일경우 예외처리
		path_cnt--;
		if (option == 1 && !(strcmp(start, subway.numbername[path[path_cnt - 1] + 1])))	//최소환승이며 출발역이 환승역일경우
		{
			subway.distance[subway.destnumber - 1] -= 500;
		}
	}

	for (i = path_cnt; i >= 1; i--)
	{
		if (!(strcmp(subway.numbername[temp], subway.numbername[path[i] + 1]))) {		//환승역 두번출력 제거
			count++;
			WHITE;
			j++;
			if (j % 4 == 0) {
				j = 0;
				printf("\n\t  ");
			}
			printf(" [환승] ");
			printf("─▶ ");
			if (option == 1)
			{
				subway.distance[subway.destnumber - 1] -= 500;
			}
			continue;
		}
		if (!(strcmp(dest, subway.numbername[path[i] + 1])))					//도착역이 환승역일경우 예외처리
		{
			if (option == 1 && !(strcmp(dest, subway.numbername[path[i] + 1])))	//최소환승이며 도착역이 환승역일경우
			{
				subway.distance[subway.destnumber - 1] -= 500;
			}
			break;
		}
		color(path, i);
		j++;
		if (j % 4 == 0) {
			j = 0;
			printf("\n\t  ");
		}
		printf("%s ", &subway.numbername[path[i] + 1]);
		WHITE;
		printf("─▶ ");
		temp = path[i] + 1;
	}
	j++;
	if (j % 4 == 0) {
		j = 0;
		printf("\n\t  ");
	}
	color(path, i);
	printf("%s\n\n", &subway.numbername[path[i] + 1]);
	WHITE;

	//요금
	cost = 0.6*(float)subway.distance[subway.destnumber - 1];		//1분에 0.6km라고 가정
	if (cost<10)
		cost = 1000;					//10km이내 1000원
	else if (cost<40) {					//10~40km 구간 5km마다 100원 추가
		temp = cost / 5;
		temp = temp - 2;
		cost = 1000 + temp * 100;
	}
	else {								//40km이후 10km마다 100원 추가
		temp = cost / 10;
		temp = temp - 4;
		cost = 1600 + temp * 100;
	}

	printf(" 환     승 : %d번\n", count);
	printf(" 소요 시간 : %d시간 %d분\n", (subway.distance[subway.destnumber - 1] / 60), (subway.distance[subway.destnumber - 1] % 60));
	printf(" 요     금 : %d원 \n", (int)cost);
	_getch();
	system("cls");
	if (option != 11) {
		save_input(start, dest);
	}

}

void color(int path[], int i) {
	if ((path[i] + 1) <= 99) {
		BLUE;
	}
	else if ((path[i] + 1) <= 151) {
		GREEN;
	}
	else if ((path[i] + 1) <= 194) {
		RED;
	}
	else if ((path[i] + 1) <= 241) {
		DARK_BLUE;
	}
	else if ((path[i] + 1) <= 292) {
		PURPLE;
	}
	else if ((path[i] + 1) <= 330) {
		BLOOD;
	}
	else if ((path[i] + 1) <= 372) {
		HIGH_GREEN;
	}
	else if ((path[i] + 1) <= 389) {
		PLUM;
	}
	else if ((path[i] + 1) <= 413) {
		GOLD;
	}
	else if ((path[i] + 1) <= 433) {
		YELLOW;
	}
	else if ((path[i] + 1) <= 472) {
		SKY_BLUE;
	}
	else if ((path[i] + 1) <= 500) {
		BLUE_GREEN;
	}
	else if ((path[i] + 1) <= 520) {
		BLUE_GREEN;
	}
	else if ((path[i] + 1) <= 529) {
		WHITE;
	}
	else if ((path[i] + 1) <= 547) {
		HIGH_GREEN;
	}
}

void save_input(char start[], char dest[]) {
	char ch[2];
	int count = 0, w = 0;
	GREEN;
	printf("┌──────────────────────────┐\n");
	printf("│                                                    │\n");
	printf("│                 "); WHITE; printf("지하철  노선도"); GREEN;
	printf("                     │\n");
	printf("│                                                    │\n");
	printf("└──────────────────────────┘\n\n\n");
	BLUE;
	printf("\t즐겨찾기에 추가하시겠습니까? (y/n)");
	scanf("%s", ch);
	if (!strcmp(ch, "y")) {
		fp = fopen("save.txt", "a");
		fprintf(fp, "%s %s\n", start, dest);
		fclose(fp);
	}
	system("cls");
	main_print();
	gotoxy(15, 8);
	printf("▶");
}

void save_output(char start[], char dest[]) {
	int w = 0, i = 0, j;
	char estart[10][30];
	char edest[10][30];
	fp = fopen("save.txt", "r");		//파일을 읽기전용으로 연다
	if (fp == NULL) {							//열린 파일이 없으면
		fclose(fp);
		printf("즐겨찾기가 없습니다.\n");	//에러문구 출력하고 다시 반복
		_getch();
		system("cls");
		main_print();
		gotoxy(15, 8);
		printf("▶");
	}
	else {
		while (w != EOF) {
			w = fscanf(fp, "%s %s", estart[i], edest[i]);
			i++;
		}

		GREEN;
		printf("┌──────────────────────────┐\n");
		printf("│                                                    │\n");
		printf("│                 "); WHITE; printf("지하철  노선도"); GREEN;
		printf("                     │\n");
		printf("│                                                    │\n");
		printf("└──────────────────────────┘\n\n\n");
		BLUE;
		printf("\t 번호         출발역         도착역\n");
		for (j = 0; j<(i - 1); j++) {
			printf("\t  %2d%15s%15s\n", j + 1, estart[j], edest[j]);
		}
		printf("\n\t 원하는 번호를 입력하세요 : ");
		scanf("%d", &w);
		strcpy(start, estart[w - 1]);
		strcpy(dest, edest[w - 1]);
		fclose(fp);
	}

}

void nosun(node subway) {
	unsigned number;
	int count = 0;
	int i;
	char s[20];
	GREEN;
	printf("┌──────────────────────────┐\n");
	printf("│                                                    │\n");
	printf("│                 "); WHITE; printf("지하철  노선도"); GREEN;
	printf("                     │\n");
	printf("│                                                    │\n");
	printf("└──────────────────────────┘\n\n\n");
	BLUE; printf(" 1. 1호선"); GREEN; printf(" 2. 2호선"); RED; printf(" 3. 3호선"); DARK_BLUE; printf(" 4. 4호선");
	PURPLE; printf(" 5. 5호선"); BLOOD; printf(" 6. 6호선"); HIGH_GREEN; printf(" 7. 7호선"); PLUM; printf(" 8. 8호선\n");
	GOLD; printf(" 9. 9호선"); YELLOW; printf(" 10. 분당선"); SKY_BLUE; printf(" 11. 인천1호선"); BLUE_GREEN; printf(" 12. 중앙선");
	BLUE_GREEN; printf(" 13. 경의선"); WHITE; printf(" 14. 공항철도"); HIGH_GREEN; printf(" 15. 경춘선\n");
	WHITE;
	while (1) {
		printf("\n");
		printf("\t\t번호를 입력하세요 :");
		scanf("%s", s);

		number = atoi(s);

		if (strlen(s) > 3 || number < 1 || number > 16)
			puts("\t\t잘못 입력하셨습니다.");
		else
			break;
	}
	printf("\n");
	switch (number) {
	case 1:
		for (i = 1; i<43; i++) {
			BLUE;
			printf("%s ", &subway.numbername[i]);
			WHITE;
			printf("◀─▶");
			count++;
			if ((count % 5) == 0)
				printf("\n");
		}
		for (i = 43; i<63; i++) {
			BLUE;
			printf("%s ", &subway.numbername[i]);
			WHITE;
			printf("◀─▶");
			count++;
			if ((count % 3) == 0)
				printf("\n\t\t    │  ");
		}
		BLUE;
		printf("%s \n\t\t    ", &subway.numbername[63]);
		WHITE;
		printf("└▶");
		count++;
		for (i = 64; i<99; i++) {
			BLUE;
			printf("%s ", &subway.numbername[i]);
			WHITE;
			printf("◀─▶");
			count++;
			if ((count % 3) == 0)
				printf("\n\t\t\t");
		}
		BLUE;
		printf("%s", &subway.numbername[99]);
		break;
	case 2:
		select(subway, 2, 100, 151);
		break;
	case 3:
		select(subway, 10, 152, 194);
		break;
	case 4:
		select(subway, 1, 195, 241);
		break;
	case 5:
		for (i = 242; i<281; i++) {
			PURPLE;
			printf("%s ", &subway.numbername[i]);
			WHITE;
			printf("◀─▶");
			count++;
			if ((count % 5) == 0)
				printf("\n");
		}
		for (i = 281; i<287; i++) {
			PURPLE;
			printf("%s ", &subway.numbername[i]);
			WHITE;
			printf("◀─▶");
			count++;
			if ((count % 4) == 0)
				printf("\n\t\t    │  ");
		}
		PURPLE;
		printf("%s \n\t\t    ", &subway.numbername[287]);
		WHITE;
		printf("└▶");
		for (i = 288; i<292; i++) {
			PURPLE;
			printf("%s ", &subway.numbername[i]);
			WHITE;
			printf("◀─▶");
			count++;
			if ((count % 3) == 0)
				printf("\n\t\t\t");
		}
		PURPLE;
		printf("%s", &subway.numbername[292]);
		break;
	case 6:
		select(subway, 4, 293, 330);
		break;
	case 7:
		select(subway, 8, 331, 372);
		break;
	case 8:
		select(subway, 11, 373, 389);
		break;
	case 9:
		select(subway, 6, 390, 413);
		break;
	case 10:
		select(subway, 12, 414, 433);
		break;
	case 11:
		select(subway, 9, 444, 472);
		break;
	case 12:
		select(subway, 3, 473, 500);
		break;
	case 13:
		select(subway, 3, 501, 520);
		break;
	case 14:
		select(subway, 13, 521, 529);
		break;
	case 15:
		select(subway, 8, 530, 547);
		break;
	}
	_getch();
	system("cls");
	main_print();
	gotoxy(15, 8);
	printf("▶");
}

int namecheck(char insert[], node* Subway) {
	int i;
	for (i = 0; i<N; i++) {
		if (!strcmp(insert, Subway->numbername[i]))
			return 1;
	}
	return 0;
}

void select(node subway, int ccolor, int v1, int v2) {
	int i = 0, count = 0;
	for (i = v1; i<v2; i++) {
		switch (ccolor) {
		case 1:
			DARK_BLUE;
			break;
		case 2:
			GREEN;
			break;
		case 3:
			BLUE_GREEN;
			break;
		case 4:
			BLOOD;
			break;
		case 5:
			PURPLE;
			break;
		case 6:
			GOLD;
			break;
		case 7:
			BLUE;
			break;
		case 8:
			HIGH_GREEN;
			break;
		case 9:
			SKY_BLUE;
			break;
		case 10:
			RED;
			break;
		case 11:
			PLUM;
			break;
		case 12:
			YELLOW;
			break;
		case 13:
			WHITE;
			break;
		}
		printf("%s ", &subway.numbername[i]);
		WHITE;
		printf("◀─▶");
		count++;
		if ((count % 5) == 0)
			printf("\n");
	}
	switch (ccolor) {
	case 1:
		DARK_BLUE;
		break;
	case 2:
		GREEN;
		break;
	case 3:
		BLUE_GREEN;
		break;
	case 4:
		BLOOD;
		break;
	case 5:
		PURPLE;
		break;
	case 6:
		GOLD;
		break;
	case 7:
		BLUE;
		break;
	case 8:
		HIGH_GREEN;
		break;
	case 9:
		SKY_BLUE;
		break;
	case 10:
		RED;
		break;
	case 11:
		PLUM;
		break;
	case 12:
		YELLOW;
		break;
	case 13:
		WHITE;
		break;
	}
	printf("%s", &subway.numbername[v2]);
}
