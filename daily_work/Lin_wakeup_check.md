# 1. wakeupCheck flow
#### *1. controller监测到wakeup事件，发起中断，confirmWakeup；* 中断函数置位flag；*
#### *2. 等待EcuM管理模块执行polling查询，该flag读取一次就会被清理，避免重复触发*

