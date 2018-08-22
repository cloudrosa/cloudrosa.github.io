---  
layout: post  
title: 大话设计模式  
description: 读书笔记  
category: nothing  
subtype: read  
---  

* 简单工厂类
![简单工厂UML](/images/designmode/SimpleFactory-Uml.jpg)
```
/// <summary>
/// 基类
/// </summary>
public class Operation
{
    public double NumberA { get; set; }
    public double NumberB { get; set; }

    public virtual double GetResult()
    {
        return 0;
    }
}

/// <summary>
/// 加法类
/// </summary>
public class OperationAdd:Operation
{
    public override double GetResult()
    {
        return NumberA + NumberB;
    }
}

/// <summary>
/// 减法类
/// </summary>
public class OperationSub:Operation
{
    public override double GetResult()
    {
        return NumberA - NumberB;
    }
}

/// <summary>
/// 乘法类
/// </summary>
public class OperationMul:Operation
{
    public override double GetResult()
    {
        return NumberA * NumberB;
    }
}

/// <summary>
/// 除法类
/// </summary>
public class OperationDiv:Operation
{
    public override double GetResult()
    {
        if (NumberB == 0)
            throw new Exception("除数不能为0.");
        return NumberA / NumberB;
    }
}

/// <summary>
/// 运算类
/// </summary>
public class OperationFactory : Operation
{
    public static Operation CreateOperation(string operation)
    {
        Operation ope = null;
        switch (operation)
        {
            case "+":
                ope = new OperationAdd();
                break;
            case "-":
                ope = new OperationSub();
                break;
            case "*":
                ope = new OperationMul();
                break;
            case "/":
                ope = new OperationDiv();
                break;
        }
        return ope;
    }
}

//简单调用
Operation operation = OperationFactory.CreateOperation("+");
operation.NumberA = 10;
operation.NumberB = 10;
Console.WriteLine(operation.GetResult());
```

* 