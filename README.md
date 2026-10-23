# steamtools-sigs — OpenSteamTool 签名文件（自维护）

本仓库存放 Steam 客户端签名文件（pattern / IPC TOML），供
[steamtools 本地管理器项目](https://github.com/nichem/steamtools) 的 OpenSteamTool 环境
按当前 DLL 哈希自动拉取。布局与获取地址一一对应（`main` 分支）：

```text
signatures/{pattern|ipc}/{steamclient|steamui}/<steamclient64.dll 或 SteamUI.dll 的 SHA-256>.toml
```

获取地址形如：

```text
https://raw.githubusercontent.com/nichem/steamtools-sigs/main/signatures/pattern/steamclient/<sha256>.toml
```

OST 启动时按当前 DLL 哈希从本仓库拉取；拉不到（404）时回退到本地缓存。

## 内容分两段

1. **上游存档（136 份）**：来自
   [OpenSteam001/steam-monitor](https://github.com/OpenSteam001/steam-monitor)——
   `pattern` 分支最后发布 2026-09-06（构建 1788652215），`ipc` 分支最后发布 2026-09-26。
   该段用于旧构建回滚与历史参照，不再更新。
2. **自产段**：此后 Steam 每次更新由本机工具链重建发布，首批为 2026-10-14 构建
   （steamclient `e9045710…`、steamui `1b945794…`、ipc `e9045710…`），
   最新为 2026-10-22 构建（steamclient / ipc `253e0a11…`、steamui `cd9226db…`）。

当前合计 142 份：pattern/steamclient 46、pattern/steamui 46、ipc/steamclient 50。

## 以后怎么更新（每次 Steam 更新后）

在 steamtools 仓库里：

```powershell
python tools/regen-patterns/regen_patterns.py --cache "<管理器>\data\metadata" --install
```

把生成的三件套（`pattern/steamclient/<新sha>.toml`、`pattern/steamui/<新sha>.toml`、
`ipc/steamclient/<新sha>.toml`，位于 `<管理器>\data\metadata` 下）拷进本仓库
`signatures/` 对应目录，提交并推送到 `main` 即可。

## 回滚应急小抄

- **退回近版本（没有任何签名的构建）**：OST 弹 “Unsupported Steam Version”，
  解锁停用，**不崩**。用上面的 regen 命令对回滚后的 DLL 现场重建即可。
- **退回老版本（09-06 及更早、本仓库有存档，且使用 2026-10-18 之后的结构修复版 DLL）**：
  可能因 `CNetPacket` 结构差异在 `RecvPkt` 处启动崩溃。二选一：
  1. 删掉该构建 TOML 中的 `RecvPkt` 段（会弹 “Missing Signatures”，可忽略，其余功能正常）；
  2. 换回更新前的旧版 DLL（管理器 `data\recovery\…-update-previous` 里留有快照）。

## 说明

- 签名文件仅为地址映射表（函数名 / RVA / 字节特征），不含任何凭据。
- 本仓库与 Valve 无关，仅供本机 OpenSteamTool 环境自用与备份。
