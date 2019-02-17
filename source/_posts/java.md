---
title: java
---
> 递归
```bash
public class TestDemo {  
    public static void main(String[] args) {
            System.out.println(sum(100));  
    }  

    public static int sum(int num){  
        if(num == 1){  
            return 1;  
        }  
        return num + sum(num-1);  
          
    }  
  
} 

```
* 终止条件
* 递归一下次参数
* 返回递归函数