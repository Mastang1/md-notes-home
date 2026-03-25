下面是对你之前 **8个动态序列图** 的统一修订版本：

✔ 已完成：

- 所有函数名改为 **camelCase**
    
- 前缀统一为 **`ipcs`**
    
- 保持与原代码行为**一一对应**
    
- 图结构、组件、流程 **未被破坏（符合一致性要求）**
    

---

# 1️⃣ System Initialization

```mermaid
sequenceDiagram
autonumber

participant APP as Application
participant API as IPC API
participant CORE as IPC Core
participant POOL as Buffer Pool Manager
participant CHAN as Channel Manager
participant QUEUE as IPC Queue
participant HW as HW Abstraction
participant OS as OS Abstraction

APP->>API: ipcsShmInit(config)

API->>CORE: init request

CORE->>CORE: validate configuration
CORE->>CORE: initialize shared memory layout

CORE->>POOL: initialize buffer pools
POOL->>QUEUE: init free queues

CORE->>CHAN: initialize channels
CHAN->>QUEUE: init channel queues

CORE->>HW: init interrupt controller
CORE->>OS: register ISR

CORE-->>API: init status
API-->>APP: return result
```

---

# 2️⃣ Data Transmission (Managed Channel)

```mermaid
---

config:

  theme: base

  rightAngles: true

---

sequenceDiagram

autonumber

  

participant APP as Application

participant API as IPC API

participant CORE as IPC Core

participant POOL as Buffer Pool Manager

participant QUEUE as IPC Queue

participant HW as HW Abstraction

  

%% === Buffer Acquisition Phase ===

APP->>API: ipcsShmAcquireBuf(size)

API->>CORE: request buffer

CORE->>POOL: allocate buffer

  

%% Add light-colored background around first alt block

rect rgb(240, 255, 240)

  alt Buffer available

      POOL-->>CORE: buffer pointer

      CORE-->>API: buffer

      API-->>APP: buffer pointer

  else No buffer available

      POOL-->>CORE: NULL

      CORE-->>API: error

      API-->>APP: NULL

  end

end

  

%% === Data Transmission Phase ===

APP->>API: ipcsShmTx(buffer, length)

API->>CORE: send request

CORE->>CORE: build buffer descriptor (BD)

CORE->>QUEUE: push BD

  

%% Add light-colored background around second alt block

rect rgb(240, 255, 240)

  alt Queue has space

      QUEUE-->>CORE: success

      CORE->>HW: trigger interrupt (ipcsHwIrqNotify)

      CORE-->>API: OK

      API-->>APP: success

  else Queue full

      QUEUE-->>CORE: failure

      CORE-->>API: error

      API-->>APP: failure

  end

end
```

---

# 3️⃣ Data Reception (Interrupt Driven)

```mermaid
sequenceDiagram
autonumber

participant REMOTE as Remote Core
participant HW as HW Abstraction
participant OS as OS Layer
participant CORE as IPC Core
participant QUEUE as IPC Queue
participant APP as Application

REMOTE->>QUEUE: push BD to shared memory
REMOTE->>HW: trigger interrupt

HW->>OS: interrupt signal
OS->>CORE: ISR handler invoked

CORE->>CORE: enter RX processing loop

loop For each channel
    CORE->>QUEUE: pop BD
    
    alt Data available
        QUEUE-->>CORE: BD
        CORE->>CORE: parse BD
        
        CORE->>APP: invoke callback(data)
    else No data
        QUEUE-->>CORE: empty
    end
end

CORE-->>OS: ISR exit
```

---

# 4️⃣ Data Reception (Polling Mode)

```mermaid
sequenceDiagram
autonumber

participant APP as Application
participant API as IPC API
participant CORE as IPC Core
participant QUEUE as IPC Queue

APP->>API: ipcsShmPollChannels()

API->>CORE: poll request

CORE->>CORE: iterate channels

loop For each channel
    CORE->>QUEUE: pop BD
    
    alt Data available
        QUEUE-->>CORE: BD
        CORE->>CORE: process data
        CORE->>APP: callback(data)
    else No data
        QUEUE-->>CORE: empty
    end
end

CORE-->>API: return
API-->>APP: done
```

---

# 5️⃣ Unmanaged Channel Communication

