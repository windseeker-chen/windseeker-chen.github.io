# 方法一：双指针滑动窗口
对于下标为[left,right]范围内可接的雨水为(right-left-1)*min(height[left], height[right]) - sum(height[left]+1...height[right]-1)

所以需要找到保证[left,right]中left,right左右方向的最大值

指针left标识雨水范围的左边界，right标识雨水范围的右边界

1. 如果height[left+1] > height[left]，需要left右移
2. 当left确定时，right初始化为left+1，遍历剩余下标找出最大值maxRight，并计算[left,maxRight]范围内接住的雨水，并将left置为maxRight下标
3. 循环直到left遍历结束

- 时间复杂度：O(n)
  1. 主循环：在 trap 函数中，通过 left 指针遍历整个数组。每次遍历从当前的 left 找到一个有效的 right。最内层的 while 循环在每次找到一个 right 后更新 left 到 maxRight。
  2. right 的推进：在寻找 right 的过程中，需要线性扫描剩余的数组部分。然而每次 right 都至少会增加，而在找到一个“容器”后，left 直接跳到 maxRight，从而避免重复检查已经计算过的部分。 因此，每个元素最多被走两遍（一次作为 left，一次在寻找 right 的过程中）。
因此，整个算法对于数组中的每个元素只会处理有限的次数，时间复杂度为 O(n)。
- 空间复杂度：O(1)
只使用了固定变量，所以时O(1) 

```golang
func trap(height []int) int {
	left, size := 0, 0
	for left < len(height)-1 {
		if height[left] == 0 || height[left+1] > height[left] {
			left++
			continue
		}
		right := left + 1
		maxRight := right
		for right < len(height) {
			if height[right] > height[left] {
				maxRight = right
				break
			}
			if height[right] >= height[maxRight] {
				maxRight = right
			}
			right++
		}
		size += calc(height, left, maxRight)
		left = maxRight

	}

	return size
}

func calc(height []int, left, right int) int {
	sum := 0
	for i := left + 1; i < right; i++ {
		sum += height[i]
	}
	return (right-left-1)*min(height[left], height[right]) - sum
}
```

# 方法二：动态规划
方法一虽然空间复杂度为O(1)，但是rigth遍历获得maxRight的过程中重复遍历了多轮[maxRight,len(height)-1]的数据，所以考虑先遍历计算出右侧最大值的位置。
1. 逆序遍历，记录下标i时右侧最大值的下标，获得rightMaxM
2. 顺序遍历left
   1. 顺序遍历[left+1,rightMaxM[left]-1]，下标为right
      1. 如果存在height[right] > height[right]，找到一个区间[left,right]，计算雨水并将left置为right
      2. 如果不存在height[right] > height[right]，获得一个区间[left,rightMaxM[left]]，计算雨水并将left置为rightMaxM[left]

- 时间复杂度：O(n)
  1. 在计算rightMaxM时遍历整个数组
  2. 在 trap 函数中，通过 left 指针遍历整个数组。每次遍历从当前的 left 找到一个有效的 right。内层的right循环在每次找到一个 right 后更新 left 到 right。在寻找 right 的过程中，遍历[left+1,rightMaxM[left]-1]，遍历后left 直接跳到 right，从而避免重复检查已经计算过的部分，因此每个元素只会走一遍。
  3. calc函数计算时会遍历[left,right]，最坏情况下数组中每个雨水区域都是相邻的，每个元素都会走一遍。
因此，整个算法对于数组中的每个元素最多会被处理3次，时间复杂度为 O(3n)。
- 空间复杂度：O(n)
rightMaxM存储长度和数组一致，所以是O(n)

```golang
func trap(height []int) int {
	rightMaxM, maxRightIndex := make(map[int]int, len(height)), len(height)-1
	for i := len(height) - 1; i >= 0; i-- {
		rightMaxM[i] = maxRightIndex
		if height[i] >= height[maxRightIndex] {
			maxRightIndex = i
		}
	}

	left, size := 0, 0
	for left < len(height)-1 {
		if height[left+1] > height[left] {
			left++
			continue
		}
		gotLeft := false
		for right := left + 1; right < rightMaxM[left]; right++ {
			if height[right] > height[left] {
				size += calc(height, left, right)
				left = right
				gotLeft = true
				break
			}
		}
		if !gotLeft {
			right := rightMaxM[left]
			size += calc(height, left, right)
			left = right	
		}
	}

	return size
}

func calc(height []int, left, right int) int {
	sum := 0
	for i := left + 1; i < right; i++ {
		sum += height[i]
	}
	res := (right-left-1)*min(height[left], height[right]) - sum
	fmt.Println(left, right, res)
	return res
}
```

