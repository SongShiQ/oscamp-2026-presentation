    Repo 与 PR 清单

> 本清单提供仓库整理状态、PR 汇总表、分支拓扑说明、可复现演示,确保每个结论可追溯到证据。
> 上游:rcore-os/tgoskits | fork:SongShiQ/tgoskits | 作者:SongShiQ

---

## 一、PR 汇总表(12 个)

| PR    | 标题                                                  | 状态   | 主题     | 体量                           | review | 链接                                                      |
| ----- | ----------------------------------------------------- | ------ | -------- | ------------------------------ | ------ | --------------------------------------------------------- |
| #656  | fix: dup/fcntl/flock syscall 兼容性                   | merged | syscall  | +671/-49, 12 文件              | 4      | [/pull/656](https://github.com/rcore-os/tgoskits/pull/656)   |
| #778  | dup2/close_range/ioctl 测例重构                       | merged | 测试     | +2347/-419, 31 文件            | 4      | [/pull/778](https://github.com/rcore-os/tgoskits/pull/778)   |
| #895  | sqlite3 CLI 多架构压力测试                            | merged | app/测试 | +592/-0, 16 文件               | 10     | [/pull/895](https://github.com/rcore-os/tgoskits/pull/895)   |
| #993  | busybox 7 高副作用 applet 安全测试                    | merged | 测试     | +103/-2, 4 文件                | 3      | [/pull/993](https://github.com/rcore-os/tgoskits/pull/993)   |
| #1006 | llama.cpp Alpine/musl 兼容迁移                        | merged | app      | +240/-2, 12 文件               | 2      | [/pull/1006](https://github.com/rcore-os/tgoskits/pull/1006) |
| #1033 | fix(kernel): riscv64 static-pie ELF loader 段错误     | merged | OS bug   | +402/-0, 6 文件                | 4      | [/pull/1033](https://github.com/rcore-os/tgoskits/pull/1033) |
| #1041 | dynamic musl smoke test                               | merged | 测试     | +309/-0, 10 文件               | 7      | [/pull/1041](https://github.com/rcore-os/tgoskits/pull/1041) |
| #1048 | glibc 动态链接测试(3 架构)                            | merged | 测试     | +437/-1, 16 文件               | 4      | [/pull/1048](https://github.com/rcore-os/tgoskits/pull/1048) |
| #1107 | cgroup v2 pids+cpu 基础设施                           | CLOSED | cgroup   | —                             | 1      | [/pull/1107](https://github.com/rcore-os/tgoskits/pull/1107) |
| #1156 | cgroup pids/cpu + bandwidth throttling                | CLOSED | cgroup   | —                             | 10     | [/pull/1156](https://github.com/rcore-os/tgoskits/pull/1156) |
| #1234 | feat(ax-cgroup): 抽取 cgroup v2 为独立 crate          | OPEN   | cgroup   | —                             | 38+    | [/pull/1234](https://github.com/rcore-os/tgoskits/pull/1234) |
| #1379 | feat(ax-cgroup): cgroup v2 L4 资源管控 + manager 接口 | Draft  | cgroup   | +1528/-72, 28 文件(本周纯增量) | —     | [/pull/1379](https://github.com/rcore-os/tgoskits/pull/1379) |

**量化总览(merged 8 个合计)**:+5101/-473,107 文件,59 commits,38 reviews;CI 181✓ + 193 跳过(架构无关)。

---

## 二、每个 PR 的"背景 / 修改清单 / 测试 / 已知问题"

### #656 — syscall 兼容

- **背景**:test_dup_v2.c 78 断言中 5 个 FAIL,作为"读 syscall→定位 bug→修内核"的最小起点。
- **修改清单**:fd_ops.rs(fcntl EINVAL + dup_fd_min)、file/fs.rs(append AtomicBool)、lock.rs 接入、cmake-toolchain 交叉编译修复、测例 526 行。
- **测试**:build ✓ / Linux 基线 106 断言 100% PASS / StarryOS aarch64 75/78。
- **已知问题**:3 个 ax-fs O_APPEND 设计层限制,不在本 PR 范围。

