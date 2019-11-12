#### 题目描述（一般）
![](https://winterliublog.oss-cn-beijing.aliyuncs.com/Japanese-Language-Learning/3-LongestSubstrings.png)

要求无重复的一个最长的字符串的长度

#### 法一（自己写的：数组加循环）
```` java  
class Solution {
    public int lengthOfLongestSubstring(String s) {
        //简单的就应该是两层for循环，第一层for循环从1-结束，里层for从i-结束，里层判断收来的字符与已收到的字符是否存在。如果存在，则len-1,负责继续收
        //复杂在里层循环可以用hash map代替里层循环
        int hi = 0,result = 1,current = 0,mapCount=1;
        boolean newStart = false;
        int s_len = s.length();
        //char[] max = new char[s_len];将数组转成用hash map来存
        Map map = new HashMap();
        
        //max[0] = s.charAt(0);
        if(s.equals("")){
            return 0;
        }
        for(int i = 0;i < s_len;i++){
             //System.out.println("max数组第一个元素现在是："+max[0]);
            for(int j = i+1;j <=s_len;j++){//没找到返回null
                if(map.get(s.charAt(j-1)) == null){//对于这种又不需要你找出来，只要看有没有的，hash map简直不要太好
                    //System.out.println("meiyou");
                    map.put(s.charAt(j-1),mapCount+"");
                    if(mapCount>result){
                       // System.out.println("这里：");
                        result = mapCount; 
                        if(result >= 95){
                            return 95;
                        }
                       // System.out.println("result:"+result);
                    }
                    mapCount++;
                    
                }else{//已经有了
                    //System.out.println("有了");
                    map = new HashMap();
                    mapCount = 1;
                    break;
                }
                
            }  
        }
                
//                 if(isExist(max,s.charAt(j-1)) == false){
//                     //不存在，将这个字符加在最后，maxlen++
//                     if(newStart == false){
//                         max[current] = s.charAt(j-1);
//                         current++;
//                         if(current>result){
//                             result = current;     
//                         }
//                         if(result >= 95){
//                             return 95;
//                         }
                         
//                         //System.out.println("xxx");
//                     }else{//重新开始往字符数组加元素
//                         max[hi] = s.charAt(j-1);
//                         hi++;
//                         if(hi>result){
//                             result = hi;     
//                         }
//                         if(result >= 95){
//                             return 95;
//                         }
//                          //System.out.println("yyy");
//                     }
                    
//                 }else{
//                     //存在，这次找子串到此结束，先清空该字符数组（这里我直接用一个新的），继续下一次
//                     hi = 0;
//                     max = new char[s_len];
//                     newStart = true;
//                      //System.out.println("zzz");
//                     break;
//                 }
//             }
            
//             //max[i] = s.charAt(i);
//         }
        //System.out.println(":"+max[1]+":");//空字符而不是null
        return result;
    }
    // public boolean isExist(char[] max,char ch){
    //      //System.out.println("调用");
    //     boolean flag = false;
    //     for(int i = 0;i < max.length;i++){
    //         if(max[i] == ch){
    //             flag = true;
    //              //System.out.println("nimmm");
    //             return flag;
    //         }
    //     }
    //     return flag;
    // }
}   

````  


这些注释的代码就是自己最开始想到的，前面两层循环，第三层是一个 isExist 函数，同样是一层循环，来判断 max[] 数组中是否存在当前字符。只不过后面换成了用 
hashmap。这种是仅仅需要判断里面存不存在的，用 hash map 简直不要太好。 
#### 一个看不懂的答案  
the basic idea is, keep a hashmap which stores the characters in string as keys and their positions as values, and keep two pointers which 
define the max substring. move the right pointer to scan through the string , and meanwhile update the hashmap. If the character is already
in the hashmap, then move the left pointer to the right of the same character last found. Note that the two pointers can only move forward.  
  
```` java 
   public int lengthOfLongestSubstring(String s) {
        if (s.length()==0) return 0;
        HashMap<Character, Integer> map = new HashMap<Character, Integer>();
        int max=0;
        for (int i=0, j=0; i<s.length(); ++i){
            if (map.containsKey(s.charAt(i))){
                j = Math.max(j,map.get(s.charAt(i))+1);
            }
            map.put(s.charAt(i),i);
            max = Math.max(max,i-j+1);
        }
        return max;
    }  
````  

