### 10.6

* [x] leetcode
* [ ] loopring合约部分总结
* [ ] zksync文档总结

### 10.7

* [ ] loopring合约部分总结
* [ ] leetcode
* [ ] 微软面试准备



### 10.22

* [ ] 微软面试
  1. 自我介绍英文
  2. 项目难点和最有成就点介绍
  3. 从起始位置起，最短的包含所有元素的子数组。(使用一个右指针，使用set记录当前已经出现的元素，当碰到一个没有在set中出现的元素时，更新结果的index值进行加1，最后针对自己写的程序进行自己构造测试用例。)
  4. 从任意位置起，最短的包含所有元素的子数组。(滑动窗口，首先确定第一个包含所有元素的子数组，然后移动左指针，改变res的值，直到不符合条件之后，向后移动右指针的位置寻找缺失的值。)



```c++
Input: [1,2,3,4,4,4]
set<int> s = {1,2,3,4}

int left;
map<int, int> m;
m[i] >= 1;
int right;

[left, right] 包含所有元素的子数组
min([left, right])

Output: 3 index
#include <bits/stdc++.h>
using namespace std;
class Solution() {
public:
    int getFirstIndex(vector<int>& v) {
        if (v.size() >= 1) {
            return v.size();
        }
        set<int> ssrc(v.begin(), v.end());
        set<int> sdes;
        map<int, int> m;
        int left = 0, int right = 0;
        for(int i = 0; i < v.size(); i++) {
            sdes.insert(v[i]);
            m[v[i]]++;
            if (ssrc.size() == sdes.size()) {
                right = i;
                break;
            }
        }
        
        int res = right-left+1;
        int val = v[left];
        while(true) {
            if (right == v.size()-1 && right - left < ssrc.size()) {
                break;
            }
            while(left < right && m[v[left]] > 0) {
               if (1 == m[vleft]) {
                    m[v[left]]--;
                	  val = v[left];
                    left++;
                    break;
                }
               res = min(res, right-left+1);
               m[v[left]]--;
               left++;
            }
            right++;
            while(right < v.size() && v[right] != val) {
                if (right == v.size()-1) {
                    break;
                }
                right++;
            }
        }
        
        
        return res;
    }
}

int main() {
    vector<int> v = {1,2,2,3,4,2,1};
    Solution s;
    cout << s.getFirstIndex(v) << endl;
    return 0;
}
```

二面：

1. 项目中的难点和影响深刻的点。
2. jvm
3. acid
4. mysql的事务隔离级别。
5. 解释一下幻影读
6. 数组的子序列
7. 数组连续子序列的最小值之和。

```c++
#include <bits/stdc++.h>
using namespace std;

[]
[1,1,1,1,1]
[1,1,1,2,2,2,3,3,3]
[1,2,3,4]
[1,2,2,2,2,4]
[1,1,1,1,3,4]
[2,3,4,4,4,4]
3 1 4
[3] [1] [4] [3,1] [1,4] [3,1,4]
3+1+4 +1+1+1
[3] 3 
[3,1] 1
[3,1,4] 1
[1] 1
[1,4] 1
[4] 4

1,3,4

class Solution{
public:
	
	int getArrayMinVal<vector<int>& v> {
        if (0 == v.size()){
        	return 0; 
    		}
			int res = 0;
			for (int i = 0; i < v.size(); i++) {
                int minVal = v[i];
                res += minVal;
                for (int j = i+1; j < v.size(); j++) {
                    if (v[j] < minVal) {
                        minVal = v[j];
                    }
                    res += minVal;
                }
         }

			return res;
   }

	vector<vector<int>> getArray(vector<int> &v) {
        set<vector<int>> res;
        res.push_back(vector<int>());
        if (0 == v.size()) {
            return vector<vector<int>>(res.begin(), res.end());
        }
        for (int i = 1; i <= v.size(); i++) {
            vector<bool> vflag(v.size(), false);
            vector<int> tmp;
            dfs(res, v, tmp, i, vflag);
        }
        
        return vector<vector<int>>(res.begin(), res.end());
    }

	void dfs(set<vector<int>>& res, vector<int>& v, vector<int>& tmp, int stp, vector<bool>& vflag) {
        if (0 == stp) {
            sort(tmp.begin(), tmp.end());
            res.insert(tmp);
            return;
        }
        
        for(int i = 0; i < v.size(); i++){
            if(!vflag[i]) {
                vflag[i] = true;
                tmp.push_back(v[i]);
                dfs(res, v, tmp, stp-1, vflag);
                tmp.pop_back();
                vflag[i] = false;
            }
        }
    }
    
};
```



