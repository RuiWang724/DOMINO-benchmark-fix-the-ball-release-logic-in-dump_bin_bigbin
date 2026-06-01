# dump_bin_bigbin 动态评测释球改动（副本）

本目录为**修改后的副本**，未改动原仓库任何文件。

## 文件对照

| 本目录（修改后） | 原仓库（对照用，勿直接混用路径） |
|------------------|----------------------------------|
| `xiugai/DOMINO/envs/dump_bin_bigbin.py` | `/work/sme-wangr/DOMINO/envs/dump_bin_bigbin.py` |

## 无需修改的文件（仅引用）

| 文件路径 | 说明 |
|----------|------|
| `/work/sme-wangr/DOMINO/envs/_base_task.py` | `stop_dynamic_object_motion()` 基类实现（L3523 起） |
| `/work/sme-wangr/DOMINO/script/eval_policy.py` | L422–423 在 contact 时调用 `TASK_ENV.stop_dynamic_object_motion()` |
| `/work/sme-wangr/DOMINO/script/eval_policy_client.py` | L534–535，同上 |

## 相对原版的 3 处改动

1. **`load_actors`**：动态模式下增加 `self._garbage_physics_released = False`
2. **新增方法**：`release_garbage_physics()`、`stop_dynamic_object_motion()`（重写，内部 `super()` + 释球）
3. **`_play_once_dynamic`**：原 L240–247 循环改为 `self.release_garbage_physics()`

## 如何应用到 DOMINO

```bash
cp /DOMINO_fix/envs/dump_bin_bigbin.py \
   /DOMINO/envs/dump_bin_bigbin.py
```


## 改完后的行为

- **策略评测**：夹爪 contact 小桶 → `eval_policy.py` 调用 `stop_dynamic_object_motion()` → 停桶滑动并释放小球 kinematic
- **专家 `play_once`**：workflow 结束后同样调用 `release_garbage_physics()`（幂等，不重复释球）

`check_success()` 未改；成功率仍取决于桶高度与 5 球是否落入大桶高度带 `[0.13, 0.25]`。
