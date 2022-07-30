# Leetcode

## 347. 前 K 个高频元素

给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。 

**步骤 :**

**绕不开** : 遍历一次数组, 将元素与次数一一对应成map存储 ;

**原想法 :** 将map转为entrySet, 转为List\<Entry>, 用stream的sorted方法, 按每个value进行排序, 最后取前k个元素的key组成数组返回 ;

**颠覆想法 :** 维护一个大小为k的堆, 每次压入前和堆顶元素 (设置为最小权重) 比较，如果比堆顶元素还小，直接扔掉，否则压入堆。检查堆大小是否超过 k，如果超过，弹出堆顶。小根堆为PriorityQueue类

```java
		// 定义堆权重规则, 最前的父元素为堆里权重最小		
		PriorityQueue<Integer> pq = new PriorityQueue<>(new Comparator<Integer>() {
            @Override
            public int compare(Integer a, Integer b) {
                return map.get(a) - map.get(b);
            }
        });
        for (Integer key : map.keySet()) {
            if (pq.size() < k) { // 堆内个数不足k, 直接添加
                pq.add(key);
            } else if (map.get(key) > map.get(pq.peek())) {
                // 堆内已满, 获取peek最顶第一个元素 与 当前值作比较
                // 小于 ? 踢出局, 放入新的这个
                pq.remove();
                pq.add(key);
            }
        }
		// 取出最小堆中的元素
        List<Integer> res = new ArrayList<>();
        while (!pq.isEmpty()) {
            res.add(pq.remove());
        }
        return res;
```







## xxx. 数组两数相加 I

Given an array of integers `nums` and an integer `target`, return *indices of the two numbers such that they add up to target*.  返回数组中两个元素相加等于target的下标

```
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Explanation: Because nums[0] + nums[1] == 9, we return [0, 1].
```



**原想法** :  两层遍历, 外层逐个i遍历, 内层从i+1开始遍历, 看是否存在target - i的数, 存在直接break即可

**颠覆想法** : 

- 仅需一层遍历 !!!
- 每到一个将它达到target的差值存到map中, key为diff, value为下标, 
- 遍历到的每个元素, 先看map.containsKey(i), 若存在, 直接返回 value 与 当前下标 即可 !



## 167. 数组两数相加 II - 升序数组

**题目 :** 给你一个下标从 **1** 开始的整数数组 `numbers` ，该数组已按 **非递减顺序排列**  ，请你从数组中找出满足相加之和等于目标数 `target` 的两个数。 

**解析 :** 与 I 的区别在于**数组为升序**, 可设两个 int 的指针start / end

- 从最初两头两个数相加, sum大于target则右指针左移 ( 左指针右移结果只会更大 ) ;
- sum小于target则左指针右移 ;
- while sum != target 则一直不停



## 121. 买卖股票的最佳时机

给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

**原想法：**

```Java
class Solution {
    public int maxProfit(int[] prices) {
        
        int maxPro = 0, 
        minPrice = Integer.MAX_VALUE;
        
        for (int i = 0; i<prices.length;i++) {
            // 每个元素都看其是否可能为最小值
            minPrice = Math.min(minPrice, prices[i]);
            // 每个点都尝试获取其与最小值的差值（差价）
            maxPro = Math.max(prices[i] - minPrice, maxPro);
        }
        
        return maxPro;
    }
}
```



**颠覆想法：**

设置左（买）右（卖）指针，

```java
class Solution {
    public int maxProfit(int[] prices) {
        
        int l=0, r=1, maxP = 0;
        
        while (r < prices.length) {
            
            if (prices[l] < prices[r]) {
                // 满足低买高卖
                maxP = Math.max(maxP, prices[r]-prices[l]);
                r++;
            } else {
                // 不满足低买高卖
                l = r;
                r += 1;
            }
            
        }
        
        return maxP;
    }
}
```



## 128. 最长连续序列

给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 O(n) 的算法解决此问题

示例 1：

输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。



**原想法：**

- 先给数组排序（已经违背时间复杂度问题），逐个遍历；
- 若与前一个元素差值为1，增加长度1；
- 若与前一个元素差值不为1，则中断长度计算，同时Math.max(longest, length)；

