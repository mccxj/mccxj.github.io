---
layout: post
comments: true
title: "马踏棋盘的算法思考"
categories: ["算法", "棋盘", "回溯法", "贪心法"]
---

### 马踏棋盘
将马随机放在国际象棋的8×8棋盘的某个方格中，马按走棋规则进行移动。
要求每个方格只进入一次，走遍棋盘上全部64个方格。

### 又是回溯?
这个做法和前面几个问题是差不多的([数独(sudoku)游戏][1])，所以这里就不做太多解释了。
总体来说，就是可选的解空间是8个方向。回溯的代码就不多提了，需要说的是，这里简单的回溯
效率不好，需要跑很久才出结果。我们有什么方式可以优化一下!

### 贪心是否有效?
首先，我们可以对可供选择的方向进行过滤，对于它相连的8个方向所在的点，
在当前的局面下，假如存在两个点只有一条路可以到达，那么可以直接回溯，
因为无论走哪个方向都不能走完整个棋盘。假如存在一个点，它只有一条路可以到达，
那么下一步必须走这个方向。

但是，即使我们过滤了一些路径，效率还是没有非常大的提升。我们需要考虑一种贪心的思路：
在可选的方向中，优先选择出口最少的目标点。因为优先选择出口少的点,
就可以让出口少的点尽量少，使得最后剩下的点有比较多的入口，这样最终达到目的的概率就会大些。

虽然这个思路，我觉得不好直接证明，不过我修改了一些代码实现，的确效率得到非常大的提升。
就我那个机器，ruby代码，不到1s，64个点的情况就全部处理完了。

### 代码示例
{% highlight ruby %}
def init
  @arr = Array.new(8)
  (0..7).each do |i| @arr[i] = Array.new(8, 0) end
  @path = Array.new(64) 
  @ok = false
  @poss = [[-2, 1], [2, 1], [1, 2], [-1, 2], [2, -1], [-2, -1], [-1, -2], [1, -2]]
end
# 判断[x,y]是否在棋盘内并未走过
def valid?(x,y)
  x>=0 && x<8 && y>=0 && y<8 && @arr[x][y]==0
end

# 判断当前点有哪些方向可以走
def filter(mark)
  posz, mustpos = {}, nil
  a, b = @path[mark]
  @poss.each do |pos|
    axp, bxp = a+pos[0], b+pos[1]
    if valid?(axp, bxp)
      sz = 0
      @poss.each do |ps|
        ap, bp = axp+ps[0], bxp+ps[1]
        sz+=1 if valid?(ap, bp) # 存在空点
      end
      if sz>0
        posz[pos] = sz
      else
        mustpos.nil? ? (mustpos = pos) : (return [])
      end
    end
  end
  # 对目标点的方向多寡进行排序
  mustpos.nil? ? posz.keys.sort {|a,b| posz[a]<=>posz[b]} : [mustpos]
end

def walk(mark)
  if mark==63
    @ok = true
    return
  end
  a, b = @path[mark]
  filter(mark).each do |pos|
    rp = [a+pos[0], b+pos[1]]
    @path[mark+1] = rp
    @arr[rp[0]][rp[1]] = 1
    walk(mark+1)
    @ok ? return : (@arr[rp[0]][rp[1]] = 0)
  end
end

(0..7).each do |i|
  (0..7).each do |j|
    init
    @path[0] = [i,j]
    @arr[i][j] = 1
    walk(0)
    if @ok
      @path.each_with_index do |ps, idx|
        @arr[ps[0]][ps[1]] = idx
      end
      @arr.each do |ar| p ar end
      puts
    else
      "no solution"
    end
  end
end
{% endhighlight %}

 [1]: 20120710_sudoku.html
