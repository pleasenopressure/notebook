# 协程对象和协程任务
在 Python 里，协程函数用 `async def` 定义，调用它**不会立刻执行**，而是先得到一个“协程对象”
例子：

`import asyncio`

`async def download():`
    `await asyncio.sleep(1)   # 暂停 1 秒，把执行权让出去`
    `return "done"`

`coro = download()            # 这里不会执行 download 里面的代码`
`print(coro)                  # <coroutine object ...>`

要让它真正跑起来，需要 **事件循环（event loop）** 来驱动，比如：

`asyncio.run(download())`

 ## 什么是任务（Task）？

任务是协程的“运行实例”（被事件循环调度的单位）。  
你可以把它理解为：**把一个协程交给 asyncio 去后台跑，并且你能管理它**（查状态、取消、等待结束）。

例子：
`import asyncio`

`async def job():`
    `await asyncio.sleep(10)`
    `return "finished"`

`async def main():`
    `t = asyncio.create_task(job())  # 创建“协程任务”`
    `print(t)                        # <Task pending ...>`
    `result = await t                # 等它结束`
    `print(result)`

`asyncio.run(main())`
这里的 `t` 就是一个 **协程任务**（asyncio.Task）。

---
### 3) 协程 vs 协程任务：一句话区别

- **协程**：还没被调度执行的“可暂停函数对象”（需要 event loop 驱动）
    
- **协程任务（Task）**：协程已经被注册到事件循环中，**正在/即将运行**，可被管理

### await() 的作用
协程任务要想实现协作式取消
- `task.cancel()` 会给协程抛一个 `CancelledError`
    
- 协程必须在 `await` 或主动检查点时才能响应取消
- 所以你需要两层机制：

- **Task.cancel()**：从外部触发
    
- **CancellationToken.checkpoint()**：从内部协作式退出（更可控、更可记录）

`await task` 这一步是在做两件非常具体的事：

1. **等待这个协程任务（Task）执行结束**（完成 / 抛错 / 被取消）
    
2. **把任务的结果或异常“拿回来”**（否则异常可能被你忽略）
    

下面用例子把它讲透。

---

## 1) `await task` 等价于“等它跑完”

你创建任务时：

`task = asyncio.create_task(long_job())`

这表示 `long_job()` 已经在后台开始跑了。

而：

`await task`

表示：

> “我现在暂停当前协程 `main()`，一直等到 task 结束，再继续往下执行。”

这跟“线程里 `join()`”很像，只不过是 async 世界里的等待。

---

## 2) `await task` 还能“拿到返回值”

如果任务有 `return`：

```
async def job():
	await asyncio.sleep(1)     
	return 123  
task = asyncio.create_task(job())
result = await task 
print(result)   # 123
```

**没有 `await task`，你拿不到 job 的返回值。**

---

## 3) 如果任务被取消，`await task` 会抛 `CancelledError`

你取消任务：

`task.cancel()`

只是发出“取消请求”。真正的取消会在任务下一次遇到 `await`（或检查点）时发生。

当你：

`await task`

会立刻得到一个结果：任务会以异常形式结束，于是抛出：

- `asyncio.CancelledError`
    

所以常见写法是：

```
task.cancel() 
try:     
	await task 
except asyncio.CancelledError:
	print("cancelled!")
```

**这就是在“确认取消已经生效并处理它”。**

---

## 4) 如果任务内部报错，`await task` 会把错误抛出来

例如：

```
async def job():
     await asyncio.sleep(1)
task = asyncio.create_task(job()) 
await task  # 这里会抛 ZeroDivisionError
```
为什么重要？  
如果你不 `await task`，这个错误可能变成 “Task exception was never retrieved” 这种警告，排查很痛苦。

---

## 5) 用一句话总结

- `task.cancel()`：发出取消请求（不保证立即停止）
    
- `await task`：**等待任务真正结束，并获取结果/异常（包括取消）**

## token.cancel() 和 task.cancel()的区别