### #778 — 测例重构

- **背景**:800 行单文件难维护,新 syscall 无测例。
- **修改清单**:test_dup_v2.c 拆 26 parts、新建 dup2/close_range/ioctl 测例、升级 test_framework.h(CHECK_RET/CHECK_ERR_SAVED)。
- **测试**:build ✓ / Linux 基线 202 PASS / 0 FAIL / 8 observe / 1 skip。
- **已知问题**:StarryOS QEMU 因 tokio pipe 已知问题阻塞(非测试逻辑)。

### #895 — sqlite3

- **背景**:方案二起点,用真实应用反推 OS 缺陷。
- **修改清单**:sqlite3-smoke(S0-S4)+ sqlite3-deep(S5-S8),各 4 架构 toml,共 16 配置;loongarch64 timeout=600s。
- **测试**:build ✓ / 四架构 smoke+deep 全 PASS / CI ✓。
- **已知问题**:S6 只验 WAL 持久化,不含并发锁冲突(留未来)。

### #993 — busybox

- **背景**:测 7 个高副作用 applet 的错误处理路径。
- **修改清单**:busybox-tests.sh 加 7 case、util-linux timeout 调高 + section timing。
- **测试**:build ✓ / 7 applet 安全失败 PASS / CI ✓。
- **已知问题**:crontab/crond daemon 在 QEMU hang,留 follow-up。

### #1006 — llama.cpp