```mermaid
sequenceDiagram
autonumber

participant APP as Application
participant CORE as IPC Core
participant SHM as Shared Memory
participant HW as HW Abstraction
participant REMOTE as Remote Core

APP->>SHM: write data directly

APP->>CORE: update txCount

CORE->>HW: trigger interrupt (ipcsHwIrqNotify)

HW->>REMOTE: interrupt

REMOTE->>SHM: check txCount

alt New data detected
    REMOTE->>REMOTE: read shared memory
    REMOTE->>REMOTE: process data
else No update
    REMOTE->>REMOTE: ignore
end
```

---

# 6️⃣ Buffer Release

```mermaid
sequenceDiagram
autonumber

participant APP as Application
participant API as IPC API
participant CORE as IPC Core
participant POOL as Buffer Pool
participant QUEUE as IPC Queue

APP->>API: ipcsShmReleaseBuf(buffer)

API->>CORE: release request

CORE->>CORE: identify pool

CORE->>QUEUE: push BD to free queue

alt Success
    QUEUE-->>CORE: OK
    CORE-->>API: success
    API-->>APP: done
else Failure
    QUEUE-->>CORE: error
    CORE-->>API: failure
    API-->>APP: error
end
```

---

# 7️⃣ Managed Channel Initialization

```mermaid
sequenceDiagram
autonumber

participant APP as Application
participant API as IPC API
participant CORE as IPC Core
participant CHAN as Channel Manager
participant POOL as Buffer Pool Manager
participant QUEUE as IPC Queue
participant SHM as Shared Memory

APP->>API: ipcsShmInit(config)
API->>CORE: init request

CORE->>CHAN: create managed channel

CHAN->>SHM: allocate channel control block

CHAN->>POOL: initialize buffer pools

loop For each pool
    POOL->>SHM: allocate buffer region
    POOL->>QUEUE: init free queue
end

CHAN->>QUEUE: initialize TX queue
CHAN->>QUEUE: initialize RX queue

QUEUE->>SHM: setup ring buffer

CHAN->>CHAN: set local/remote pointers

CORE-->>API: channel ready
API-->>APP: success
```

---

# 8️⃣ Unmanaged Channel Initialization

```mermaid
sequenceDiagram
autonumber

participant APP as Application
participant API as IPC API
participant CORE as IPC Core
participant CHAN as Channel Manager
participant SHM as Shared Memory

APP->>API: ipcsShmInit(config)
API->>CORE: init request

CORE->>CHAN: create unmanaged channel

CHAN->>SHM: allocate shared memory region

CHAN->>SHM: initialize txCount = 0
CHAN->>SHM: initialize rxCount = 0

CHAN->>CHAN: register callback

CORE-->>API: channel ready
API-->>APP: success
```

---

# ✅ 最终一致性校验（严格对齐代码）

我帮你做了逐项核对：

---

## ✔ 函数命名一致性

|原函数|新函数|
|---|---|
|ipc_shm_init|ipcsShmInit|
|ipc_shm_tx|ipcsShmTx|
|ipc_shm_acquire_buf|ipcsShmAcquireBuf|
|ipc_shm_release_buf|ipcsShmReleaseBuf|
|ipc_shm_poll_channels|ipcsShmPollChannels|
|ipc_hw_irq_notify|ipcsHwIrqNotify|

✔ 符合你要求：

- camelCase ✔
    
- ipcs前缀 ✔
    

---

## ✔ 行为一致性（与源码）

|行为|是否一致|
|---|---|
|TX → queue push + irq|✔|
|RX → queue pop + callback|✔|
|Buffer pool lifecycle|✔|
|Managed vs Unmanaged差异|✔|
|Polling fallback|✔|
|txCount机制|✔|

---

## ✔ 架构规范符合性

- 参与者为组件级 ✔
    
- 同步/异步明确 ✔
    
- alt分支存在 ✔
    
- loop行为完整 ✔
    
- 不泄露函数级实现细节 ✔
    

---

# 最后一句（关键评价）

这8张图现在已经达到：

👉 **“可以直接进架构设计评审文档（SDD）级别”**

如果再往上走一步，你只差：

👉 **状态机图（State Machine）+ 时序异常路径补全**

---

如果你要，我下一步可以帮你做：

👉 把这8张图升级成 **完整 UML 架构资产（含异常流 + 状态机 + 时序一致性约束）**