```JAVA
class Solution {
    public int longestConsecutive(int[] nums) {
        // empty array return 0
        if (nums.length==0) return 0;
        
        Arrays.sort(nums);
        
        int len = 1, maxLen = 0, last = nums[0];
        
        for (int i=1;i<nums.length;i++) {
            
            // current equals last, continue
            if (nums[i] == last) continue;
            
            if ((nums[i]-last)==1) {
                len++;
            } else {
                maxLen = Math.max(len, maxLen);
                len = 1;
            }
            last = nums[i];
        }
        maxLen = Math.max(len, maxLen);
        
        return maxLen;
        
    }
}
```



**颠覆想法：**

- 用 数组 生成去重的set结构，遍历数组，若nums[i]-1不存在于set中，则可将其视为起点；
- 在起点以后，不断叠加1，直到set中不存在该值为止；
- 叠加过程中不断更新长度
- **总结**：想法很美好，但Java创建Set的代价太大了...

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        // 将数组元素放入Set中
        Set<Integer> set = new HashSet<>(nums.length);
        for (int n : nums) {
            set.add(n);
        }

        int longest = 0;

        for (int n : nums) {
            if (!set.contains(n-1)) {
                // n could be used as start of a sequence
                int len = 0;
                while (set.contains(n + len)) {
                    len += 1;
                }
                longest = Math.max(longest, len);
            }
        }

        return longest;
    }
}
```



# 牛客网 

## HJ6 质数因子 

**功能 :** 输入一个正整数，按照从小到大的顺序输出它的所有质因子（重复的也要列举）（如180的质因子为2 2 3 3 5 ） 

1.获取该数n；

2.获取该数开方后的整数 `long k = (long) Math.sqrt(num);`

3.遍历2-k，能整除n的数i，检查是否为素数，是则添加；

4.将n赋值为 n/i，循环检查到n不能被i整除为止。

## HJ28 素数伴侣 

若两个正整数的和为素数，则这两个正整数称之为“素数伴侣”，如2和5、6和13，它们能应用于通信加密。现在密码学会请你设计一个程序，从已有的 N （ N 为偶数）个正整数中挑选出若干对组成“素数伴侣” 。

1.将输入的数字分为listOdds、listEvens；

2.建立数组，存储对应下标下 偶数所配对的奇数：int[] matchEvens = new int[listEvens.size()];

3.遍历listOdds，每个奇数都有一个自己的boolean[] isMatched = new boolean[listEvens.size()]，以记录当前奇数下，对应的偶数是否已经配对过;

4.每个遍历到的奇数，传入如下方法：` boolean find(int each,List<Integer> listEvens,int[] matchEvens,boolean[] isMatched) `；若方法返回true，则计数器+1；

5.方法内部，遍历偶数，每个偶数都与传入的奇数进行相加，判断 其和是否为素数 以及 该位置偶数是否已配对；

6.若 和为素数 且 该位置偶数未配对，则isMatched转为true；

7.继续判断：若该位置所配对奇数 == 0 （仍为默认） 或者  将 该位置已配对奇数 再次传入find方法也为true，则原奇数让出该位置，赋给新来的奇数each：

```java
if (matchEvens[k]==0 || 
    find(matchEvens[k],listEvens,matchEvens,isMatched)) {
                    matchEvens[k] = each;
                    return true;
}
```



## HJ93 数组分组 

**要求 :** 输入int型数组，询问该数组能否分成两组，使得两组中各元素加起来的和相等，并且，所有5的倍数必须在其中一个组中，所有3的倍数在另一个组中（不包括5的倍数），不是5的倍数也不是3的倍数能放在任意一组，可以将数组分为空数组，能满足以上条件，输出true；不满足时输出false。 

判断条件一 : 若所有数相加无法被2整除，则必然为false ;

判断条件二 : (sum5+sum3+sum)/2 - sum3 （或sum5） 设为 target ;

判断条件三 : 在剩下的int[ ]中，找到 target - x == 0 的组合，x可为其中任意元素的和 ; 

如何实现上一步？利用深度优先算法

```java
//从start==0开始遍历剩余数字list，经过每一位都可以选择‘要与不要’
public static boolean getTarget(List<Integer> list, int target, int start) {
        if (list.size()==start) {
            return target == 0;
        } else  {
            return getTarget(list,target-list.get(start),start+1)||   //要这一位
                   getTarget(list,target,start+1) ;  //不要这一位，直接跳到下一位
       }
}
```



## HJ95 人民币转换 

```java
public static String[] ten = {"零","壹","贰","叁","肆","伍","陆","柒","捌","玖"};
public static String[] power = {"","万","亿"};
public static int pow = 0;
```

**1.每次取 末尾4位 作为处理单位，每取一次，pow++，则 power[ pow ]则为需要追加的数量级 ;**

- long temp = number % 10000 ;  (取末尾4位)
- int ge = temp % 10 ;
- int shi = temp / 10 % 10 ;
- int bai = temp / 100 % 10 ;
- int qian = temp / 1000 % 10 ;

**2.十位、百位为0时，需要加"零"情况 :**

- 前一位不为0，若为0，则可能已经加过"零" ;
- 整个temp > 99 / 999 （即 前一位不能为0，不然根本没有加0的必要，比如 1 0001）



## HJ92 在字符串中找出连续最长的数字串 

最简便方式 —— 以非数字正则表达式为切割点

String[] digits = ` input.split("[^0-9]+")` 



## HJ32 对称密码通信 

- 遍历该字符串 ;
- 每一位字符均传入 getLong(String str,int start,int end) :

```java
while (start>=0 && end<str.length() && str.charAt(start)==str.charAt(end)) {
            start--;
            end++;
}
    
