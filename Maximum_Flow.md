#  Network Flow 網絡流: Find Maximum Flow 尋找最大流
## 什麼是 Maximum Flow 問題?
  <div>
    舉一個Maximum Flow經典的案例，如果有兩個大城鎮A與B，要從A移動到B城鎮，會經過數個小城鎮、並且有數條路可以通行，
    這些路是開車的通勤族每天上下班的交通要道。如果交通部長想要知道兩城鎮之間最大可以乘載多少車流量，以確認是否可以承受尖峰時的需求，
    他該怎麼計算呢？這時候，我們就可以攤開地圖，先把從A、B城鎮以及兩城之間的小城鎮畫出來，接著把可以通行的點與點用線串起來，
    最後標上每條路一個時間內可以乘載的最大車流量也標上去，就完成要開始解Maximum Flow問題的起手式了！
  </div>
  <div align="right">
    <img alt=traffic src='https://user-images.githubusercontent.com/62165222/196332143-8f57c677-eb45-4087-ab13-4ac6bb327a50.png' width="30%"/>
  </div>



### 在開始之前，先了解一下graph的一些名詞解釋
<div align='center'>
<img height='250px' alt='exlain1' src='https://user-images.githubusercontent.com/62165222/196347197-38baa752-d889-4539-bece-c97fabb2f11a.png'/>
<img height='250px' alt='exlain2' src='https://user-images.githubusercontent.com/62165222/196347239-0bde2037-b8fd-4a96-a408-5a81167a5d8e.png'/>
</div>

  - s 起點 (source): 以上面的例子來說就是城鎮A
  - t 終點 (sink): 以上面的例子來說就是城鎮B
  - capacity 容量: 每條路的最大車流
  - residual 剩餘容量 = 路還可以乘載多少車流
  - augmenting path： 簡單路徑，不能有回圈
  - bottleneck capacity 瓶頸容量：一個簡單路徑中管徑大小最小的邊
  - saturated 飽和：當某個邊剩餘容量已經變為零
  - block flow 阻塞流：當已沒有流可以從s到t，此時所有流向終點、或者流出起點的流量總和（兩者會相等）即為阻塞流，maximum flow也是其中一種
    
### 簡單解的做法
  1. 隨意找一條能通道終點的路徑
  2. 此路徑能流通的的量為bottleneck邊的量，將路徑的每個邊減去這個量
  3. 若此時有邊已經飽和(值 = 0)則將這個邊拿掉；
  4. 反覆進行，直到沒有路徑可以通往終點
  5. 將原圖各邊的量減去此時還剩下的邊的量：`FLow = Capacity - Residual`
  6. 得出的結果即為阻塞流(或稱最大流)，其流量等於所有從起點向外流出的量、或也可以看流進終點的量。阻塞流的意義為這個流動的方式(前面的步驟)，已經把所有路徑都堵住無法再增加流量了。
  
### 用簡單解找出的不一定是最大流
  舉下面的例子來說，如果在選路徑時，先選中紫色的，則會得到一個阻塞流，但其實他並非最大流
  |![image](https://user-images.githubusercontent.com/62165222/196371747-bf77bf5e-271d-472d-aca8-d7da9dc4f2db.png)|![image](https://user-images.githubusercontent.com/62165222/196372515-3a71558e-baa1-4b2f-bba1-c9b0eca87c84.png)|
  |---|---|
  |最大流|非最大流|

******************************
## Maximum Flow的經典解法
### 1. Ford-Fulkerson & Edmonds-Karp Algorithm
  Ford-Fulkerson的概念基本上和簡單解相同，但最大的差別是允許「反悔」。他在每一輪完成後，在此次路徑經過的邊上，新增一個流量與正向相同的反向邊，如此一來便可以修正先前非最佳的路徑選擇。</br>
  舉例來說，思考在這張圖中，如果沒有添加反向的邊，那麼就沒有路可以走了，將會終止。但當添加了反向的邊之後，就能夠再找到其他的路徑。</br>
  <left><img height='250px' alt='Ford-Fulkerson' src='https://user-images.githubusercontent.com/62165222/196353462-0e738903-11c9-4f9a-a73e-f21088245230.png'/></left>

  - Ford-Fulkerson解法的步驟如下：
    1. 隨意找一條能通道終點的路徑
    2. 此路徑能流通的的量為bottleneck邊的量，將路徑的每個邊減去這個量
    3. 若此時有邊已經飽和(=0)則將邊拿掉；
    4. 新增一個反向邊帶有方才流經的量
    5. 若有方向相同的多條路徑則進行合併(merge edges)
    6. 反覆進行，直到沒有路徑可以通往終點
    7. 將反向邊都刪掉
    8. 將原圖各邊的量減去此時還剩下的邊的量 FLow = Capacity - Residual
    9. 得出阻塞流/最大流

  - 時間複雜度：`O(f * E)` f:最大流大小, E:邊數
    Ford-Fulkerson雖然能找到最佳解，但其效率很差，而且如果遇到例如像下面這樣的worst case情況，就會因為反向邊不斷循環而增加不必要的運算，像是下面就會以每次只有1的流量的速度來回跑。</br>
    <img height='250px' alt='Ford-Fulkerson worst case' src='https://user-images.githubusercontent.com/62165222/196353865-5507883b-5759-4017-88e1-978c246ef897.png'/>

  - 優化版的Edmonds-Karp Algorithm：複雜度：`O(VE^2^)`
    Edmonds-Karp Algorithm與Ford-Fulkerson Algorithm唯一的差別，只有在一開始決定要走哪條路徑時，Edmonds-Karp規定一定要選擇最短路徑。因此實作上在執行時會先跑一個DFS

### 2. Dinic’s
  - level graph 定義：將節點分層，每層之間都必須是走一步就能到達的點，level graph只能按順序向下走，不可以回頭，同階層的點也不能通行。例如下圖的階層1的兩個點之間如果本來有連通，則要將其去除。</br>
    <img height='250px' alt='level graph' src='https://user-images.githubusercontent.com/62165222/196376116-cb5dadfe-38ae-4991-869b-edb95ea535a0.png'/>

  - Dinic’s解法步驟如下：
    1. 每輪開始前，都要先製作level Graph
    2. 在level graph上找到阻塞流（最佳解的所有步驟）。
    3. 將阻塞流的各邊流量，減去原圖的量，並更新圖。
    4. 在圖上放上阻塞流的路徑的反向邊。
    5. 反覆進行上述步驟，直到level graph已經找不到阻塞流
    7. 將留下的residual圖此去原圖，得到最大流
  - 時間複雜度：`O(E^2^)`

## Minimun Cut Problem 最小割問題
   常常與上面的maximum flow一起被討論的另一個問題是「最小割問題」，這個問題是想要嘗試找出一個，能切斷所有從起點到終點的可能路徑的cut。</br>
    <img height='250px' alt='min-cut' src='https://user-images.githubusercontent.com/62165222/196377134-0a616bb2-a9f7-4b95-9150-094d14e0491b.png'/>
  - 定義：
    - S-T Cut定義: 把圖切成S子集合(包含s)和T子集合(包和t)兩部份，只要切這一刀即所有管道就都不通了
    - S-T Cut的容量: 所有從S通往T子集合得流量
    - Minimum S-T Cut 最小割：S-T Cut的容量最小
    - 基本上，最大流的流量=最小割的流量，所以我們能透過上面的方法來尋找

  - 找出最小割步驟：
    1. 用Edmonds-Karp或Dini’c最大流算法得到residual graph，將反向邊刪除
    2. 將residual graph中所有s能到達的邊圈成S集合、剩下的則為T集合