`token.cancel()
task.cancel()`

分别在做两件**不同层面**的事——一个是“**业务层的取消信号**”，一个是“**asyncio 调度层的强制取消**”。

---

## 1) `token.cancel()`：业务层的“协作式取消信号”

**它做什么：**

- 只是把一个标志位设为“已取消”（比如 `token.cancelled = True` 或 event set）
    
- 不会自动打断正在运行的 await
    
- 需要你的代码在合适的地方主动检查：`await token.checkpoint()`（或 `if token.cancelled: ...`）
    

**用在哪：**

- 你想让任务在“安全点”退出：保存 session、发事件、清理子任务、写 trace
    
- 你有多个子任务/组件要共享同一个取消信号（LLM 流、工具调用、评估器都能看 token）
    

**一个直观例子：**

for chunk in stream:
	await token.checkpoint()  # 这里能优雅退出（还能发 cancelled 事件/保存状态）     handle(chunk)

---

## 2) `task.cancel()`：asyncio 层的“强制取消请求”

**它做什么：**

- 向这个 Task 注入一个 `CancelledError`
    
- 让任务在下一次遇到 `await` 时尽快中断（很多 asyncio awaitable 会立刻响应）
    
- 不依赖你自己写 checkpoint，也能打断某些长 await（如 `asyncio.sleep`、部分网络 I/O）
    

**用在哪：**

- 你希望“尽快停下来”，尤其任务正卡在某个 await 上
    
- 防止用户取消后还要等很久才停
    

**一个直观例子：**

`await asyncio.sleep(600)  # 很长 # task.cancel() 可以立刻把这里打断`

---

## 3) 为什么通常要两句一起用？

因为它们优势互补：

- `task.cancel()` 负责 **停得快**（尽快把协程从 await 里拽出来）
    
- `token.cancel()` 负责 **停得稳**（让你的逻辑在检查点做收尾：写状态/发事件/清理）
    

所以你 cancel endpoint 常写成“双保险”：

`token.cancel()   # 告诉业务逻辑：该优雅退出了 
`task.cancel()    # 告诉 asyncio：尽快打断正在 await 的地方`

---

## 4) 你可以用一句话背下来（面试也好讲）

- **token**：面向“业务逻辑”的取消（可传播、可控、可收尾）
    
- **task**：面向“事件循环调度”的取消（快速打断 await）

## 区分async and asyncio
我们用**三个超具体例子**把它讲透：你看完应该能一眼分清 “`async/await` 是什么” vs “`asyncio` 做了什么”，以及 503 过载到底像什么场景。

---

#### 例子 1：只有 `async def`，但不用 `asyncio`，会怎样？

`async def hello():     return "hi"  coro = hello() print(coro)`

输出大概是：

`<coroutine object hello at 0x...>`

这说明什么？

- `async def` 只是**定义了一种“可以暂停/恢复”的函数**。
    
- 调用它不会执行，只是得到一个 **协程对象 coroutine**。
    
- **没有事件循环（asyncio）去“跑”它，它就永远不执行。**
    

👉 类比：`async def` 像你写了一张“任务说明书”，但没雇人去做。

---

#### 例子 2：用 `asyncio.run()` 让协程真正跑起来

`import asyncio  
async def hello():     
	return "hi"  
print(asyncio.run(hello()))`

这次输出：

`hi`

这里发生了啥？

- `asyncio.run(...)` 做了关键工作：  
    **创建事件循环 → 把协程放进去执行 → 执行完关闭循环**
    

👉 类比：你不但写了任务说明书，还叫来了“执行团队”并启动了工作流程。

**结论：**

- `async/await`：描述“任务怎么写”
    
- `asyncio`：负责“任务怎么跑、怎么调度、怎么并发”
    

---

#### 例子 3：`await` 到底“让出了什么”？（最重要的直觉）

看两个函数并发执行：

import asyncio  
async def task_a():
	print("A1")  
	await asyncio.sleep(1)  # 这里暂停，把控制权交回去  
	print("A2")  
