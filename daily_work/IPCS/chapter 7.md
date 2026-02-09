# 第七章 IPCF 共享内存布局

## 7.1 内存布局和大小 [7.1]

用户可以使用以下公式计算可共享内存的大小：

**IPCF 非托管通道内存布局和大小**

- `非托管通道总大小 (TOTAL UNMANAGE CHANNEL SIZE) = 16 + buffer_size`

**IPCF 托管通道内存布局和大小**

- `TX 环形缓冲区 (RING for TX) = 16 + (total_buffers_no + 1) * 8`
- `pool0 环形缓冲区 (RING for pool0) = 16 + (pool0.buffers_no + 1) * 8`
- `Pool0 = ring_for_pool + pool0.buf_size * num_bufs`
- `Channel0 = RING for TX + pool0 + ...`
- `托管通道 (MANAGE CHANNEL) = channel0 + ...`
