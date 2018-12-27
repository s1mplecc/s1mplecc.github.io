---
title: Java 泛型之 LinkedStack
date: 2018-02-24T03:38:55.000Z
tags: ['Java', 'Data Structure']
categories: [Coding]
---
## Preface

> Java 泛型可以说是大大增强了这门语言的动态性和灵活性，本文基于《Java 编程思想》上的代码案例，实现了一个带泛型的**下推堆栈 LinkedStack** ，也算是巩固一下 Java 泛型和数据结构的知识
 
- 示例代码已经提交到 Github 上，[链接](https://github.com/s1mplecc/MyLinkedStack)

## LinkedStack

- **LinkedStack** 类的代码

    ```
    package info.s1mple;

    public class LinkedStack<T> {
        private static class Node<T> {
            T item;
            Node<T> next; //下一个节点的引用，类似于C中的指针

            Node() { //没有参数的Constructor，用于初始化时生成栈顶
                item = null;
                next = null;
            }

            Node(T item, Node<T> next) {
                this.item = item;
                this.next = next;
            }

            boolean isEmpty() { //栈空标记
                return item == null && next == null;
            }
        }

        private Node<T> top = new Node<>(); //初始化栈顶

        public void push(T item) {
            top = new Node<>(item, top); //与上一节点链接
        }

        public T pop() {
            T result = top.item;
            if (!top.isEmpty()) {
                top = top.next; //指针后移
            }
            return result;
        }

    }

    ```

> 定义了一个 **LinkedStack** 类，包含一个内部类 **Node** ，模型化了链表的 Node 节点，包括**值**和**指向下一个节点的指针**俩个部分，**堆栈可以看成最左边是栈顶的单链表**，通过`top = new Node<>(item,top);`将节点串联起来

![linked](/0.png)

## Testing

- 测试类的代码，测试 String 和 Integer 两种类型

    ```
    import info.s1mple.LinkedStack;
    import org.junit.jupiter.api.Test;

    class LinkedStackTest {

        @Test
        void should_string_success() {
            LinkedStack<String> lss = new LinkedStack<>();
            for (String s : "This is an Apple.".split(" ")) {
                lss.push(s);
            }
            String s;
            while ((s = lss.pop()) != null) {
                System.out.println(s);
            }
        }

        @Test
        void should_integer_success() {
            LinkedStack<Integer> lsi = new LinkedStack<>();
            for (int i : new int[]{1, 2, 3, 4, 5}) {
                lsi.push(i);
            }
            Integer i;
            while ((i = lsi.pop()) != null) {
                System.out.println(i);
            }
        }
    }

    ```

> 需要注意的是，上述代码第24行`Integer i`必须是 **Integer** 而不能是 **int** ，否则25行`!=null`会报错。这是因为 Java 的**自动包装机制** ，基础数据类型会被包装成对应的对象 ，泛型只支持 **Object** 对象，也正是通过自动包装机制而间接支持基础数据类型
    
- String 类型和 Integer 类型的测试结果
    ![Screen-Shot-2018-02-24-at-2.29.06-PM-2](/1.png)

    ![Screen-Shot-2018-02-24-at-2.32.11-PM](/2.png)

## Conclusion

> 可以看到两种类型都可以正常运行，并满足堆栈**先进后出**的特点。在定义 **LinkedStack** 时通过`<T>`和`T item`引用泛型。在使用时通过`LinkedStack<String> lss = new LinkedStack<>();`或者`LinkedStack<Integer> lsi = new LinkedStack<>();`指定具体类型。而正是泛型给 Java 这种静态语言提供了**动态性**


