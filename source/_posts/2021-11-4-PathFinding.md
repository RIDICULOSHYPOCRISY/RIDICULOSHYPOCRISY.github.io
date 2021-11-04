#### 寻路算法

**A***

总成本 = estimatedCost + nodeTotalCost

启发式搜索，预估距离为关键。



```c#
public class Astar
{
   
    public static PriorityQueue closedList,openList;
    private static float NodeCost(Node a,Node b)
    {
        //float vecCost = Math.Abs(a.position.x - b.position.x) + Math.Abs(a.position.z - b.position.z);
        //return vecCost;
        //若用水平竖直距离之和计算预估距离，会导致走大直角的情况比较严重，从而加大误差
        Vector3 vecCost = a.position - b.position;
        return vecCost.magnitude;
    }
    public static ArrayList FindPath(Node start,Node goal)
    {
        openList = new PriorityQueue();
        openList.Push(start);
        start.nodeTotalCost = 0.0f;
        start.estimatedCost = NodeCost(start,goal);
        closedList = new PriorityQueue();
        Node node = null;
        while (openList.Length != 0)
        {
            node = openList.First();
            //已经走到了终点
            if(node.position == goal.position)
            {
                return CalculatePath(node);

            }
            ArrayList neighbours = new ArrayList();
            GridManager.instance.GetNeighbours(node,neighbours);
            for(int i = 0; i < neighbours.Count;i++)
            {
                Node neighbourNode = (Node)neighbours[i];
                if(!closedList.Contains(neighbourNode))
                {
                    float estCost = NodeCost(node,neighbourNode);
                    float totalCost = node.nodeTotalCost + estCost;
                    float neighbourNodeEstCost = NodeCost(neighbourNode,goal);
                    //neighbourNodeEstCost = neighbourNodeEstCost + totalCost;
                    neighbourNode.nodeTotalCost = totalCost;
                    neighbourNode.estimatedCost = totalCost + neighbourNodeEstCost;
                    //记录父节点便于保存路径
                    neighbourNode.parent = node;

                    if(!openList.Contains(neighbourNode))
                    {
                        openList.Push(neighbourNode);
                    }
                }
                

                
            }
            closedList.Push(node);
            openList.Remove(node);

        }
        if(node.position != goal.position)
        {
            Debug.LogError("Goal not found!");
            return null;
        }
        return CalculatePath(node);
    }
    private static ArrayList CalculatePath(Node node)
    {
        ArrayList list = new ArrayList();
        while(node!=null)
        {
            list.Add(node);
            node = node.parent;
        }
        list.Reverse();
        return list;
    }
}
```