async def task_b():    
	print("B1")    
	await asyncio.sleep(1) 
	print("B2") 
async def main(): 
	await asyncio.gather(task_a(), task_b())  
	asyncio.run(main())

你会看到输出类似：

`A1 B1 A2 B2`

重点来了：

- `await asyncio.sleep(1)` 并不是“阻塞 1 秒”
    
- 它是告诉事件循环：  
    **“我现在没事干了，你先去跑别人，1 秒后再回来继续我。”**
    

👉 类比：你在银行办业务，柜员说“系统处理要 1 分钟”，你不必一直站柜台占着位置，你先去旁边等，柜员去服务别的人。

所以：

- `await` 的核心作用：**让出执行权**
    
- `asyncio` 的核心作用：**在你让出的这段时间里，安排别人去跑**
    

---

#### 那 `asyncio.run_in_executor` 是啥？（你 Day3 用到的）

你今天 Gemini streaming 是**同步阻塞**的：

`for chunk in resp_stream:   # 这个循环会一直占着线程     ...`

如果你把它直接写在 async 函数里，会发生：

- 整个 event loop 被占住，别的协程都卡住（SSE/WS 推送也可能卡）
    

所以你用：

- `asyncio.run_in_executor(...)`：把这种“阻塞活”扔到线程池里做
    
- 然后用 `asyncio.Queue` 把结果一段段送回 async 世界
    

👉 类比：

- `asyncio` 是餐厅“总调度”
    
- `await` 是服务员说“我这桌菜在做，先去服务别桌”
    
- `run_in_executor` 是“后厨很慢，专门扔给后厨做，我继续在前厅跑”

## 什么是事件流：把“运行过程”像直播一样推给前端/外部

**事件流 = 程序在运行时持续产生一连串事件（event），并按时间顺序实时发送给订阅者。**

### 例子：你跑 `/session/test/chat` 之后实际发生了什么？

后台 job 在做这些事（不是一次性做完）：

- 开始运行 → 发一个事件：`run_start`
    
- 开始调用模型 → 发：`llm_start`
    
- 模型每吐一段文字（chunk）→ 连续发很多次：`llm_delta`
    
- 完成 → 发：`llm_done`、`run_done`
    
- 如果你中途 cancel → 发：`cancel_requested`，然后 job 停止 → 发：`cancelled`
    
- 如果云端 503 → 发：`error`
    

所以“事件流”看起来像这样（按时间顺序）：

`{"type":"run_start","kind":"chat"}` 
`{"type":"llm_start","model":"gemini-2.5-flash"}` 
`{"type":"llm_delta","text":"多智能体系统是..."}` 
`{"type":"llm_delta","text":"它由多个智能体..."}`
`{"type":"llm_delta","text":"通过协作..."}` 
`{"type":"llm_done"}` 
`{"type":"run_done","kind":"chat"}`

这就是**过程的直播**：一边运行，一边把进度/结果碎片实时推出来。

## 工具调用
###  代码模式总结

这段代码展示了在这个系统中添加新工具的标准范式：

1. **定义函数**：编写一个函数（
    
    ```
    def
    ```
    
     或 
    
    ```
    async def
    ```
    
    ），接收字典 
    
    ```
    args
    ```
    
    ，返回字典结果。
2. **包装为 
    
    ```
    ToolSpec
    ```
    
    **：
    - **Metadata**：给工具起名字 (
        
        ```
        name
        ```
        
        )、写说明书 (
        
        ```
        description
        ```
        
        )、定义参数格式 (
        
        ```
        input_schema
        ```
        
        )。
    - **Behavior**：绑定函数 (
        
        ```
        func
        ```
        
        )、设置它是同步还是异步 (
        
        ```
        is_async
        ```
        
        )、设置超时 (
        
        ```
        timeout_s
        ```
        
        ) 和重试策略 (
        
        ```
        retry
        ```
        
        )。
