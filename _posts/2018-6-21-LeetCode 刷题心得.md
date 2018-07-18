---
layout: post
author: syea
title:  LeetCode 刷题心得
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - Swift
    - Algorithm
---

# LeetCode 刷题心得

不知不觉刷 LeetCode 快一个月了，不知道为什么居然坚持了下来，而且还越做越有兴趣。好像回到了读书的时候，就喜欢做难题，做出来成就满满。<br>

其实去年就接触过 LeetCode , 师傅推荐，朋友也有做算法的，只是一进去就被几道动态规划题劝退了。尴尬..<br>

ok，总结一下心得。<br>

#### 1.参数判断
基本上我要写解的时候，会直接写上`guard else`.也算是一种经验了，往往题目的测试用例中会有很多特殊情况，空值，nil等等。其实平常在我们写代码时，也要注意对方法中的参数做判断，增强代码的健壮性。

#### 2.借助额外空间
这个也比较常见，比如
>
给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。<br><br>
你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。<br><br>
示例:<br><br>
给定 nums = [2, 7, 11, 15], target = 9<br><br>
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
>

我们可能会马上想到遍历数组，数组一大，妥妥超时。其实我们可以借助一个`Map`,当取到数组的一个值时，将差值也存入`Map`，这样当下次遍历到该差值时就可以直接输出结果。

```
func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    guard nums.count > 1 else {
        return nums
    }
    var map = [Int : Int]()
    for i in 0..<nums.count {
        let n = nums[i]
        if map[n] == nil {
            map[n] = 0
            map[target - n] = 1
        }else if map[n] == 1{
            return [nums.index(of: target - n)!,i]
        }
    }
    return []
}
```
其他还可以借助 Array、Set...可能写算法时还会有空间要求，但是平常写代码时，相信一点点空间的浪费来换取高性能的方法执行，还是非常好的。

**Tips**<br>
1. 当输入数据有限制时，比如只有小写字母，只有0-9数字等等，可以考虑用包含所有这些情况的数组。
2. 当输入数据没有限制时，通常可以用 map 将数据存储起来。

#### 3.尽量想全边界条件
很多时候，题目给你的例子都比较正常，便于你理解，但往往也有很多特殊情况存在，求解时一定要想清楚。
举个例子：
>
给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。<br>
有效字符串需满足：<br>
左括号必须用相同类型的右括号闭合。<br>
左括号必须以正确的顺序闭合。<br>
注意空字符串可被认为是有效字符串。<br><br>
示例 1:<br>
输入: "()"<br>
输出: true<br><br>
示例 2:<br>
输入: "()[]{}"<br>
输出: true<br><br>
示例 3:<br>
输入: "(]"<br>
输出: false<br><br>
示例 4:<br>
输入: "([)]"<br>
输出: false<br><br>
示例 5:<br>
输入: "{[]}"<br>
输出: true
>

我一开始的思路是准备一个 Map
```
    let map : [Character : Int] = ["(":0,")":0,"[":0,"]":0,"{":0,"}":0]
```
然后遍历字符串，遇到左括号设值 +1，右括号设值-1.最后看相加是不是都等于0.<br>
这显然是审题不清，没仔细看第4个示例。<br>

然后我想到用栈结构
```
func isValid(_ s: String) -> Bool {
    guard s.count % 2 == 0 else {
        return false
    }
    let map : [Character : Character] = ["(":")","[":"]","{":"}"]
    var stack = [Character]()
    for c in s {
        if c == "(" || c == "{" || c == "[" {
            stack.append(c)
        }else{
            if stack.last == nil || c != map[stack.last!]  {
                return false
            }else{
                stack.removeLast()
            }
        }
    }
    return true
}
```
做完我真的觉得100%正确，但是我还是忽略了全是左括号这种情况。最后应该
```
    return stack.count == 0
```



#### 小结
目前只想到这么点，以后有更深的体会了再来补充。