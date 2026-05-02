# delta-t-display-hook

注意：除了这一段，下文均由 agent 编写，可能不准确。若你想参考本项目：

- 先准备 Frida 环境：root 后使用 `frida-server`，未 root 可使用 `frida-gadget`。
- 本项目里的 RVA 是针对 3.14.2 写死的；你需要在 IDA 中为你的版本找到对应函数的 RVA 和逻辑，并在项目的基础上做一定程度的修改。
- 最后注入js需要 `frida-compile`，下文有更清楚的说明。
- 不保证按照`README.md`的指引，该项目能运行起来，只作学习参考用途。
##
Frida IL2CPP hook script — 在音游中实时显示 delta-t（判定偏移）、准确率和各判定计数。

## 功能

- **Delta-T 显示**: 每次判定后显示时间偏移量（+为晚判，-为早判）
- **颜色指示**: 黄色(≤80ms) → 蓝色(≤180ms) → 红色(>180ms)
- **准确率显示**: 实时计算并显示加权准确率 (Perfect=100%, Good=65%)
- **判定计数**: 显示 Perfect / Good / Bad / Miss 各项计数

## 环境要求

- **Node.js** >= 18.0.0
- **Frida** 已安装并可运行 (`pip install frida-tools`)
- 目标进程为 Unity IL2CPP 构建的游戏

## 安装

```bash
npm install
```

## 使用

### 1. 编译

```bash
npm run build
```

编译产物输出到 `dist/agent.js`。

### 2. 注入

编译后，使用 Frida 注入到目标进程：

```bash
# 本地进程
frida -l dist/agent.js --runtime qjs -f <进程名或包名>

# Android 设备
frida -l dist/agent.js --runtime qjs -f <包名> -H <设备IP>
```

### 3. 开发模式（watch）

修改代码后自动重新编译：

```bash
npm run watch
```

## 显示说明

```
◄ -12      ►        ← 早判 12ms (黄色)
      +45  ►        ← 晚判 45ms (黄色)
MISS               ← 未击中 (红色)
acc=95.23%         ← 当前准确率
P:120  G:30  B:5  M:2  ← 各判定计数
```

## 文件结构

```
delta_t_display_hook/
├── delta_t_display_bridge.ts   # 主 hook 脚本
├── package.json
├── tsconfig.json
└── README.md
```

## 依赖

- [frida-il2cpp-bridge](https://github.com/vfsfitvnm/frida-il2cpp-bridge) — IL2CPP 运行时桥接
- [frida-compile](https://github.com/nicolo-ribaudo/frida-compile) — TypeScript 编译工具
