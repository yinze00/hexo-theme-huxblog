---
title: 2020-5-13-AI-Reversi
date: 2020-05-13 17:46:21
tags: AI 
iframe: "https://www.wangjunwei.top/2020/05/13/2020-05-13-AI-Reversi/"
author: "Yinze"
header-img: "reversi.jpeg"
---
<center><font face="黑体" color=black size=6>AI Reversi Report</font> </center>


#### 1. 选型及介绍

​	之前写过五子棋用的是α-β pruning(α-β剪枝) ；这次想试试MCTS(蒙特卡洛树搜索)

##### 1.1 MCTS

​	![MCTS](MCTS.png)

 * **选择**(Selection)

   从根节点R开始，选择连续的子节点向下至叶子节点L。后面给出了一种选择子节点的方法，让游戏树向最优的方向扩展，这是蒙特卡洛树搜索的精要所在.

 * **扩展**(Expansion)

   除非任意一方的输赢使得游戏在L结束，否则创建一个或多个子节点并选取其中一个节点C.

 * **仿真**(Simulation)

   在从节点C开始，用随机策略进行游戏, 这一模拟过程用叫快速走子策略, 迅速地随机地走到底, 得到一个胜负结果. 也称rollout

 * **反馈**(Backpropagation)

   使用随机游戏的结果，更新从*C*到*R*的路径上的节点信息

##### 1.2  UCT的选择方法

​	每个子节点得分：
    ![](sqrt.JPG)
​	x 是节点的当前胜率估计;N 是节点的访问次数。C 是一个常数。C 越大就越偏向于广度搜索，C 越小就越偏向于深度搜索。注意对于原始的 UCT 有一个理论最优的 
    ![](122.JPG)

#### 2. 具体实现

##### 2.1  数据结构设计

​	每一个node节点都是一个board类, 节点之间的关系通过python字典连接:

  * self.children = dict() :

    字典类, `key = node` 一个棋盘对象(`node = Board()`);  `keyvalue = set()` 键值是一个`set()`集合, 包含当前节点的各种走法的下一步棋盘对象

  * self.Q 和 self.N:

    都是`defaultdict()`类的一个对象,  用来记录每一个node(棋盘对象)的访问次数和获胜次数

​	*基本数据结构框架如下:*

```python
reversi = {'O': 'X', 'X': 'O'}

class MCTS:
    "Monte Carlo tree searcher. First rollout the tree then choose a move."

    def __init__(self, exploration_weight=1, color='0'):
        self.Q = defaultdict(int)  # total reward of each node
        self.N = defaultdict(int)  # total visit count for each node
        self.children = dict()     # children of each node
        self.exploration_weight = exploration_weight
        self.color = color

    def find_children(self, node, ccolor='O') -> set():
		pass

    def find_random_child(self, node, color='O'):
		pass

    def is_terminal(self, node) -> bool:
		pass

    def choose(self, node):
        "Choose the best successor of node. (Choose a move in the game)"
		pass

    def do_rollout(self, node):
        "Make the tree one layer better. (Train for one iteration.)"
		pass
    def _select(self, node):
        "Find an unexplored descendent of `node`"
		pass

    def _expand(self, node):
        "Update the `children` dict with the children of `node`"
		pass

    def _simulate(self, node, flag: bool):
        "Returns the reward for a random simulation (to completion) of `node`"
		pass

    def _backpropagate(self, path, reward):
        "Send the reward back up to the ancestors of the leaf"
		pass

    def _uct_select(self, node):
        "Select a child of node, balancing exploration & exploitation"
		pass
```

 * 与main.py的接口部分

   由于只需要调用`.get_move(board)`函数来获得action做出判断. 下面是get_move:

   ```python
   tree = MCTS(1.414, self.color)
   for _ in range(200):
       tree.do_rollout(board)
       actionboard = tree.choose(board)
   
       for idxc in range(8):
           for idxr in range(8):
               if ((actionboard[idxc][idxr] != board[idxc][idxr]) 
                   and (board[idxc][idxr] == '.')):
                   return chr(ord('A') + idxr) + str(idxc + 1)
   ```

##### 2.2  算法设计

 * 选择

   ```python
       def _select(self, node):
           "Find an unexplored descendent of `node`"
           path = []
           while True:
               path.append(node)
               if node not in self.children or not self.children[node]:
                   # node is either unexplored or terminal
                   return path
               unexplored = self.children[node] - self.children.keys()
               if unexplored:
                   n = unexplored.pop()
                   path.append(n)
                   return path
               node = self._uct_select(node)  # descend a layer deeper
       def _uct_select(self, node):
           "Select a child of node, balancing exploration & exploitation"
   
           # All children of node should already be expanded:
           assert all(n in self.children for n in self.children[node])
           log_N_vertex = math.log(self.N[node])
           def uct(n):
               "Upper confidence bound for trees"
               return self.Q[n] / self.N[n] + self.exploration_weight * math.sqrt(
                   log_N_vertex / self.N[n]
               )
   
           return max(self.children[node], key=uct)
   ```

 * 扩展

   ```python
       def _expand(self, node):
           "Update the `children` dict with the children of `node`"
           if node in self.children:
               return  # already expanded
           self.children[node] = self.find_children(node, self.color)
   ```

 * 模拟

   ```python
   	def _simulate(self, node, flag: bool):
           "Returns the reward for a random simulation (to completion) of `node`"
           invert_reward = True  # 代表 self.color = 'O' 在找落子点
           while True:
               if self.is_terminal(node):
                   reward = node.get_winner()[0]  # 黑0白1
                   return reward if flag else 1-reward
               node = self.find_random_child(
                   node, (self.color if invert_reward else reversi[self.color]))
               invert_reward = not invert_reward
   ```

 * 反馈

   ```python
      def _backpropagate(self, path, reward):
           "Send the reward back up to the ancestors of the leaf"
           for node in reversed(path):
               self.N[node] += 1
               self.Q[node] += reward
   ```

 * 输出

   ```python
       def choose(self, node):
           "Choose the best successor of node. (Choose a move in the game)"
           if node not in self.children:
               # return node.find_random_child()
               return self.find_random_child(node, self.color)
   
           def score(n):
               if self.N[n] == 0:
                   return float("-inf")  # avoid unseen moves
               return self.N[n]
   
           return max(self.children[node], key=score)
   ```

* 辅助函数

  ```python
  	def find_children(self, node, ccolor='O') -> set():
          ret = set()
          k = list(node.get_legal_actions(self.color))
          for item in k:
              temp = deepcopy(node)
              temp._move(item, self.color)
              ret.add(temp)
          return ret
  
      def find_random_child(self, node, color='O'):
          directions = list(node.get_legal_actions(color))
          if directions:
              newnode = deepcopy(node)
              newnode._move(random.choice(directions), color)
              return newnode
          else:
              return node
  
      def is_terminal(self, node) -> bool:
          k = node.count('.')
          if k == 0:
              return True
          l1 = list(node.get_legal_actions('X'))
          l2 = list(node.get_legal_actions('O'))
          if not l1 and not l2:
              return True
          return False
  ```

#### 3. 效果及感想

 * 本来想再写一个α-β剪枝来博弈一下,但是时间来不及了

 * 和RandomPlayer的博弈来看, 偶尔会输,基本都是获胜结局,记录如下:
    ![](res.JPG)

* 能看到自己写的AI打败自己真实太有成就感了(fulfillment)!