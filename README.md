#include<string.h>
#include<ctype.h>
#include<malloc.h> /* malloc()等 */
#include<limits.h> /* INT_MAX等 */
#include<stdio.h> /* EOF(=^Z或F6),NULL */
#include<stdlib.h> /* atoi() */
/* 函数结果状态代码 */
#define TRUE 1
#define FALSE 0
#define OK 1
#define ERROR 0
#define INFEASIBLE -1
#define OVERFLOW 0
typedef int Status; /* Status是函数的类型,其值是函数结果状态代码，如OK等 */
typedef int Boolean; /* Boolean是布尔类型,其值是TRUE或FALSE */
#define MAXLENGTH 25 /* 设迷宫的最大行列为25 */
#define STACK_INIT_SIZE 10 /* 存储空间初始分配量 */
#define STACKINCREMENT 2 /* 存储空间分配增量 */
typedef struct
{
    int r, c;                // 以行号和列号作为“坐标位置”类型
} PosType;
typedef struct
{
    int ord;                // 通道块在路径上的序号
    PosType seat;        // 通道块在迷宫中的“坐标位置”
    int di;                        // 从此通道块走向下一通道块的“方向”
} SElemType;                       // 定义堆栈元素的类型
typedef struct
{
    SElemType * base;                        // 在栈构造之前和销毁之后，base的值为NULL
    SElemType * top;                        // 栈顶指针
    int stacksize;                                // 当前已分配的存储空间，以元素为单位
} SqStack;
typedef struct
{
    char arr[10][11];
} MazeType;       // 定义迷宫类型（二维字符数组）
/* 定义墙元素值为0,可通过路径为1,不能通过路径为-1,通过路径为足迹 */
Status Pass(MazeType MyMaze, PosType CurPos)
{
    if (MyMaze.arr[CurPos.r][CurPos.c]==' ' || MyMaze.arr[CurPos.r][CurPos.c]=='S' || MyMaze.arr[CurPos.r][CurPos.c]=='E')
        return 1; // 如果当前位置是可以通过，返回1
    else
        return 0; // 其它情况返回0
}
void FootPrint(MazeType &MyMaze, PosType CurPos)
{
    MyMaze.arr[CurPos.r][CurPos.c] = '*';
}
PosType NextPos(PosType CurPos, int Dir)
{
    PosType ReturnPos;
    switch (Dir)
    {
    case 1:
        ReturnPos.r = CurPos.r;
        ReturnPos.c = CurPos.c + 1;
        break;
    case 2:
        ReturnPos.r = CurPos.r + 1;
        ReturnPos.c = CurPos.c;
        break;
    case 3:
        ReturnPos.r = CurPos.r;
        ReturnPos.c = CurPos.c - 1;
        break;
    case 4:
        ReturnPos.r = CurPos.r - 1;
        ReturnPos.c = CurPos.c;
        break;
    }
    return ReturnPos;
}
void MarkPrint(MazeType &MyMaze, PosType CurPos)
{
    MyMaze.arr[CurPos.r][CurPos.c] = '!';
}
Status InitStack(SqStack *S)
{
    /* 构造一个空栈S */
    (*S).base=(SElemType *)malloc(STACK_INIT_SIZE*sizeof(SElemType));
    if(!(*S).base)
        exit(OVERFLOW); /* 存储分配失败 */
    (*S).top=(*S).base;
    (*S).stacksize=STACK_INIT_SIZE;
    return OK;
}
Status Push(SqStack *S,SElemType e)
{
    /* 插入元素e为新的栈顶元素 */
    if((*S).top-(*S).base>=(*S).stacksize) /* 栈满，追加存储空间 */
    {
        (*S).base=(SElemType *)realloc((*S).base,((*S).stacksize+STACKINCREMENT)*sizeof(SElemType));
        if(!(*S).base)
            exit(OVERFLOW); /* 存储分配失败 */
        (*S).top=(*S).base+(*S).stacksize;
        (*S).stacksize+=STACKINCREMENT;
    }
    *((*S).top)++=e;
    return OK;
}
Status StackEmpty(SqStack S)
{
    /* 若栈S为空栈，则返回TRUE，否则返回FALSE */
    if(S.top==S.base)
        return TRUE;
    else
        return FALSE;
}
Status Pop(SqStack *S,SElemType *e)
{
    /* 若栈不空，则删除S的栈顶元素，用e返回其值，并返回OK；否则返回ERROR */
    if((*S).top==(*S).base)
        return ERROR;
    *e=*--(*S).top;
    return OK;
}
Status MazePath(MazeType &maze, PosType start, PosType end)
{
    // 算法3.3
    // 若迷宫maze中从入口 start到出口 end的通道，则求得一条存放在栈中
    // （从栈底到栈顶），并返回TRUE；否则返回FALSE
    SqStack S;
    PosType curpos;
    int curstep;
    SElemType e;
    InitStack(&S);
    curpos = start; // 设定"当前位置"为"入口位置"
    curstep = 1; // 探索第一步
    do
    {
        if (Pass(maze, curpos))   // 当前位置可通过，即是未曾走到过的通道块
        {
            FootPrint(maze, curpos); // 留下足迹
            e.di = 1;
            e.ord = curstep;
            e.seat = curpos;
            Push(&S, e); // 加入路径
            if (curpos.r == end.r && curpos.c == end.c)
                return (TRUE); // 到达终点（出口）
            curpos = NextPos(curpos, 1); // 下一位置是当前位置的东邻
            curstep++; // 探索下一步
        }
        else     // 当前位置不能通过
        {
            if (!StackEmpty(S))
            {
                Pop(&S, &e);
                while (e.di == 4 && !StackEmpty(S))
                {
                    MarkPrint(maze, e.seat);
                    Pop(&S, &e); // 留下不能通过的标记，并退回一步
                } // while
                if (e.di < 4)
                {
                    e.di++;
                    Push(&S, e); // 换下一个方向探索
                    curpos = NextPos(e.seat, e.di); // 当前位置设为新方向的相邻块
                } // if
            } // if
        } // else
    }
    while (!StackEmpty(S));
    return FALSE;
} // MazePath
int main()
{
    int i, j;
    PosType start, end;                                                // 起点终点坐标
    MazeType maze;                                                        // 迷宫
    memset(maze.arr, 0, sizeof(maze.arr)); // 将字符串设置为空
    for(i=0; i<10; i++) // 读取迷宫数据
    {
        gets(maze.arr[i]);
        for(j=0; j<10; j++)
        {
            if(maze.arr[i][j] == 'S') // 获得起点坐标
            {
                start.r = i;
                start.c = j;
            }
            else if(maze.arr[i][j] == 'E')  // 获得终点坐标
            {
                end.r = i;
                end.c = j;
            }
        }
    }
    MazePath(maze, start, end); // 移动
    for(i=0; i<10; i++) // 输出状态
    {
        puts(maze.arr[i]);
    }
    return 0;
}