```c#
public class GridManager : MonoBehaviour
{
    //只要一个地图即可，可设置为单实例模式
/*
单例适用场景：

1.需要生成唯一序列的环境
2.需要频繁实例化然后销毁的对象。
3.创建对象时耗时过多或者耗资源过多，但又经常用到的对象。 
4.方便资源相互通信的环境
*/
    private static GridManager s_Instance = null;
    //设置静态方法调用单例对象
    public static GridManager instance
    {
        get
        {
            if(s_Instance == null)
            {
                s_Instance = FindObjectOfType(typeof(GridManager)) as GridManager;
                if(s_Instance == null)
                {
                    Debug.Log("No GridManager object!\n");

                }
            }
            return s_Instance;
        }
    }
    //释放单例
    void OnApplicationQuit()
    {
        s_Instance = null;
    }

    public int numOfRows;
    public int numOfColumns;
    public float gridCellSize;
    public bool showGrid = true;
    private GameObject[] obstacleList;
    private Node[,] nodes;
    public enum Obstacle{yesOb,noOb1,noOb2,noOb3,noOb4};
    public Obstacle obstacle;
    public GameObject obstaclePrefab;
    //public List<GameObject> obstacleList = new List<GameObject>();


    //封装origin
    private Vector3 origin = new Vector3();
    
    public Vector3 Origin
    {
        get {return origin;}
    }
    
    void Awake()
    {
        origin = this.transform.position;
        GenerateObstacle();
        //Debug.Log(obstacleList);
        CalculateObstacles();
        //List<GameObject> obstacles = new List<GameObject>();
    }
    public void GenerateObstacle()
    {
        nodes = new Node[numOfColumns,numOfRows];
        //用序号进行编号
        int index = 0;
        for(int i = 0;i < numOfColumns;i++)
        {
            for(int j = 0;j < numOfRows;j++)
            {              
                Node node = new Node(GetGridCellCenter(index));
                nodes[i,j] = node;
                index++;               
            }
        }
        //int m = 0;
        for(int i = 0;i < numOfRows;i++)
        {
            for(int j = 0;j < numOfColumns;j++)
            {
              if((!(i == 0 && j == 0)) && (!(i == numOfColumns - 1 && j == numOfRows - 1)))
              {

              
                obstacle = (Obstacle)Random.Range(0, 5);
                switch (obstacle)
                {
                  case Obstacle.yesOb:
                    Instantiate(obstaclePrefab,nodes[i,j].position,Quaternion.identity);
                    //GameObject obstaclePre = Instantiate(obstaclePrefab,nodes[i,j].position,Quaternion.identity);
                    //obstacleList[m].Add(Instantiate(obstaclePrefab,nodes[i,j].position,Quaternion.identity));
                    //obstacleList[m] = obstaclePre;
                    //m++;
                    break;
                  
                  case Obstacle.noOb1:
                    break;
                  case Obstacle.noOb2:
                    break;
                  case Obstacle.noOb3:
                    break;  
                  case Obstacle.noOb4:
                    break;
                
                }
              }
            }
        }
        obstacleList = GameObject.FindGameObjectsWithTag("Obstacle");
        
    }
    public void CalculateObstacles()
    {
        
        //标记障碍物
        if(obstacleList != null)
        {
            foreach(GameObject data in obstacleList)
            {
                int indexCell = GetGridIndex(data.transform.position);
                int row = GetRow(indexCell);
                int col = GetColumn(indexCell);
                nodes[row,col].MarkAsObstacle();
            }
            //Debug.Log("!!!!");
        }
    }
    //获得编号
    public int GetGridIndex(Vector3 pos)
    {
        pos -= Origin;
        int col = (int)Mathf.Floor((pos.x/gridCellSize));
        int row = (int)Mathf.Floor((pos.z/gridCellSize));
        return (row*numOfColumns + col);

    }
    //网格中心坐标
    public Vector3 GetGridCellCenter(int index)
    {
        Vector3 cellPos = GetGridCellPosition(index);
        cellPos.x += (gridCellSize/2.0f);
        cellPos.z += (gridCellSize/2.0f);
        return cellPos;

    }
    //网格顶点坐标
    public Vector3 GetGridCellPosition(int index)
    {
        int row = GetRow(index);
        int col = GetColumn(index);
        float xPosInGrid = (col)*gridCellSize;
        float zPosInGrid = (row)*gridCellSize;
        return Origin + new Vector3(xPosInGrid,0.0f,zPosInGrid);
    }
    //根据编号获得所在列数
    public int GetColumn(int index)
    {
        int col = index % numOfColumns;
        //if(index % numOfColumns != 0)
            //col = (index % numOfColumns)-1;
        //else col = 9;
        return col;
    }
    //根据编号获得所在行数
    public int GetRow(int index)
    {

        int row = (int)(index/numOfColumns);
        return row;
    }
    // Start is called before the first frame update
    void Start()
    {
        //GenerateObstacle();
    }

    // Update is called once per frame
    void Update()
    {
        
    }
    
    void OnDrawGizmos()
    {
        if(showGrid)
        {
            DebugDrawGrid(transform.position,numOfRows,numOfColumns,gridCellSize,Color.white);
        }
        //在原点生成一个初始参考球体
        Gizmos.DrawSphere(transform.position,0.5f);
    }
    private void DebugDrawGrid(Vector3 origin,int numRows,int numColumns,float cellSize,Color color)
    {
        float width = numColumns * cellSize;
        float height = numRows * cellSize;
        for(int i = 0; i < numRows + 1; i++)
        {
            Vector3 startPos = origin + i*cellSize*new Vector3(0.0f,0.0f,1.0f);
            Vector3 endPos = startPos + width*new Vector3(1.0f,0.0f,0.0f);
            Debug.DrawLine(startPos,endPos,color);
        }
        for(int j = 0; j < numColumns + 1; j++)
        {
            Vector3 startPos = origin + j*cellSize*new Vector3(1.0f,0.0f,0.0f);
            Vector3 endPos = startPos + height*new Vector3(0.0f,0.0f,1.0f);
            Debug.DrawLine(startPos,endPos,color);
        }
    }
    public void GetNeighbours(Node node,ArrayList neighbours)
    {
        Vector3 neighbourPos = node.position;
        int neighbourIndex = GetGridIndex(neighbourPos);
        int row = GetRow(neighbourIndex);
        int column = GetColumn(neighbourIndex);
        //下
        int leftNodeRow = row-1;
        int leftNodeColumn = column;
        AssignNeighbour(leftNodeRow,leftNodeColumn,neighbours);
        //上
        leftNodeRow = row+1;
        leftNodeColumn = column;
        AssignNeighbour(leftNodeRow,leftNodeColumn,neighbours);
        //左
        leftNodeRow = row;
        leftNodeColumn = column-1;
        AssignNeighbour(leftNodeRow,leftNodeColumn,neighbours);
        //右
        leftNodeRow = row;
        leftNodeColumn = column+1;
        AssignNeighbour(leftNodeRow,leftNodeColumn,neighbours);
        
        //左下
        leftNodeRow = row-1;
        leftNodeColumn = column-1;
        AssignNeighbour(leftNodeRow,leftNodeColumn,neighbours);
        //右下
        leftNodeRow = row-1;
        leftNodeColumn = column+1;
        AssignNeighbour(leftNodeRow,leftNodeColumn,neighbours);
        //左上
        leftNodeRow = row+1;
        leftNodeColumn = column-1;
        AssignNeighbour(leftNodeRow,leftNodeColumn,neighbours);
        //右上
        leftNodeRow = row+1;
        leftNodeColumn = column+1;
        AssignNeighbour(leftNodeRow,leftNodeColumn,neighbours);
        

    }
    public void AssignNeighbour(int row,int column,ArrayList neighbours)
    {
        if(row != -1 && column != -1 && row < numOfRows && column < numOfColumns)
        {
            Node nodeToAdd = nodes[row,column];
            //如果为障碍物则不放入此数组
            if(!nodeToAdd.isObstacle)
            {
                neighbours.Add(nodeToAdd);
            }
        }

    }
}
```

