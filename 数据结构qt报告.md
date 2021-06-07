# 北邮校园导航系统

设计者：陈哲安	班级：2018214101	学号：2018212432 	完成日期：2020年12月11日	

![2](../../../markdown_photo/2-1607610854345.jpg)

[toc]

## 设计思路

- 第一步，设计出两个按钮，对应两个主要的窗口：**学生端窗口**和**管理员端窗口**，对应两个窗口类，并增加能够实现所需功能的函数：

  **Student_window**

  实现的功能

  - 在窗口画出地图，并能够根据不同的指令来进行不同的画图操作，实际项目里，设定了一个int型变量 drawing_state , 每个drawing_state 在paintevent里对应不同的画图操作
  - 找路函数
  - 查询景点信息的函数，实际以点击景点来展示
  - 查询景点，并在图中标注出来
  - 读取景点和道路数据

  **Administrator_window**
  实现的功能

  - 添加路径
  - 修改路径权值
  - 删除路径
  - 添加景点
  - 画出所有景点和所有道路

- 第二步，对应不同的功能，如果需要建立新的ui界面，那就进行对应的ui界面设计，如果需要新的功能，就去针对各个功能进行定制设计，有时候会需要在各个窗口之间互相传参，这里就需要根据实际需求来进行相应的设计

- 第三步，对于项目的各个底层功能进行检查，在这个过程中修复发现的bug

- 第四步，进行项目的优化与程序的外观设计美化

- 第五步，打包整个程序

## 输入/输出形式，测试数据与预想结果说明

输入的形式

（1）点击地图上的两个景点，然后分别选择**设为起点**和**设为终点**

输出：从起点到终点的最短路

（2）点击地图上的景点

输出：景点名称和介绍

（3）查询窗口，输出待查询的景点名称

输出：在地图对应地方圈出景点



测试数据(请在文件夹下找到这两个文件)：

**road.txt**

**point.txt**



预想结果是：No bug !!!

## 数据结构类型说明与定义

主要就是一个数据结构，自定义的 Graph 类

还有一些琐碎的结构体定义

```c++
class Graph
{
public:
    Graph();

    // 全部设为外部声明
public:
    static const int max_num_points =500;   //假设最大顶点数为  max_num_points, 由txt文件里有多少数据决定，在.cpp文件里重新赋值，默认为50;
    static const int INF = 999999;          //无穷大

    // 图的实现，和最短路径算法声明 //
    struct ArcCell    //边信息
    {
        int distance_kind;//道路长度的类型 -1是默认长度，1是自定义长度
        int kind; //道路类型 ，主干道，无名小路？
        int distance;//边权
    };
    struct PointCell  //点信息
    {
        int number;  // 目标编号
        int x,y;//目标坐标
        int type = 0;// 判断是什么类型的景点，默认为0：普通景点
        QString name;// 目标名称
        QString description; // 目标描述
    };
    struct Map_Graph //地图结构
    {
        PointCell point[max_num_points];                      //景点集合
        ArcCell arc[max_num_points][max_num_points];          //临接矩阵
        int point_num;                                        //顶点数
        int arc_num;                                          //边数
        //int kind;                                           //图的类型
    };
    //全局图
    Map_Graph graph;

    int prev[max_num_points];                  //最短路上的前驱顶点构成的集合
    int d[max_num_points];                     //表示边e = (u,v)的权值(不存在时为INF,不过d[i][i]=0)
    bool used[max_num_points];                 //已经求得最短路的点集合


public:
    QVector<int> get_Path(int startPos,int endPos);         //求到顶点endPos的最短路
    void init_mapgragh(int num);                      //初始化这个校园图
    //void CreateGraph();
    void dijkstra(int startPos);               //求从起点startPos出发到各个顶点的最短距离（dijkstra算法）
};
```

## 项目框架说明

<img src="../../../markdown_photo/image-20201210212657197.png" alt="image-20201210212657197" style="zoom:67%;" />**整个项目的所有文件**

##### 项目的文件关系

![mainwindow .h(.cpp)主窗口](../../../markdown_photo/mainwindow .h(.cpp)主窗口.png)

##### 下面讲解学生端和管理员端的变量与函数

学生端:

