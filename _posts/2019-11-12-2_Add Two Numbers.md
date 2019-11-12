#### 题目描述（一般）
![](https://winterliublog.oss-cn-beijing.aliyuncs.com/Japanese-Language-Learning/2-Add-Two-Numbers.png)

就是给你两个 ListNode，每个都存了一个非负整数，要你把它们两个加起来然后存到一个（新的或者旧的）ListNode 中，并返回。 
这是我的代码  
```java  
// // *
// //  * Definition for singly-linked list.
//   public class ListNode {
//       int val;
//       ListNode next;
//       ListNode(int x) { this.val = x; }
//   }//每个结点就是这个 ListNode 类的一个对象

class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        //传过来的是两个对象（已经赋好了值）
        ListNode LC = new ListNode(0);
        ListNode pa = l1;
        ListNode pb = l2;
        
        ListNode pc = LC;//pc 指向要返回的 ListNode 的最后一个结点
        System.out.println("pc2222:"+pc.val);
        System.out.println("pc的next:"+pc.next);
        int addnum = 0;//两个数相加的结果，存到 LC 中
        int co = 0;//0 表示没有进位，1 表示进位
        
        /*
        输出l1,l2
        */
        // while(pa!=null){
        //     System.out.println(pa.val);
        //     pa = pa.next;
        // }
        
        while((pa !=null) && (pb != null)){
            //pc = ;
            addnum = pa.val + pb.val + co;
             //System.out.println("1111pa:"+addnum);
            if(addnum < 10){
                co = 0;
                //pc.val = addnum;
                pc.next = new ListNode(addnum);//利用构造函数创建一个值为 addnum 的结点
                // if(LC == null){
                //     LC = pc;
                // }
                // while(LC != null){
                    
                //     LC=LC.next;
                // }
                 //System.out.println("pc:"+pc.val);
                pc = pc.next;
                 System.out.println("我的pc:"+pc.val);
                
                 System.out.println("通过LC来取值:"+LC.next.val);
                pa = pa.next;
                //System.out.println("pa:"+pa.val);
                pb = pb.next;
                 //System.out.println("pb:"+pb.val);
                //pc.next = new ListNode();
            }else{//相加大于 10，需要进位
                co = 1;
                addnum = addnum % 10;
                //pc.val = addnum;//最后一个进位要注意，如果最后产生了进位，同时pa、pb没有了，那么要把这个进位加到pc里面，也就是加1！
                pc.next = new ListNode(addnum);//利用构造函数创建一个值为 addnum 的结点
                //System.out.println("pc:"+pc.val);
                pc = pc.next;
                pa = pa.next;
                pb = pb.next;
            }
        }
        while(pa != null){
            addnum = pa.val + co;
            if(addnum < 10){
                co = 0;
                //pc.val = addnum;
                pc.next = new ListNode(addnum);//利用构造函数创建一个值为 addnum 的结点
                pc = pc.next;
                pa = pa.next;
                //pc.next = new ListNode();
            }else{//相加大于 10，需要进位
                co = 1;
                addnum = addnum % 10;
                //pc.val = addnum;//最后一个进位要注意，如果最后产生了进位，同时pa、pb没有了，那么要把这个进位加到pc里面，也就是加1！
                pc.next = new ListNode(addnum);//利用构造函数创建一个值为 addnum 的结点
                pc = pc.next;
                pa = pa.next;
            }
        }
        
        while(pb != null){
            addnum = pb.val + co;
            if(addnum < 10){
                co = 0;
                //pc.val = addnum;
                pc.next = new ListNode(addnum);//利用构造函数创建一个值为 addnum 的结点
                pc = pc.next;
                pb = pb.next;
                //pc.next = new ListNode();
            }else{//相加大于 10，需要进位
                co = 1;
                addnum = addnum % 10;
                //pc.val = addnum;//最后一个进位要注意，如果最后产生了进位，同时pa、pb没有了，那么要把这个进位加到pc里面，也就是加1！
                pc.next = new ListNode(addnum);//利用构造函数创建一个值为 addnum 的结点
                pc = pc.next;
                pb = pb.next;
            }
        }
        if(co != 0){
            pc.next = new ListNode(1);
        }
        // while(LC != null){
        //     System.out.println("LC:"+ LC.val);
        //     LC = LC.next;
        // }
        
        return LC.next;
    }
}  
```  
#### 第一种解法  
```java  
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;//加了这两句就知道哪个需要进行下一个
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
}  
```
其实思想还是挺简单的，就是 l1 和 l2 两个链表，位对位相加，超过 10 则进位，用 co 来记录进位。其实每一次位的相加看成两个 0-9 的数字相加，如果这一位没有就用 0 来代替。我写的代码其实很多代码是冗余的，所以需要在逻辑上进行优化，去除冗余代码。  
代码优化：A wonderful solution but for the time consumed, just try replace sum%10 with sum>=10 ? sum-10 : sum will extremely speed up!
过程图：  
![](https://winterliublog.oss-cn-beijing.aliyuncs.com/Japanese-Language-Learning/2-Add-Two-Numbers-1.png)  
#### 总结  
时间复杂度：O（max(m,n)）  
空间复杂度：O（max(m,n)），但是新的链表的长度是 O（max(m,n)）+ 1.
