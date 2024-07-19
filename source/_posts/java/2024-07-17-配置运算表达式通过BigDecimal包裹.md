---
layout: post
title: 配置运算表达式通过BigDecimal包裹
categories: [Java]
description: 配置运算表达式通过BigDecimal包裹
permalink: java/groovy-script.html
tags: Groovy,算术运算
---

需求场景：

通过页面配置算术运算，支持在实例化运行时，根据配置的变量动态计算表达式的值，由于考虑到变量的值可能包含字符串、数字、浮点等不同情况，甚至需要支持金额运算时，为保证结果一致性，转为 `BigDecimal`。

示例代码：
```java
import java.util.List;
import java.util.Objects;
import java.util.Stack;

public class ComputeValueUtil {
    public static String evaluate(List<String> expressions) {
        Stack<String> values = new Stack<>();
        Stack<String> operators = new Stack<>();
        for (String expression : expressions) {
            if (Objects.equals(expression, "(")) {
                operators.push(expression);
            } else if (Objects.equals(expression, ")")) {
                while (!"(".equals(operators.peek())) {
                    values.push(applyOperation(values.pop(), values.pop(), operators.pop()));
                }
                String leftHand = values.pop();
                values.push("(" + leftHand + ")");
                // 弹出左括号
                operators.pop();
            } else if (isOperator(expression)) {
                while (!operators.isEmpty() && hasPrecedence(expression, operators.peek())) {
                    values.push(applyOperation(values.pop(), values.pop(), operators.pop()));
                }
                operators.push(expression);
            } else {
                // 变量
                values.push("new BigDecimal(" + expression + ")");
            }
        }
        while (!operators.isEmpty()) {
            values.push(applyOperation(values.pop(), values.pop(), operators.pop()));
        }
        return values.pop();
    }

    private static boolean isOperator(String op) {
        return "+".equals(op) || "-".equals(op) || "*".equals(op) || "/".equals(op);
    }

    private static boolean hasPrecedence(String op1, String op2) {
        if ("(".equals(op2) || ")".equals(op2)) {
            return false;
        }
        return (!"*".equals(op1) && !"/".equals(op1)) || (!"+".equals(op2) && "-".equals(op2));
    }

    private static String applyOperation(String b, String a, String op) {
        switch (op) {
            case "+":
                return a + ".add(" + b + ")";
            case "-":
                return a + ".subtract(" + b + ")";
            case "*":
                return a + ".multiply(" + b + ")";
            case "/":
                return a + ".divide(" + b + ", 4, RoundingMode.HALF_UP)"; // 使用4位小数精度和四舍五入模式
            default:
                return "";
        }
    }
}
```