```c++
    //map用来存储有关北京邮电大学校园图和图节点的信息
    Graph BUPT_map;

public:
    QString map_file;     //地图文件名
    QString point_file;   //景点文件名
    QString road_file;    //道路文件名

    void init(QString pointfile, QString roadfile);//用来传递景点，道路文件名，用于界面的初始化以及其他操作
private:
    QLabel *x_pos;        //鼠标位置，x坐标
    QLabel *y_pos;        //鼠标位置，y坐标

public:
    explicit Student_window(QWidget *parent = nullptr);
    ~Student_window();

public:
    int read_data(FILE*p);                            //读取数据         
    void read_data();								  //读取数据，调用上面两个函数
    void input_point_data(QList<QString> a);	      //保存道路文件数据到Graph中
    void input_road_data(QList<QString> a);           //保存景点文件数据到Graph中
    void shortest_find_and_draw(int start_pos, int end_pos);//找到最短路并在图中画出来
    QPixmap pix;                                      //用来保存图片的Qpixmap
    QVector<int> active_shortest_path;                //用来记录当前最短路
    int start_pos=-1,								  //起点
    end_pos=-1;                                       //终点
    int drawing_state=0;                              //画图状态
    int switch_state;                                 //接收point_display发出的信号的函数，用来判断设置的是起点还是终点

    int point1=-1;                                     //最终设置的起点
    int point2=-1;                                     //最终设置的起点

    QString place_that_want_to_search;                 //找到当前要查询的地点

protected:
    void mouseMoveEvent(QMouseEvent *event);           //鼠标移动事件。用来获取鼠标的坐标							
    void paintEvent(QPaintEvent *);                    //画图事件，所有的画图都是利用这个函数完成
    void drawRoad(QPainter *p, int kind, int x1, int y1, int x2, int y2);
                                                       //画一条道路
    void drawPoint(int type, int x, int y, int number);//在地图上以按钮的形式"画"出点来
private slots:
    void point_display(int number);                     //用来连接display类窗口
    void on_pushButton_rechoose_clicked();              //重新选择按钮
    void on_pushButton_return_clicked();                //返回主窗口按钮
    void receiveData(int data);                         //接受函数，用来接受选择的是起点还是终点的 0/1 数
    void receive_place_serch(QString name);             //用来接受查的是什么名字的景点
    void on_pushButton_place_search_clicked();          //景点查询按钮事件
```

管理员端：

```c++
public:
    explicit Administrator_window(QWidget *parent = nullptr);
    ~Administrator_window();

public:
    QPixmap pix;                    //放地图照片
    Graph BUPT_map;                 //map用来存储有关北京邮电大学校园图和图节点的信息
    QString pointfile;
    QString roadfile;
    int drawing_state=0;
    void addroad();
    void input_road_data(QList<QString> a);
    void read_data();
    void drawRoad(QPainter *p, int type, int x1, int y1, int x2, int y2);
    void drawPoint(int type, int x, int y, int i);
    void drawPoint(int type, int x, int y, int i, Administrator_window *active);
    void drawing_all_path();        //画出所有路
    int simple_distance(int x1, int y1, int x2, int y2);//欧式距离
    void gragh_to_txt();            //将图中所有路的信息写回道路文件
    void init(QString pointfile, QString roadfile); //主要是为了刷新窗口传参使用

protected:
    void mouseMoveEvent(QMouseEvent *event);
    void paintEvent(QPaintEvent *);             //画图事件
    void input_point_data(QList<QString> a);
    void mousePressEvent(QMouseEvent *event);

private slots:
    //    void on_pushButton_clicked();
    void on_pushButton_2_clicked();
    void place_clicked(int );                    //地图景点点击事件（为了删除景点）
    void on_close_button_clicked();              //返回按钮事件
    void on_pushButton_delete_clicked();         //删除道路按钮事件

    void on_pushButton_clicked();
```

##### 下面说明graph中实现的函数

```c++
//初始化
void Graph::init_mapgragh(int num)
{
    //num表示point.txt中的数据数量
    graph.point_num = num;                        //初始化点数目

    for (int i = 0; i < graph.point_num; i++)
    {
        for (int j = 0; j < graph.point_num; j++)
        {
            if (i == j)
                graph.arc[i][j].distance = 0;
            else
                graph.arc[i][j].distance = INF;
            graph.arc[i][j].distance_kind = 0;
        }
    }
}

//迪杰斯特拉算法，用来算startpos到其他地方的最短路（实际被下面调用）
void Graph::dijkstra(int startPos)
{
    for(int i=0;i<graph.point_num; i++)
    {
        d[i]=graph.arc[startPos][i].distance;

        used[i]=false;

        if( i != startPos)
            prev[i]=startPos;
        //        /*下面这个语句表示如果当前点到源点之间无路，
        //         *则没有最短路径
        //         *但是我的思路是：即使两点之间无路，还是让前驱节点
        //         *指向源点，这样在图上永远会有一条最短路，哪怕是
        //         *两点之间的直接连线*/
        //          if( i != startPos && d[i] < this->INF)
        //               prev[i]=startPos;
        //          else
        //               prev[i]=-1;
    }

    //源点指向-1，因为下面获取最短路径的时候，循环的终止条件就是 prev[i]==-1
    prev[startPos]=-1;
    d[startPos] = 0;
    used[startPos]=true;

    //核心算法
    for(int i=0;i<graph.point_num-1;i++)
    {
        int min=this->INF;
        int u=startPos;

        for(int j=0;j<graph.point_num;j++)
        {
            if( (!used[j]) && (d[j]<min) )
            {
                u=j;
                min=d[j];
            }
        }

        used[u]=true;

        for(int k=0;k<graph.point_num;k++)
        {
            if( (!used[k]) && (graph.arc[u][k].distance!=this->INF) )
            {
                if(d[u]+graph.arc[u][k].distance<d[k])
                {
                    d[k]=d[u]+graph.arc[u][k].distance;
                    prev[k]=u;
                }
            }
        }

    }

}

//真正的找路算法，参数一看就懂
QVector<int> Graph::get_Path(int startpos,int endPos)
{
    dijkstra(startpos);

    QVector<int> path;

    //倒序的获取最短路径上的结点
    for ( ; endPos != -1; endPos = prev[endPos])
    {
        path.push_back (endPos);
    }

    std::reverse(path.begin (), path.end ());

    return path;
}

```