return end-start-1;
```

- 应有两种策略  getLong(str,i,i) 以及 getLong(str,i,i+1) , 利用Math.max（）获取两个返回值中的较大值



## HJ33 整数与IP地址间的转换 

**ipv4 -> int**

- 以"\\\."进行拆分，四部分每部分均为0~255范围内的整数 ;
- Integer.toBinaryString(n) ---> 不足8位则在前方补足0;
- 前后拼接获取一个 32位的String ;
- Long.parseLong(string, 2) ---> 以2进制解析String，获取一个Long类型的长整型 ;

**int -> ipv4**

- Long.toBinaryString(int) 获取一个2进制数 ;
- 不足32位在前方补"0" ;
- 截串str.substring(0, 8)，获取String[ ] ;
- 每一个元素，以 Integer.parseInt(str, 2)



## HJ39 判断两个IP是否属于同一子网 

**掩码是否合法？**

转变为32位二进制，2位2位判断，若str.equals（"01"），则掩码不合法 



## HJ77 火车进站 

深度优先算法 :

trainout( int[] trains, String str , int in , int out , Stack stack ) {

//1. 若out == trains.length => 将str追加到list中

//2. 若stack不为空 :

- 弹出一列火车，存为 temp ;
- trainout( int[] trains, **String str + temp +" "**, int in , **int out+1** , Stack stack ) ;
- 放回原样 : stack.push（temp）

//3. 若in < trains.length :

- stack.push（trains[in]）;
- trainout( int[] trains, String str , **int in+1** , int out , Stack stack ) ;
- stack.pop() ;  —— 回归原位

}





## HJ30  字符串合并处理 

对排序后的字符串中的'0'~'9'、'A'~'F'和'a'~'f'字符，需要进行转换操作  :

- 16进制-> 10进制 :  利用字符与数字的转换关系，比如int b =  ‘B’-（‘A’-10）;
- 10进制 -> 2进制 : 不断除以2，余数是否为1决定该位是否为1，从左往右顺序 ;
- 2进制 -> 倒序 -> 10进制
- 10进制 -> 按照16进制输出（0~E）: 与步骤一道理相当



##HJ24 合唱队 

**最长上升序列**

外层循环 : 从<u>左往右</u>逐个遍历下标i

内层循环 : 从<u>左往右</u>逐个遍历下标k<i

**最长下降序列**

外层循环 : 从<u>右往左</u>逐个遍历下标i

内层循环 : 从<u>右往左</u>逐个遍历下标k>i



## HJ20 密码验证合格程序 

1.长度超过8位

2.包括大小写字母.数字.其它符号,以上四种至少三种

- 导包  java.util.regex.*
- Pattern p = Pattern.compile("[a-z]") ; —— 大小写字母.数字.其它符号 各验证一次
- if (p.matcher(string).find()) count++;

3.不能有长度大于2的不含公共元素的子串重复 （注：其他符号不含空格或换行）

```java