3. **注册**：调用 
    
    ```
    reg.register(...)
    ```
    
     把说明书提交给系统。

## Day6 目标

把 Day5 的工具体系真正接进 Agent 控制流，做出一个最小可用的 **ReAct 闭环**：

> **LLM 规划（选择工具）→ 执行工具 → Observation 回灌 → LLM 输出最终答案**

并且全程 **可观测（SSE 事件流）+ 可取消（TaskManager token）**。

---

## 你完成的核心功能

### 1) ReAct Loop（多步控制流）

- 引入 `max_steps`（例如 6）防止模型死循环
    
- 每一步都有：
    
    - `react_step_start`
        
    - `react_model_raw`（模型原始输出）
        
    - （可能）工具调用与 observation
        
    - 直到 `final`
        

你跑通的例子非常典型：

- Step1：模型输出 tool call JSON → 选择 `calc`
    
- Step2：模型输出 final JSON → 给出答案
    

---

### 2) 动作协议（JSON action format）

你没有直接用 Gemini 的原生 tool-calling，而是先用最稳定、可移植的 **JSON 协议**：

- 工具调用：
    
    `{"type":"tool","tool_name":"calc","args":{"expression":"(19.5 + 2.3) * 4"}}`
    
- 最终回答：
    
    `{"type":"final","final":"87.2"}`
    

优点：

- 任何模型都能用同一套 loop
    
- 易调试：解析失败可以直接看 `react_model_raw`
    

---

### 3) 工具执行真正进入 agent loop

当模型选择工具后，你把它交给 Day5 的 `ToolRunner` 执行：

- `tool_start`
    
- `tool_end` / `tool_error`
    
- 结果变成 Observation，再喂回 LLM
    

你 SSE 中的关键证据：

- `tool_start` → `tool_end`
    
- `react_observation`（里面带 `value: 87.2`）
    

---

### 4) 观测性（Observability）完整闭环

你通过 EventBus + SSE，把整个 ReAct 过程“直播”出来：

- 模型的决策（`react_model_raw`）
    
- 工具的开始/结束（`tool_start/tool_end`）
    
- 回灌内容（`react_observation`）
    
- 最终输出（`final`）
    

这为后续 Studio/调试/评估奠定了基础。

---

### 5) 可中断（Cancellation / Steering 基础）

你在循环关键点放了：

- `await token.checkpoint()`
    

意味着：

- 用户调用 `/cancel` 能在下一次 checkpoint 处中断 ReAct（包括工具执行前/下一步推理前）。
    

---

## Day6 你踩到的关键坑与修复思路

### 1) “为什么没看到模型输出？”

你看到的模型输出主要以两种事件体现：

- `react_model_raw`：模型原始 JSON 输出
    
- `final`：最终答案
    

因为 Day6 用的是 `llm.generate()`（一次性生成），还没有做 Day4 那种 `llm_delta` 流式输出。

---

### 2) “为什么我想调用 sleep，却调用了 calc？”

你定位到了根因之一：**输入在客户端就乱码了**，导致模型无法理解“等待”意图。

- Windows PowerShell 里 `Invoke-RestMethod -Body` 可能用非 UTF-8 编码发出字符串
    
- FastAPI 按 UTF-8 解码 JSON → 服务端收到的是乱码（你在 `react_user_input` 看到 `???`）
    

解决方法（你已经掌握方向）：

- 用 UTF-8 字节发送 JSON（最稳）
    
- 或调整终端/脚本编码，确保请求体是 UTF-8
    

---

## 你 Day6 最重要的“能力增长”

- 你从“能流式输出”升级为“能做 agent 控制流”：  
    **模型不只是回答，而是能“选择工具、执行工具、利用工具结果再回答”。**
    
- 你把工具执行从“函数调用”提升为“可治理动作”（timeout/retry/cancel/事件）。
    
- 你做出了一个可调试的 ReAct 原型：任何一步出问题都能在 SSE 里定位。