## 项目一览

###### 运行前的动画

<img src="../../../markdown_photo/bupt_excited.gif" alt="bupt_excited" style="zoom: 67%;" />



###### 主界面

<img src="../../../markdown_photo/image-20201210215426044.png" alt="image-20201210215426044" style="zoom: 67%;" />



###### 学生端：

点击查询地点，输入"东门" 

<img src="../../../markdown_photo/image-20201210203249459.png" alt="image-20201210203249459" style="zoom: 50%;" />

点击确认 

<img src="../../../markdown_photo/image-20201210203422134.png" alt="image-20201210203422134" style="zoom: 50%;" />

点击 "体育馆"的图标

<img src="../../../markdown_photo/image-20201210203642668.png" alt="image-20201210203642668" style="zoom: 50%;" />

设置 "东门" 为起点，"学生餐厅"为终点后找到的最短路

<img src="../../../markdown_photo/image-20201210203824497.png" alt="image-20201210203824497" style="zoom: 50%;" />



###### 管理员端

红色的路径表示这条路为手动添加的权值，绿色的路表示这条路为默认权值（欧氏距离）

<img src="../../../markdown_photo/image-20201210203957894.png" alt="image-20201210203957894" style="zoom: 50%;" />

新建一个景点  “在这里！！！！！！！”

<img src="../../../markdown_photo/image-20201211203030652.png" alt="image-20201211203030652" style="zoom:50%;" />

建好以后

<img src="../../../markdown_photo/image-20201211203123501.png" alt="image-20201211203123501" style="zoom:50%;" />

新建一条从“在这里！！！！！！！”到 “路口63” 的默认权值的路

<img src="../../../markdown_photo/image-20201211203250376.png" alt="image-20201211203250376" style="zoom:50%;" />

建好之后

<img src="../../../markdown_photo/image-20201211203324985.png" alt="image-20201211203324985" style="zoom: 80%;" />



## 找路算法（求解单源最短路径：Dijikstra算法）

算法的思想可以用下图来描述

#### 算法介绍

