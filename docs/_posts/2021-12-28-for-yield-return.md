---
title: For Yield Return学习笔记
---

我有个List<string>里面包含了20个英文数字的元素，当我要分解成四个List<string>里面包含5个英文数字的元素，该怎么处理？

首先我们先赋初值给numberList
	
```
var numberList = new List<string>
            {
                "one",
                "two",
                "three",
                "four",
                "five",
                "six",
                "seven",
                "eight",
                "nine",
                "ten",
                "eleven",
                "twelves",
                "thirteenth",
                "fourteen",
                "fifteen",
                "sixteen",
                "seventeen",
                "eighteen",
                "nineteen",
                "twenty"
            };
```
然后写一个方法让我们可以重复使用
```
private static IEnumerable<IEnumerable<string>> Separate(IEnumerable<string> numList) 
        {
            var total = numList.Count();
            var list = numList.ToList();
            var batchSize = 5;
            for (int i = 0; i < total; i += batchSize)
            {
								/*这里看个人想用哪一个， 测试了结果都一样*/
                // yield return list.GetRange(i, Math.Min(batchSize, total - 1));
                yield return numList.Skip(i).Take(batchSize);
            }
        }
```
接下来就是分解的时候
```
 //把List转成IEnumerable
 IEnumerable<string> numberListToEnumerable = numberList.AsEnumerable();
	//获取结果
 var result = Separate(numberListToEnumerable);
```
测试结果如以下截图	
<img src="https://i.imgur.com/g5NyFCCl.png">										
	
完整测试代码
```
class Program
    {
        static void Main(string[] args)
        {
            var numberList = new List<string>
            {
                "one",
                "two",
                "three",
                "four",
                "five",
                "six",
                "seven",
                "eight",
                "nine",
                "ten",
                "eleven",
                "twelves",
                "thirteenth",
                "fourteen",
                "fifteen",
                "sixteen",
                "seventeen",
                "eighteen",
                "nineteen",
                "twenty"
            };

            IEnumerable<string> numberListToEnumerable = numberList.AsEnumerable();
            var result = Separate(numberListToEnumerable);

        }

        private static IEnumerable<IEnumerable<string>> Separate(IEnumerable<string> numList)
        {
            var total = numList.Count();
            var list = numList.ToList();
            var batchSize = 5;
            for (int i = 0; i < total; i += batchSize)
            {
                // yield return list.GetRange(i, Math.Min(batchSize, total - 1));
                yield return numList.Skip(i).Take(batchSize);
            }
        }
    }
```
