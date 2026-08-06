

---

## Part 4: CUDA Features 速查

### 4.1 Unified Memory 速查
| API | 作用 |
|-----|------|
| `cudaMallocManaged(&ptr, size)` | 分配统一内存 |
| `cudaMemPrefetchAsync(ptr,size,dev,stream)` | 预取到指定设备 |
| `cudaMemAdvise(ptr,size,advice,dev)` | 访问提示 |

Advise: ReadMostly=多设备只读副本 / PreferredLocation=首选位置 / AccessedBy=映射不迁移