——算法参考自CSDN[（迪杰斯特拉）Dijkstra算法详细讲解](https://blog.csdn.net/qq_37796444/article/details/80663810)

迪杰斯特拉(Dijkstra)算法是典型最短路径算法，用于计算一个节点到其他节点的最短路径。 
	它的主要特点是以起始点为中心向外层层扩展(广度优先搜索思想)，直到扩展到终点为止

##### 基本思想

1. 通过Dijkstra算法计算图G中的最短路径时，需要指定起点startPos

2. 指定三个数组 ： 
   d 数组用来指定当前除了startPos外的点到startPos的最短路的距离
   $$
   d=\{d[i]\;|\;d[i]=the\;shortest\;distance\;between\;startPos\;and\; i\}
   $$
   used数组用来确定第 i 个点的最短路径是否已经被找到
   $$
   used=\{used[i]\;|\;i=1,2,3...,n\}\\used[i]= \begin{cases}
      True &\text{if the shortest path between startPos and i have found}  \\
      False &\text{else}  
   \end{cases}
   $$
   prev数组是每个点的最短路上的前驱节点构成的集合
   $$
   prev[i]=J\quad{}means\ J\ is\ the\ last\ point\ to\ reach\ i
   $$

3. 求解最短路的思想大概是：每次找到1条最短路，然后不断更新d,used,prev数组，来逐步地找到从startPos到每个点 i 的最短路上的所有点，循环的次数为n-1次

##### 操作步骤

一个预声明：

> v( i , j ) 表示的是邻接矩阵上 i -> j 的权值

1. **初始化**

   - 令所有的点（包括startPos）对应的 d[i] 设为 v(startPos,i) , 如果是INF就设为INF（在实际程序中应该是一个很大的值）
   - 所有的点（除了startPos）对应的 used[i] = false ；used[startPos]=True
   - 所有的点（除了startPos）对应的 prev[i] = startPos ； prev[startPos]= -1 

2. **进行循环** 

   每次循环都能找到一条最短路，比如第一次:

   - startPos到每个点（除了startPos）的最短路 d[i] = v(startPos,i),这样一定能找到$\min\;d[i]$,  比如找到的是 d[k]最短 ，那么 used[k]=True ；对于所有的除了 k 和 startPos的点 i ，去进行判断，如果v(k,i)!=INF，且d[i]>d[k]+v(k,i) , 那么d[i]=d[k]+v(k,i) , prev[i]=k

   **一般来说**：第 m 次循环时，找到 min d[i] =d[k] , 然后used[k]=True ；对于所有的没有找到最短路的点 ，去进行判断，如果v(k,i)!=INF，且d[i]>d[k]+v(k,i) , 那么d[i]=d[k]+v(k,i) , prev[i]=k

   **具体看下面的代码**

   ```c++
       for(int i=0;i<graph.point_num-1;i++)
       {
           //找到这次循环所能找到的最短路
           int min=this->INF;
           int u=startPos;
           for(int j=0;j<graph.point_num;j++)
           {
               if( (!used[j]) && (d[j]<min) )
               {
                   u=j;
                   min=d[j];
               }
           }
   
           //表示u的最短路已经找到
           used[u]=true;
   
           //更新d数组和prev数组
           for(int k=0;k<graph.point_num;k++)
           {
               if( (!used[k]) && (graph.arc[u][k].distance!=this->INF) )
               {
                   if(d[u]+graph.arc[u][k].distance<d[k])
                   {
                       d[k]=d[u]+graph.arc[u][k].distance;
                       prev[k]=u;
                   }
               }
           }
   
       }
   ```

3.**寻找从startPos到 endPos 的最短路**

记录endPos，这需要从prev数组进行逆向搜索，搜索prev[endPos]=k，记录k，然后搜索

prev[k]=m，记录m，……，以此类推，直到prev[x]=startPos，记录startPos，然

prev[startPos]=-1 ,搜索结束，接着倒序输出路径

##### Dijkstra算法的证明

下面使用归纳法证明：

1）第一次更新时，一定可以找到一条不能在更短的最短路（注意这里指的是从源点到某个点 k 

的最短路，下文同样），此时更新除了源点以外的所有点的d[i]数组，这时的d[i]的含义时从源点到 i 

至多途径一个点得到的最短路; 

2）假设第 $h$ 次更新时，找到了一条不能在更短的最短路（从源点到  $hh$），更新除了源点以外的

所有点的d[i]数组，这时的d[i]的含义是至多途径 $h+1$个点得到的最短路 ; 假设此时 

$d[k]=\min{}d[i]$ ,  然后从源点到k的最短路为：$源点\to{}k_1\to...\to{}k_m\to{}k$  ,  反设此时最短路上还

能再多加一个点  $i$ ,

使得d($源点\to{}k_1\to...\to{}k_a\to{}i\to{}k_b\to{}k_m\to{}k$) <  d($源点\to{}k_1\to...\to{}k_m\to{}k$) ,

那么会有：
$$
d(源点\to{}k_1\to...\to{}k_a\to{}i\to{}k_b\to{}k_m\to{}k) \\{}\ge{}d(源点\to{}k_1\to...\to{}k_a\to{}i\to{}k_b\to{}k_m)\\{}\to(因为d[k]=\min{}d[i])\ge{}d(源点\to{}k_1\to...\to{}k_a\to{}k_b\to{}k_m\to{}k)
$$
产生矛盾，所以最短路上无法再加点了，从源点到 k 的最短路上所有点都找到了，无法再更新了！



## 经验体会

怎么说呢，这次QT项目制作的过程，是一次道路曲折的过程，在这个过程中，经历了非常非

常多的bug，但好在网上针对大部分的bug都有解决方案，所以最后还是顺利的完成了这个项

目。虽然是一个非常小的项目，但是针对这个项目进行设计的时候还是需要考虑非常多方

面，比如说：界面的布局，功能间互相的联系，还有最短路径算法的学习与构造等等，但是

当把这些东西都啃完之后，还是收益颇丰。同时，在完成这 个项目的过程中，也让我对于使

用结构体构造的图这种特殊的数据结构产生了更深的理解，让我对于经典的找路算法：

dijikstra , bellman-ford , floyd ….之类的算法进行了更好的学习。迪杰斯特拉算法并不是我随

便挑选的，而是针对这个地图项目进行了分析之后，挑选出的最合适的算法（第一，地图上

的最短路一定是单源的；第二，这个地图假设上不可能出现负权路。）基于这个算法，地图

的最短路径现实也变得十分的成功。最后，总结一下我的下一步的心态，那就是，不忘编程

的初心与细心，继续学习更多的知识！