- **背景**:验证 StarryOS 跑大模型推理的兼容性。
- **修改清单**:llama-cpp 构建脚本 + 3 架构 toml + L0-L4 测试脚本;cmake CROSSCOMPILING、GGML_RVV=OFF、x86 -cpu max。
- **测试**:build ✓ / 3 架构 L0-L4 通过,L5 17/18;性能基线 tok/s 记录。
- **已知问题**:loongarch64 工具链 404、riscv64 static-pie 段错误(后由 #1033 修复)、mmap 未验证。

### #1033 — ELF loader 修复

- **背景**:#1006 暴露的 riscv64 static-pie 段错误根因。
- **修改清单**:elf_loader/mod.rs(vaddr_to_file_offset、populate_area、relocation 处理、跳过未定义符号/COPY)、static-pie-test。
- **测试**:build ✓ / static-pie-test PASS + busybox 回归全 PASS(确认不影响非 PIE)。
- **已知问题**:其他架构 ELF loader 类似问题待排查。

### #1041 — musl 动态链接

- **背景**:验证 INTERP 段加载 + NEED 依赖解析。
- **修改清单**:dynamic-musl-test + prebuild.sh(readelf 提取 INTERP 装 ld-musl)+ 3 架构 toml;riscv64 --strip-debug、x86 -cpu max。
- **测试**:build ✓ / 3 架构输出 "dynamic musl test OK" RC=0。
- **已知问题**:无重大,loongarch 待工具链。

### #1048 — glibc 动态链接

- **背景**:比 musl 更严苛(NPTL pthread、locale regex)。
- **修改清单**:4 子测试(glibc-smoke/proc-self-exe/pthread/regex)+ 3 架构 + Debian 变体。
- **测试**:build ✓ / 4 子测试通过 / "ALL TESTS COMPLETED"。
- **已知问题**:loongarch 待工具链。

### #1234 — ax-cgroup 独立 crate(进行中)

- **背景**:cgroup 模块化最终方向,工厂注册表统一框架。
- **修改清单**:ax-cgroup crate(core/controller/provider + 5 控制器)、cgroupfs VFS、内核接线。
- **测试**:host 单测通过;fmt/clippy 零警告。
- **状态**:OPEN,CHANGES_REQUESTED,review 打磨中。
- **本轮工作(T2)**:逐条修复 review + 七维度全面逻辑审核 + 本地 commit(不 push)。

### #1379 — L4 资源管控(Draft)

- **背景**:在 #1234 框架上推进 L4 + manager 接口,独立 Draft 固定成果。
- **修改清单**:10 commits(chk0~chk8)——host 测试地基、5 控制器 round-trip、io.max 持久化、cpu.weight→nice、memory 层次扣费、pids.events+delegation、cpuset effective 求交、cpu.max 限流基础设施、systemd/docker manager 接口。
- **测试**:host 8 模块 41 函数(本轮修复 delegation 失败测试后全绿)、fmt/clippy 零警告、aarch64 整树构建、QEMU 启动序列。
- **状态**:Draft,base=dev,等 #1234 合入后 rebase 转正式。
- **本轮工作(T3)**:修失败测试 + 分支同步核查 + cgroup.events/inotify 核验 + 全量验证 + 本地 commit(不 push)。

---

## 三、分支拓扑说明

```
origin/dev
   ├── (merged PRs: #656/#778/#895/#993/#1006/#1033/#1041/#1048)
   ├── pr/cgroup-modularize        ← #1234 OPEN(框架,工厂注册表)
   └── feat/cgroup-unified-rebased
          └── goal/cgroup-next-build ← #1379 Draft(L4 + manager,#1234 的超集探索)
```

- `pr/cgroup-modularize`(#1234)与 `goal/cgroup-next-build`(#1379)是**平行探索线,不是 stack**。
- #1379 选 base=dev 的独立 Draft 而非 Stacked PR:因为两分支基线 diff 达 620 文件,指向 #1234 分支会让 diff 更混乱。
- #1379 计划:等 #1234 合入 dev → rebase 到最新 dev(diff 收窄到接近纯 28 文件)→ Ready for review 转正式。

---

## 四、仓库整理清单(评审检查项)

| 检查项                         | 状态         | 说明                                                                   |
| ------------------------------ | ------------ | ---------------------------------------------------------------------- |
| README 说明目标/构建/运行/测试 | ✓           | ax-cgroup 含中英文 README + Asterinas 对比表 + systemd/Docker 兼容矩阵 |
| 分支清晰                       | ✓           | 已 merge 走 dev;cgroup 两条活跃分支拓扑明确(见上)                      |
| 关键 commit 可读               | ✓           | Conventional Commits;cgroup 用 chk0~chk8 里程碑标记                    |
| 调试输出/临时代码清理          | T2/T3 验收项 | 整理工作区时删除临时文件、调试输出、未用导入                           |
| "我的工作"小节                 | ✓           | 本清单 + 总结汇报稿即为此用                                            |
| commit 无 AI 署名              | ✓(红线)     | 仅署名 SongShiQ,禁止任何 AI 标记                                       |

---

## 五、可复现演示

**`demo_pids.sh` + `DEMO.md`**(存于 goal 分支 `cgroup演示文档/`):在真实 cgroup v2 上跑通 pids 完整路径——
检测环境 → Registry 注册 → 委托子树创建 cgroup → 写 cgroup.procs → 设 pids.max=10 →
第 11 个 fork 被内核真实拒绝(捕获非零退出码 + pids.events 自增作为不可伪造证据)→ 读回 stat → 自动清理。
严守"只在 delegated subtree 内操作"的硬不变量。

这是"L4 真实管控"中 pids 唯一完全闭环的可视化证据。

---

## 六、诚实标注的未验证项

- cgroup 端到端 QEMU 测试代码就绪,四架构未全绿。
- cpu.max:扣费+节流标志生效,调度器 pick_next 跳过未接(准闭环)。
- memory charge 接页分配器、io 接块层 QoS:有接入点,未接通。
- loongarch64:llama.cpp/glibc-dynamic 因工具链问题未覆盖。
- cgroup.events/inotify 端到端:T3 核验后据实标注完成度。