```c#
public class TestCode : MonoBehaviour
{
    private Vector3 startPos,endPos;
    GameObject startPoint,endPoint;
    public Node startNode {get; set; }
    public Node goalNode {get; set; }
    public ArrayList pathArray;

    public float intervalTime = 1.0f;
    public float elapsedTime = 0.0f;
    // Start is called before the first frame update
    void Start()
    {
        startPoint = GameObject.FindGameObjectWithTag("Start");
        endPoint = GameObject.FindGameObjectWithTag("End");
        FindPath();
    }
    private void FindPath()
    {
        startPos = startPoint.transform.position;
        endPos = endPoint.transform.position;
        //Debug.Log(startPos);
        //根据位置坐标获取开始和目标节点
        startNode = new Node(GridManager.instance.GetGridCellCenter(GridManager.instance.GetGridIndex(startPos)));
        goalNode = new Node(GridManager.instance.GetGridCellCenter(GridManager.instance.GetGridIndex(endPos)));
        //Debug.Log(GridManager.instance.GetGridCellCenter(GridManager.instance.GetGridIndex(startPos)));
        /*
        if(Input.GetKey("k"))
        {
            pathArray = Astar.FindPath(startNode,goalNode);
            Debug.Log("Astar");
        }
        if(Input.GetKey("l"))
        {
            pathArray = Dijkstra.FindPath(startNode,goalNode);
            Debug.Log("Dijkstra");
        }
        */
        pathArray = Astar.FindPath(startNode,goalNode);
        
        //pathArray = Dijkstra.FindPath(startNode,goalNode);
        
    }

    // Update is called once per frame
    void Update()
    {
        //一秒执行一次
        elapsedTime += Time.deltaTime;
        if(elapsedTime >= intervalTime)
        {
            elapsedTime = 0.0f;
            FindPath();
        }
    }
    void OnDrawGizmos()
    {
        if(pathArray == null) return;
        if(pathArray.Count>0)
        {
            int index = 1;
            //Debug.Log("Succeed!");
            foreach(Node node in pathArray)
            {
                if(index < pathArray.Count)
                {
                    Node nextNode = (Node)pathArray[index];
                    Debug.DrawLine(node.position,nextNode.position,Color.black);
                    index++;
                    Debug.Log(index);
                }
            }
        }
    }
}

```



**Dijkstra**

Dijkstra与A*相比，只是少了预估的成本那一项，A *试图通过使用启发式函数来寻找一条更好的路径，该函数优先考虑应该被认为比其他节点更好的节点，而Dijkstra只是探索所有可能的路径。

