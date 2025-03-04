---
layout: post
title: LLM开发者教程——LangChain
categories:
- 人工智能
- 大模型
tags:
- LLM
- 大模型
- 大模型开发
---

## 基本概念

LangChain是一个用于开发基于LLM应用的框架，LangChain由以下包构成

-   langchain-core：包含核心组件
-   langchain：包含了通用的模型、链等组件
-   langchain-community：由社区维护，包含第三方集成
-   langchain-*：langchain将一些流行的集成，如openai，单独拆分到各自的包中
-   langgraph：提供了用于创建常见类型代理的高级接口，以及用于组合自定义流程的低级API
-   langserve：一个将LangChain链部署为REST API的包
-   LangSmith：一个开发者平台，让你能够调试、测试、评估和监控LLM应用程序

## LCEL

langchain表达式（LCEL），langchain通过重载`__or__`运算符和`__ror__`运算符实现了通过`|`来组合各个组件，如模板、模型、解析器等

e.g.

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

# 定义提示模板
prompt = ChatPromptTemplate.from_template("回答用户关于{topic}的问题：{question}")

# 初始化模型
model = ChatOpenAI()

# 定义输出解析器
output_parser = StrOutputParser()

# 使用管道符组合链：提示模板 → 模型 → 输出解析器
chain = prompt | model | output_parser

# 调用链并输出结果
result = chain.invoke({"topic": "科技", "question": "量子计算的未来发展"})
print(result)  # 输出解析后的字符串结果
```

**Runnable基类**

`Runnable`类是langchain链的核心类，langchain核心类图如下

<img src="assets/640.webp" alt="UML" style="zoom: 67%;" />

`Runnable`类部分定义如下

```python
class Runnable(Generic[Input, Output], ABC):
    """A unit of work that can be invoked, batched, streamed, transformed and composed.
     Key Methods
     ===========
    * invoke/ainvoke: Transforms a single input into an output.
    * batch/abatch: Efficiently transforms multiple inputs into outputs.
    * stream/astream: Streams output from a single input as it's produced.
    * astream_log: Streams output and selected intermediate results from an input.
    """
    def __or__() -> RunnableSerializable[Input, Other]:
        """Compose this runnable with another object to create a RunnableSequence."""
        return RunnableSequence(self, coerce_to_runnable(other))
    def __ror__() -> RunnableSerializable[Other, Output]:
        """Compose this runnable with another object to create a RunnableSequence."""
        return RunnableSequence(coerce_to_runnable(other), self)
```

`Runnable`类包含的核心方法如下，以a开头的方法为异步方法

-   `invoke`/`ainvoke`：简单地将输入转换为输出，即调用
-   `batch`/`abatch`：将多个输入转换为输出
-   `stream`/`astream`：流式调用
-   `astream_log`：在异步流式调用的过程返回中间结果

与`Runnable`相关的重要类如下

-   `RunnableSequence`

    `RunnableSequence`是最重要的一个类，可以直接创建可运行的对象，并且将对象的输出作为下一个对象的输入

    使用`RunnableLambda`类，可以直接将一个函数转换为`RunnableSequence`

    ```python
    from langchain_core.runnables import RunnableLambda
    
    def add_one(x: int) -> int:
        return x + 1
    
    def mul_two(x: int) -> int:
        return x * 2
    
    runnable_1 = RunnableLambda(add_one)
    runnable_2 = RunnableLambda(mul_two)
    sequence = runnable_1 | runnable_2
    # Or equivalently:
    # sequence = RunnableSequence(first=runnable_1, last=runnable_2)
    sequence.invoke(1)
    await sequence.ainvoke(1)
    
    sequence.batch([1, 2, 3])
    await sequence.abatch([1, 2, 3])
    ```

-   `RunnableParallel`

    支持langchain的并行执行分支，将一个输入分发到独立的多个串行分支中进行处理，返回一个结果map

    ```python
    from langchain_core.runnables import RunnableLambda
    
    def add_one(x: int) -> int:
        return x + 1
    
    def mul_two(x: int) -> int:
        return x * 2
    
    def mul_three(x: int) -> int:
        return x * 3
    
    runnable_1 = RunnableLambda(add_one)
    runnable_2 = RunnableLambda(mul_two)
    runnable_3 = RunnableLambda(mul_three)
    
    sequence = runnable_1 | {  # this dict is coerced to a RunnableParallel
        "mul_two": runnable_2,
        "mul_three": runnable_3,
    }
    
    sequence.invoke(1)  # {"mul_two": 4, "mul_three": 6}
    ```

-   `RunnableBranch`

    `RunnableBranch`实现了分支判断

    ```python
    from langchain_core.runnables import RunnableBranch
    
    branch = RunnableBranch(
        (lambda x: isinstance(x, str), lambda x: x.upper()),
        (lambda x: isinstance(x, int), lambda x: x + 1),
        (lambda x: isinstance(x, float), lambda x: x * 2),
        lambda x: "goodbye",
    )
    
    branch.invoke("hello") # "HELLO"
    branch.invoke(None) # "goodbye"
    
    # 等同于以下语句
    def branch(x):
        if isinstance(x, str):
            return x.upper()
        elif isinstance(x, int):
            return x + 1
        elif isinstance(x, float):
            return x * 2
        else:
            return "goodbye"
    ```

## 组件

**聊天模型**

使用一系列消息作为输入并返回聊天消息作为输出的语言模型，聊天模型支持为对话消息分配不同的角色，有助于区分来自AI、用户和系统消息等指令的消息

聊天模型类以Chat开头，通常有以下参数

-   `model`：模型名称
-   `temperature`：采样温度
-   `timeout`：请求超时
-   `max_tokens`：生成的最大令牌数
-   `stop`：默认停止序列
-   `max_retries`：请求重试的最大次数
-   `api_key`：大模型供应商的API密钥
-   `base_url`：发送请求的端点

**消息**

聊天模型交互的内容通过消息封装，主要有`SystemMessage`、`HumanMessage`、`AIMessage`、`ToolMessage`，每种消息都有`role`和`content`两个属性

-   `HumanMessage`：输入prompt
-   `AIMessage`：输出completion，其中的`tool_calls`属性表示模型调用工具的决策，为一个`ToolCall`列表，每个`ToolCall`包含以下属性
    -   `name`：应该被调用的工具的名称
    -   `args`：该工具的参数
    -   `id`：该工具调用的id
-   `SystemMessage`：系统prompt
-   `ToolMessage`：包含工具调用的结果，除了`role`和`content`，该消息还有以下属性
    -   `tool_call_id`：传达调用该工具以生成此结果的调用id
    -   `artifact`：可以用于传递工具执行的任意工件，这些工件有助于跟踪，但不应发送给模型

**提示词模板**