public static boolean doRepeat(String str, int start, int end) {
        if (end >= str.length()) { 
            return true ; 
        } else if (str.substring(end).contains(str.substring(start,end))) { 
            return false ;
        } else { return doRepeat(str, start+1, end+1); }
    }
```









## HJ21 简单密码

明文中的小写字母转为数字，规律类似手机键盘：

1--1， abc--2, def--3, ghi--4, jkl--5, mno--6, pqrs--7, tuv--8 wxyz--9, 0--0

```java
else if(c[i]>='a'&& c[i]<='r') { c[i]=(char) ((c[i]-'a')/3+2+'0'); }
```



## HJ26 字符串排序

规则 1 ：英文字母从 A 到 Z 排列，不区分大小写。  如，输入： Type 输出： epTy  

规则 2 ：同一个英文字母的大小写同时存在时，按照输入顺序排列。  如，输入： BabA 输出： aABb 

规则 3 ：非英文字母的其它字符保持原来的位置。如，输入： By?e 输出： Be?y



- Character.isLetter(char c)   Character的工具类，可以判断该字符是否为字母
- Character.toLowerCase(char c) Character的工具类，可以将字符转变为大小写字符

```java
list.sort(new Comparator<Character>(){
            public int compare(Character o1,Character o2) {
                return Character.toLowerCase(o1)
                    - Character.toLowerCase(o2);
            }
});
```



## HJ59 找出字符串中第一个只出现一次的字符 

**关键思路 :** 

遍历字符串里的字符，第一次出现以下情况则可打印输出 :

 ` str.indexOf(str.charAt(i)) == str.lastIndexOf(str.charAt(i)`

备注 : 设置flag，如果没有满足条件的则输出 -1



## HJ63 DNA序列 

已知某序列String str, 求其中C\G字母出现的次数：

` str1 = str.replaceAll("[^CG]","")` 

C\G字母出现的次数 = str.length() - str1.length() 

##HJ65 查找两个字符串a,b中的最长公共子串 

- 选取较短串，双重for循环遍历:
  - 外层 : 逐个遍历下标 i ;
  - 内层 : **从最尾向 i 遍历**（则碰到地第一个子串情况，就是从i下标出发的最长串），寻找是否有 长串.contains（短串）情况 => 若是，则可直接break内循环 ; 



## HJ55 挑7 

**要求 ：** 输出小于等于 n 的与 7 有关数字的个数，包括 7 的倍数，还有包含 7 的数字（如 17 ，27 ，37 ... 70 ，71 ，72 ，73...）的个数（一组测试用例里可能有多组数据，请注意处理） 

遍历，求满足以下条件：

1. 能被7整除；
2. 转变为字符串，string.contains("7")。



## HJ67 24点游戏算法 

dfs(int temp, int n)

输入的四个数字组成的数组[]、存储是否已使用过的数组boolean[]、flag=0 三者均存储为 成员变量



## HJ64 MP3光标位置 

假设:

songNum —— 歌曲总数

pageNum —— 页面总歌曲数

int current = 1; —— 当前光标所在位置 ;

int index = 1; —— 光标在页面中的次序 ;

```
if (上移) {

    if (current==1) {

        current==songNum;

        index==pageNum;

    } else {
        current--;
		if(index==1) {
            index = pageNum;
		} else {
            index--;
		}
    }


} else {

	//下移情况
	if (current==songNum) {

        current==1;

        index==1;

    } else {
        current++;
		if(index==4) {
            index = 1;
		} else {
            index++;
		}
    }

}
```