# 方法三： 动态规划2
上面的两中方法的思路是先找到区间，再计算区间中的雨水。我们可以从区间的构成发现，在下标i处，雨水量为左右方向上的最小值减去height[i]。假设左侧最大值为leftMaxM[i]，右侧最大值为rightMax[i]，则雨水为min(leftMaxM[i],rightMaxM[i])-height[i]
1. 顺序遍历，记录下标i时左侧最大值的下标，获得leftMaxM
2. 逆序遍历，记录下标i时右侧最大值的下标，获得rightMaxM
3. 顺序遍历，计算下标i处的雨水

- 时间复杂度：O(n)
  1. 在计算leftMaxM时遍历整个数组
  2. 在计算rightMaxM时遍历整个数组
  3. 顺序遍历计算雨水时遍历整个数组
因此，整个算法对于数组中的每个元素最多会被处理3次，时间复杂度为 O(3n)。
- 空间复杂度：O(n)
leftMaxM，rightMaxM存错长度和数组一致，所以是O(2n)

```golang
func trap(height []int) int {
	//正序遍历，记录当前下标左侧最大值
	leftMax := make([]int, len(height))
	maxLeft := 0
	for idx, h := range height {
		leftMax[idx] = maxLeft
		if h > maxLeft {
			maxLeft = h
		}
	}
    //反序遍历，记录当前下标右侧最大值
	rightMax := make([]int, len(height))
	maxRight := 0
	for r := len(height) - 1; r >= 0; r-- {
		rightMax[r] = maxRight
		if height[r] > maxRight {
			maxRight = height[r]
		}
	}
	//遍历，计算当前下标存错雨量 min(leftMax,rightMax)-height
	sum := 0
	for idx, h := range height {
		if h >= leftMax[idx] || h >= rightMax[idx] {
			continue
		}
		got := min(leftMax[idx], rightMax[idx]) - h
		sum += got
	}
	return sum
}
```

# 方法四：单调栈（看答案的）
方法一和方法二采用遍历的方式来寻找雨水区间的右边界，可以使用单调栈来简化查找操作。
每个高度都是一堵墙，用栈保存，当遍历墙的高度的时候，如果当前高度小于栈顶(当前左侧)的墙高度，说明这里会有积水，我们将墙的高度的下标入栈。

如果当前高度大于栈顶的墙的高度，说明这是之前的积水右边界，，我们可以计算下有多少积水了。计算完，就把当前的墙继续入栈，作为新的积水的墙。

1. 初始化栈，存储元素下标
2. 顺序遍历，当前下标为i
   1. 当栈长度大于等于2时，记栈顶元素为 floor，floor 的下面一个元素是 left (height[left] >= height[floor] )
      1. 出栈floor
      2. 如果height[i] > height[floor],意味着存在一个凹区域可以接水，宽度为i-left-1,高度为min(height[i],height[left])-height[floor]
      3. 循环直到栈空或height[i] <= height[floor]
   2. i入栈	
- 时间复杂度：O(n)
  1. 每个元素会入栈和出栈一次
因此，整个算法对于数组中的每个元素最多会被处理1次，时间复杂度为 O(n)。
- 空间复杂度：O(n)
栈长度最坏和数组一致，空间复杂度为o(n)

```golang
func trap(height []int) int {
	sum := 0
	var stack []int
	for i, h := range height {
		for len(stack) > 0 && h > height[stack[len(stack)-1]] {
			floor := stack[len(stack)-1]
			stack = stack[:len(stack)-1]
			if len(stack) == 0 {
				break
			}
			left := stack[len(stack)-1]
			currentHeight := min(height[left], h) - height[floor]
			currentWidth := i - left - 1
			sum += currentHeight * currentWidth
		}
		stack = append(stack, i)
	}
	return sum
}
```

# 方法五：双指针（看答案）
在方法三中，我们需要维护两个数组来存储i下标左右的最大值，可以使用2个指针和2个变量替代。
1. 初始化定义left为0,leftMax为0,right为len(height)-1,rightMax为0
2. 循环 left < right 
   1. 更新leftMax=max(leftMax,height[left])
   2. 更新rightMax=max(rightMax,height[right])
   3. 如果leftMax > rightMax,right处水量为rightMax-height[right],right--
   4. 如果leftMax <= rightMax,left处水量为leftMax-height[left],left++
- 时间复杂度：O(n)
  1. 每个元素只会被操作一次
因此，整个算法对于数组中的每个元素最多会被处理1次，时间复杂度为 O(n)。
- 空间复杂度：O(1)
只维护了2个指针和2个变量，空间复杂度为o(1)
```golang
func trap(height []int) int {
	sum := 0
	leftMax := 0
	rightMax := 0
	left, right := 0, len(height)-1
	for left < right {
		leftMax = max(leftMax, height[left])
		rightMax = max(rightMax, height[right])
		if leftMax > rightMax {
			sum += rightMax - height[right]
			right--
		} else {
			sum += leftMax - height[left]
			left++
		}
	}
	return sum
}
```

