---
layout: post
title:  "委托Delegate和事件Event用法的解释"
date:   2016-09-02
desc: "本文用简单的语言说明C#中委托Delegate和事件Event的写法"
keywords: "C#,delegate,event,委托,事件"
categories: [HTML]
tags: [C#,.NET]
icon: icon-csharp
---

# 1. 场景

>一家人爸爸、妈妈、baby、狗在看电视，突然baby哭了。此时，爸爸和妈妈听到baby的哭声, 立刻起来哄孩子。妈妈说：“宝贝，妈妈爱你。” 爸爸说：“宝贝，爸爸也爱你。” 但是坐在一旁的狗毫无反映，虽然它也听到了哭声.如果baby哭的时候，奶奶回来了。那么，奶奶也会去哄孩子。为什么？因为奶奶进入了这个场景（也就是说，奶奶这个功能模块被加入进来），她也“关注”这件事。如果baby哭的时候，奶奶已经出门了。那么，奶奶不会去哄孩子。为什么？因为奶奶这个功能模块已经被移除了。

在这个场景中，有2个人关注baby哭这件事，并各自作出反映，狗不关注，不做出任何反映。奶奶的反应则会变化。
用计算机语言如何描述这个场景呢？由一个“哭”的事件，所引起的不同的处理方式。

简单说，baby是事件的“发起者（source）”，爸爸，妈妈是事件的“订阅者”。奶奶在家时，订阅这个事件；不在家时，不订阅（不关注）。狗始终没有“订阅（关注）”这件事。


# 2. 为什么要用Delegate和Event

你有没有注意到，在这个例子中，不是所有的物体都必须对“哭”事件作出反映。这合理吗？当然，太合理了。在我们的生活中，不正是这样吗？一件事没有必要让社会上的所有人都关注，都做出反映。如果这样，显然社会运行“成本”太高。   

抽象来说，“哭”这件事和妈妈、爸爸、奶奶的哄孩子这个动作是相关的，和狗的动作无关。如果奶奶在，她和哭是相关的；不在，不相干。在程序中，你要如何实现“奶奶模块”的加入和移除，不影响其他人作出的动作？也就是降低奶奶模块和爸爸、妈妈模块的耦合度。

**Delegate和Event正是用来降低程序间的耦合度，利于程序的扩展和删除。**   



# 3. 代码说明

代码如下：

``` c#
/*本例说明在 委托-事件 中传递参数的用法
本例中有4个关键类：
1. Senario ：应用场景
2. Baby : 事件的发布者 Publisher
3. Mum :  事件的接受者 subscriber
4. Dad :  事件的接受者 subscriber
本例中有1个用来传递参数的类：
5. currentTime : EventArgs 
*/
using System;
 
//构建一个自定义的时间参数类
//用来在event-delegate中传递参数
public class currentTime : EventArgs
{
	public string Time;
}

public class Senario
{ //这里我们构建一个baby哭的场景
	//故事从这里开始
	public static void Main()
	{
		//实例化一个baby
		Baby baby = new Baby();
		//实例化一个Mum
		Listener_Mum m = new Listener_Mum();
		m.Subscribe(baby); //mum 关注baby哭这件事
		//实例化一个Dad
		Listener_Dad d = new Listener_Dad();
		d.Subscribe(baby); //dad 关注baby哭这件事
		
        //触发事件，事情发生了。let the baby cry
		baby.cry();
	}
}

public class Baby
{

	//1.声明一个委托，其实就是个“命令”
	public delegate void cryEventHandler(Baby sender, currentTime e);
	//2.声明一个事件, 基于该“命令”
	public event cryEventHandler onCry;
    //3. Fire the event 事件
	public void cry()
	{		
		if (onCry != null) //如果有人关注了这个事件，baby就哭
		{
			Console.WriteLine("\n\tBaby crying : ....#%$&*((*$@)()_^~&#....\n");
			currentTime t = new currentTime();
			t.Time = " [pass parametre here]";
			//3. Fire the event 事件
			onCry(this, t);
		}
	}
}

public class Listener_Mum
{
	public void Subscribe(Baby b)
	{
		b.onCry += new Baby.cryEventHandler(HeardIt);//妈妈关注（订阅）了哭这件事
	}

	private void HeardIt(Baby b, currentTime e)
	{
		System.Console.WriteLine("Mum HEARD IT. get:"+e.Time+".Do something here");
	
	}
}

public class Listener_Dad
{
	public void Subscribe(Baby b)
	{
		b.onCry += new Baby.cryEventHandler(HeardIt);//爸爸关注（订阅）了哭这件事
		//如果爸爸这个类被移除了，我们还需要修改程序其他地方吗？答案是只需要改一个地方:
		//Senario类的main()中删除对爸爸类的实例化即可。
		//你看，这样是不是把代码维护成本极大的降低了？！你不需要改妈妈类，不需要改baby类。
	}

	private void HeardIt(Baby b, currentTime e)
	{
		System.Console.WriteLine("Daddy HEARD IT. get:"+e.Time+".Do something here");
	}
}
```

# 4. 代码分析
使用委托-事件,必须有3个步骤：
``` c#
    //1.声明一个委托，其实就是个“命令”
	public delegate void cryEventHandler(Baby sender, currentTime e);
	//2.声明一个事件, 基于该“命令”
	public event cryEventHandler onCry;
    //3. Fire the event 事件
	public void cry()｛...｝
```