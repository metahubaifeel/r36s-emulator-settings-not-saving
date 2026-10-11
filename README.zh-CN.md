# R36S 模拟器设置保存不住？连发键关不掉？

> **一句话结论，分两部分。**
> **（1）设置：** RetroArch 的设置分成**四层**——全局、核心、内容目录、单个游戏。它们是层层叠加上去的，而且**一旦某个核心/目录/单游戏的 override 存在，RetroArch 就会拒绝从那个游戏里保存全局配置。这是上游的设计，不是 bug。** 要保存到哪一层，就用写那一层的菜单项去存。
> **（2）连发：** 按键自己连发，是 **turbo（连发）**绑定在起作用。而 turbo 在 RetroArch 里**有两个互相独立的层，部分核心还有第三个层**，并且按键映射本身存在独立的 `.rmp` remap 文件里、不属于普通配置。清掉一处，另外几处还在。

本文针对 R36S 及其 ArkOS 系固件写，但 RetroArch 那一半对任何跑 RetroArch 的掌机都适用。

给中文读者的一句对齐说明：群里说的**「全能模拟器」就是 RetroArch**。本文讲的是同一件事。

---

## 一、现象：大家原话是怎么说的

以下引文来自中文掌机交流 QQ 群，**只标群名**。原始数据集里发言人无法可靠归属到每条消息，所以不写昵称。

> `[r36s（开源掌机）交流1群]` 我搞一上午都搞不明白，他这个全能模拟器设置修改以后怎么保存的，一退出去设置又变回原来的样子。

> `[r36s（开源掌机）交流1群]` 连发键设置后，所有游戏都是连发了，请问怎样取消

> `[安伯尼克掌机交流群]` rg35xxh的A在主界面没啥问题在全能模拟器玩GBA会连发，但是我看控制里面没有设置连发什么问题

> `[安伯尼克掌机交流群]` 官方系统那个搜索游戏一直不知道怎么搜索

> `[r36s（开源掌机）交流1群]` 这个掌机一搜索游戏，返回主页，游戏就显示搜索到的几个，不能显示全部

> `[r36s（开源掌机）交流1群]` 有大神知道吗？为什么我玩这个游戏屏幕那么小😂点菜单也没有可以设置的

> `[安伯尼克掌机交流群]` 玩模拟器不设置，是不行的

第三条要再读一遍，因为整个谜题都在里面：**那位用户翻了「控制」页面，没看到连发，可按键就是在连发。** 因为它不在「控制」里。它在另外两个地方，而其中一个压根不属于配置系统。

---

## 二、为什么：RetroArch 的设置是四层，不是一个页面

RetroArch 没有「那个设置」。它有一摞纯文本配置文件，按下面的顺序加载，每一层覆盖它上面所有层：

| # | 层 | 文件（相对配置目录） | 由谁写入 |
|---|---|---|---|
| 1 | 全局 | `retroarch.cfg` | 主菜单 → 配置文件 → 保存当前配置 |
| 2 | 核心 override | `config/<核心>/<核心>.cfg` | 快捷菜单 → Overrides → 保存核心 Override |
| 3 | 内容目录 override | `config/<核心>/<内容目录>.cfg` | 快捷菜单 → Overrides → 保存内容目录 Override |
| 4 | 单个游戏 override | `config/<核心>/<游戏>.cfg` | 快捷菜单 → Overrides → 保存游戏 Override |

libretro 官方文档给出的加载顺序原话是 *"Game settings → Content directory settings → Core settings → Default settings from `retroarch.cfg`"*；那个目录就是你**设置 → 目录 → 配置**里填的那个（对应的配置键叫 `rgui_config_directory`）。

另外还有三套系统，它们是**并排**在配置层之外，而不是在配置层里面：

- **核心选项（Core options）**——全局是 `retroarch-core-options.cfg`，单个游戏是 `config/<核心>/<游戏>.opt`。官方文档特别注明：只有把「自动加载内容专属核心选项」打开，单游戏的核心选项才会自动生效。
- **按键映射（Input remaps）**——`.rmp` 文件，默认在 `config/remaps/<核心>/` 下。官方文档写得很明确：*"Input settings are handled separately with the input core and game Remap system."*（按键设置是由独立的按键 remap 系统处理的。）
- **着色器预设（Shader presets）**——自己一套文件（`.cgp` / `.glslp` / `.slangp`）。

<div align="center">
<img src="docs/01-config-layers-and-paths.png" alt="RetroArch 四层配置示意图：全局 retroarch.cfg、核心 override、内容目录 override、单游戏 override，各自的文件路径与写入菜单，并注明单游戏层优先级最高" width="100%">
</div>

### 那条没人告诉你的规则

ArkOS 项目把这件事做成了 FAQ，而答案直白得少见（原文照抄）：

> **Q. When I'm in a game, I go to the retroarch menu and make a change and when I attempt to save the current configuration, it fails to save. Why is that happening?**
>
> **A.** The most likely cause is that you have saved a core, content, or game override for the particular emulator core or game you currently have loaded. Once you have those, you will no longer be able to save a global configuration from that emulator core or game. **This is by design from libretro.** What you'll have to do is from within Emulationstation, scroll through the systems to the retroarch menu, then select either retroarch or retroarch32, then make your global changes there and save current configuration there.

大意：最可能的原因是，你当前加载的这个核心或游戏已经存过核心/内容/游戏 override 了。**一旦有了这些，你就没法再从那个核心或那个游戏里保存全局配置了——这是 libretro 的设计。** 你要做的是：在 EmulationStation 里翻到 retroarch 那一项，选 retroarch 或 retroarch32，在那里改全局设置、在那里保存当前配置。

libretro 官方文档从另一个方向说了同一件事：*"once you have created an override any future changes will need to be saved via the Quick Menu."*（一旦你创建了 override，之后所有的改动都要通过快捷菜单来保存。）

所以「改一个设置、退出、又变回原样」的那个人，通常**不是没保存成功**。他是成功地加载了一个内容写着别的东西的 override。

<div align="center">
<img src="docs/04-authoritative-quotes.png" alt="三段逐字引用：ArkOS FAQ 关于核心/内容/游戏 override 会挡住全局保存的答复；libretro override 文档描述的加载层级；以及两行 ArkOS 启动器脚本，证明配置路径是 /home/ark/.config/retroarch/" width="100%">
</div>

### R36S 这类 ArkOS 机器上，文件到底在哪

`/home/ark/.config/retroarch/` —— 这不是猜的。以下两行来自 ArkOS / dArkOS 的公开源码：

```sh
# Emulationstation/emulationstation.sh.rg351mp
isitrgui=$(cat ~/.config/retroarch/retroarch.cfg | grep 'menu_driver = "rgui"')

# dArkOS_Tools/Advanced/Reset Retroarch Settings.sh
cp -f /home/ark/.config/retroarch/retroarch.cfg.bak /home/ark/.config/retroarch/retroarch.cfg
```

⚠️ **未实测：** 这两行是我们从项目仓库里读出来的；本次我们**没有**在真机上跑过。你的固件可能不一样，而且对任何一台具体设备来说，权威答案就是**设置 → 目录 → 配置**里显示的那个路径。

---

## 三、怎么修

### 第一步 —— 动手之前，先想清楚你要哪一层

<div align="center">
<img src="docs/02-which-menu-writes-which-file.png" alt="对照表：想让设置只对某个核心 / 某个 ROM 文件夹 / 某个游戏 / 按键映射 / 单游戏核心选项生效时，分别该走哪条 RetroArch 菜单路径、最终写进哪个文件" width="100%">
</div>

问自己一句：*我是想让它到处都生效，还是只想在这一个模拟器 / 这一个文件夹 / 这一个游戏上生效？* 然后按对应那一行去做。最常见的错误就是：明明已经存在 override，却做了一个「以为是全局」的改动。

### 第二步 —— 如果「保存当前配置」看起来毫无反应

这就是上面文档里写的那种情况。按顺序来：

1. 先确认 override 真的存在：游戏运行时打开 **快捷菜单 → Overrides**。如果那里显示的是 *Delete Core Override* / *Delete Game Override*（删除……），而不是 *Save*（保存……），说明 override 已经存在。
2. 要么**删掉 override** 再保存全局，要么**去 override 所在的那一层改**——也就是用新值重新保存一次 override。
3. 如果你确实想要一个全局改动、而这个 override 因为别的原因必须留着，就走 ArkOS 答案给的那条路：退回前端，打开 **EmulationStation 菜单里的 RetroArch 那一项**，而不是在游戏里进 RetroArch。那里没有加载任何内容，因此没有 override 生效，全局保存能正常写入。

### 第三步 —— 如果退出游戏时还在，一关机就没了

- 检查 **设置 → 配置 → 退出时保存配置**（Save Configuration on Quit）。上游 RetroArch 里它**默认是开的**（`config.def.h` 里 `DEFAULT_CONFIG_SAVE_ON_EXIT true`），这也是大多数人从没遇到这个问题的原因。如果它是关的，退出时什么都不会写盘，所有改动随进程一起消失。
- 确认文件可写、而且没有被重新生成。下面是一个真实例子，来自 ArkOS/dArkOS 公开的启动器脚本——前端会在你背后改这个文件：

  ```sh
  sed -i '/input_exit_emulator_btn/s//;input_exit_emulator_btn/g' /home/ark/.config/retroarch/retroarch.cfg
  ```

  这一行会在你选择「直接启动到 RetroArch」而不是启动到前端时，把某个配置键注释掉。同一个文件里，启动器还会改 `menu_driver` 和 `xmb_menu_color_theme`。⚠️ 这是我们从仓库里读到的，**没有**在真机上复现这个操作序列。但结论与是否复现无关：**在这套固件上，`retroarch.cfg` 不完全是你的。**

### 第四步 —— 重置工具做的事，比你以为的少

ArkOS 里有一个 **Advanced → Reset Retroarch Settings**。它的提示原文是：

> *"This process will restore the default retroarch settings. Use this if you can't open the retroarch menu anymore. Be aware that any global setting changes you've changed will be reverted to the default settings as initially set from the most recent update."*

而整个脚本的正文只有一行：

```sh
cp -f /home/ark/.config/retroarch/retroarch.cfg.bak /home/ark/.config/retroarch/retroarch.cfg
```

它只还原 `retroarch.cfg`，**别的什么都不动**。你在 `config/<核心>/` 下的 override、在 `remaps/<核心>/` 下的按键映射，它一概不碰。所以「我重置了 RetroArch，结果全局设置还是存不住」有一个很直白的解释：**那个挡住全局保存的 override 还原封不动地待在那儿。** 得手动删掉它。

⚠️ **未实测** —— 这是对公开仓库里脚本的阅读，不是在设备上录得的结果。

---

## 四、连发（turbo）：那个停不下来的按钮

连发就是「按住不放，按键连续触发」。它**故意不在**「控制」页面里——也就是第一节那位用户去找的地方。要查四个地方，而且它们互相独立。

<div align="center">
<img src="docs/03-turbo-check-order.png" alt="按键自动连发的四步排查清单：设置里的全局 RetroPad Turbo 绑定、Turbo Fire 行为选项、核心自带的内置 turbo（Core Options）、以及已保存的按键 remap 文件和手柄本身的硬件连发" width="100%">
</div>

**1. 全局的 RetroPad Turbo 绑定。**
`设置 → 输入 → RetroPad Binds → 端口 1 控制`（Port 1 Controls）。按 RetroArch 官方文档的说法，*"Turbo 在这个列表的最下面"*，而且*"默认没有分配"*。一旦分配了，在 **Classic（经典）** 连发模式下，*"同时按住 turbo 键和任意其它键，那个键就会连发"*。这一句话就解释了第一节的第二条引文：一个多出来的 Turbo 绑定加上 Classic 模式，等于**每个游戏都在连发**。

**2. Turbo Fire 行为选项。**
`设置 → 输入 → Turbo Fire`，游戏运行时也可以走 `快捷菜单 → 控制 → Turbo Fire`。一共五个选项：

- **Turbo Period** —— 连发的触发频率。
- **Turbo Duty Cycle** —— 每次连发按住多久。
- **Turbo Mode** —— `Classic`、`Single Button (Toggle)`、`Single Button (Hold)`。后两种单键模式**不需要你按住 turbo 键**，就会让某一个指定按键自己连发。
- **Turbo Default Button** —— 单键模式作用的那个键。
- **Allow Turbo D-Pad Directions** —— 是否允许十字键参与连发。

如果**没按 turbo 键**按键也在连发，那你就在单键模式下，要清掉的是 **Turbo Default Button**。

**3. 核心自带的内置 turbo。** 部分核心是在核心内部自己实现连发的，Retropad 绑定会直接体现出来：

| 核心 | 内置连发绑定（RetroPad 名称，据官方文档） |
|---|---|
| mGBA（GBA） | Turbo A → **Y**，Turbo B → **X**，Turbo L → 左扳机，Turbo R → 右扳机 |
| Gambatte（GB/GBC） | Turbo A → **Y**，Turbo B → **X** |
| FCEUmm（NES） | Turbo A → **Y**，Turbo B → **X**，Turbo A+B → 右摇杆按下 |

开关在 `快捷菜单 → 核心选项 → 输入`（文档写的是「为玩家 1 和/或玩家 2 启用这些连发键」的选项），作用目标可以在 `快捷菜单 → 控制 → 端口 1 控制` 里看到。**「玩 GBA 时 A 键连发，但控制里没设连发」最可能就是这一条**：连发的是核心的 Turbo A，绑定在 RetroPad 的 Y 位置上；而在实体按键字母跟 RetroPad 字母对不上的掌机上，Y 落到哪个物理键就取决于启动器怎么映射。

**4. 已保存的 remap，还有手柄本身。**
`<配置目录>/remaps/<核心>/*.rmp`。remap 文件会在**每次加载内容时重新应用**，这正是「在菜单里清了绑定，当场好了，进下一个游戏又回来了」的原因。去配置目录下的 `remaps/` 里找 `.rmp` 文件，删掉或重新保存。

以及，完全在软件之外的一种情况：很多便宜的 USB 手柄上有一个物理 **TURBO** 键，按下去会锁住一个指示灯并让某个键自动连发，跟任何软件设置都无关。如果你插着手柄，**先拔掉手柄再测一遍**，别先花一晚上翻菜单。

**把连发关掉的操作顺序：**

1. `快捷菜单 → 控制 → 端口 1 控制` —— 清掉 Turbo 绑定（按键对话框里未分配会显示成 `---`）。
2. `快捷菜单 → 控制 → Turbo Fire`（或 `设置 → 输入 → Turbo Fire`）—— 把 **Turbo Default Button** 设为无/关，让单键模式没有作用对象；顺便检查 Turbo Mode。
3. `快捷菜单 → 核心选项 → 输入` —— 关掉核心自带的 Turbo A / B / L / R。
4. 清掉或重新保存那个核心的 `.rmp`，否则第 1 步会在下次加载时被覆盖回去。
5. 保存：`主菜单 → 配置文件 → 保存当前配置`；如果存在 override，还得把它也重新保存一次——否则你只修好了今天这一局。

⚠️ **未实测。** 上面所有菜单路径和选项名，都取自 RetroArch 官方文档和 libretro 官方 override 文档；我们**没有**在 R36S 上跑过这套流程。「未分配」状态和「关闭连发」那项在屏幕上的确切措辞，在不同 RetroArch 版本之间会有差异。

---

## 五、「搜索游戏」和前端的索引

这些设备上被叫做「搜索」的东西有两个，而第一节里的两条抱怨都是关于第二个的。

**RetroArch 自己的搜索**是 RetroArch 菜单里的一个搜索框，用来在 RetroArch 的播放列表里找条目。它不是用来在卡上找 ROM 的，也不影响前端。

**前端的游戏列表**——EmulationStation 系——是**索引**，不是实时搜索。它会读取每个系统的文件夹并生成一份 **gamelist（游戏列表）**，然后给你看那份列表。所以「一搜索只显示几个、不能显示全部」几乎总是索引/筛选的问题，不是搜索的问题。

ArkOS 项目自己对「我把 ROM 放进对应文件夹了，但插上卡就是找不到游戏」给出的答案，是一个三条的清单（原文照抄）：

> - The roms are in the supported extension for the system per the Emulators and Ports section
> - Try pressing start then go to **UI Settings** then **visible systems** and make sure to **select all**.
> - Try pressing start then go to **Advanced settings** and be sure **parse gamelists only** is off.

把第二条和第三条当成索引的两个开关来读：

- **「visible systems → select all」（可见系统 → 全选）** 解决的是系统列表被收窄的问题——正好就是「只显示几个、不能显示全部」那条抱怨。
- **「parse gamelists only → off」（只解析 gamelist → 关）** 就是索引重建开关。开着的时候，前端只读已有的缓存 gamelist、不去重新扫描文件夹，所以新拷进去的 ROM 永远不出现。关掉它才会真正去扫。

另外有一条关于安伯尼克原厂系统家族的、来自群里的说法：

> `[安伯尼克掌机交流群]` 卡格式建议：ntfs/exFAT 每个压缩包下载后解压出来的文件夹放入卡上根目录即可在RG前端里面按X进行扫描。

「按 X 扫描」是那家前端特有的行为。⚠️ **未实测：** 我们**没有**核实安伯尼克原厂系统的搜索菜单和它的扫描键，这里只是转述群里的说法、没有实测。不同前端构建之间菜单措辞不一样，所以在非 ArkOS 固件上，请把任何确切标签都当成「去找对应的那一项」。

⚠️ **同样未实测：** RetroArch 内置搜索框的菜单路径在 `rgui` 和 `xmb` 两种菜单驱动下不一样，而 ArkOS 历史上在这两者之间切换过。本文没有核实任何一种驱动下的标签。

---

## 六、怎么验证改动真的留住了

按顺序做，在第一个失败的地方停下——它会告诉你哪一层在跟你较劲。

1. **正常退出游戏**回到前端，再进同一个游戏。如果这里就变回去了，说明是 override 在管事。
2. **彻底关机再开机**，再进那个游戏。如果这里变回去了、但第 1 步是好的，说明压根没写盘——去查「退出时保存配置」，再查文件是不是可写。
3. **用同一个核心换一个游戏加载。** 如果改动跟过去了，你存的是核心层；如果没跟过去，你存的是单游戏层。
4. **把卡上实际有什么列出来。** 用 SSH（ArkOS：*Options → Enable Remote Services*，然后用 `ark` / `ark` 登录），配置目录就是 RetroArch 在 **设置 → 目录 → 配置** 里报的那个：

   ```sh
   ls -l  <配置目录>/retroarch.cfg          # 全局配置，以及它的时间戳
   ls -R  <配置目录>/config                 # 所有核心 / 内容目录 / 单游戏 override
   ls -R  <配置目录>/remaps                 # 所有已保存的按键 remap
   grep -n turbo <配置目录>/retroarch.cfg   # 如果你在查连发，就是这些键
   ```

<div align="center">
<img src="docs/05-where-to-look.png" alt="RetroArch 配置目录结构注释图：全局配置、按核心分的 override 文件夹（含核心 / 内容目录 / 单游戏 / .opt 文件）、remaps 文件夹，以及一组连发相关的配置键名" width="100%">
</div>

上面那组连发相关的键名（`input_turbo_period`、`input_turbo_duty_cycle`、`input_turbo_mode`、`input_turbo_button`、`input_turbo_bind`、`input_turbo_enable`、`input_turbo_allow_dpad`）是从 RetroArch 自己的配置源码里读出来的，所以拿它们去 grep 是可靠的。**我们刻意不打印具体取值，因为取值因机器而异。** `ls` 的输出同理——本文**没有**在真机 R36S 上跑过这些命令，所以只给命令、不给示例输出。

**光看时间戳通常就够了。** 如果 `retroarch.cfg` 的修改时间还是你保存之前那个，那就什么都没写进去。如果 `config/<核心>/` 下某个 `.cfg` 的时间戳是今天，那它就是那个正在生效的 override。

---

## 七、怎么别再踩

- **一次只改一个，然后验证。** 存完、退出游戏、再进一次。一次改五样，你分不清是哪个层把哪个吃掉了。
- **记住 override 是黏的。** 建了一个 override，就等于悄悄关掉了那个核心或那个游戏的全局保存。没有理由就不要建。
- **改全局，优先用前端的 RetroArch 入口。** 没加载内容、没有 override 生效，保存会落到你预期的位置。
- **记住按键设置不在配置里。** 按键绑定走 `.rmp` remap，不在配置层里。按键行为怪异时，先看 remap 和核心自己的选项。
- **不要留一个全局 Turbo 绑定。** 这是让整台机器上每个游戏看起来都坏掉的最便宜的方法。
- **折腾之前先备份配置目录。** ArkOS 启动器里还有一个 **Backup dArkOS Settings**（备份 dArkOS 设置）选项，可以当起点。把 `/home/ark/.config/retroarch/` 整个拷到电脑上不花什么代价，上面所有操作就都可逆了。
- **理解这里的「根因」通常是「层选错了」，而不是「保存坏了」。** 这套设置系统完全按设计在工作，只是它的设计跟屏幕上那个菜单给人的印象不一样。

---

## 八、常见问题

**RetroArch 只有一个设置文件吗？**
不是。一个全局文件，加上三层 override；另外还有核心选项、按键 remap、着色器预设三套独立系统。

**为什么在游戏里点「保存当前配置」没反应？**
因为那个核心、内容目录或游戏已经存在 override。上游 RetroArch 在 override 生效时会**故意**拒绝全局保存。删掉 override，或者从「没加载任何内容」的前端 RetroArch 入口去改。

**我把 RetroArch 重置成默认了，还是存不住，为什么？**
因为 ArkOS 的重置工具只还原 `retroarch.cfg`。override 在另一个文件夹里，重置不会动它们。

**连发（turbo）设置在哪？**
不在「控制」里。全局绑定在 `设置 → 输入 → RetroPad Binds → 端口 1 控制`（最后一项）。行为选项在 `设置 → 输入 → Turbo Fire` 或 `快捷菜单 → 控制 → Turbo Fire`。有些核心在 `快捷菜单 → 核心选项 → 输入` 里还有自己的连发。

**为什么只有 GBA 会连发，而且只在 RetroArch 里？**
mGBA 自带 Turbo A 和 Turbo B，绑在 RetroPad 的 Y 和 X 上，由核心自己的选项启用。前端对这件事一无所知——这正是它表现为「只有在 RetroArch 里才这样」的原因。

**新拷进去的 ROM 一直不出现，怎么重建列表？**
靠前端扫描。ArkOS 上是 `Start → Advanced settings`，把 **parse gamelists only** 关掉；如果系统列表本身被收窄了，就 `Start → UI Settings → visible systems → select all`。

---

## 九、相关链接

- 我们的在售主板版本与配色：<https://r36s.space/products/r36s>
- 上手与首次开机说明：<https://r36s.space/guide>
- 常见问题：<https://r36s.space/faq>

### 本文用到的来源

- [Using Content, Folder, and Core Overrides for Custom Settings](https://github.com/libretro/docs/blob/master/docs/guides/overrides.md) —— libretro 官方文档。用于 override 的加载层级、`<配置目录>/config/<核心>/…` 的文件位置、「按键设置由独立的 remap 系统处理」这条说明，以及「创建 override 之后改动都要通过快捷菜单保存」这条注记。
- [How to Set Up a Turbo Button in RetroArch](https://docs.retroachievements.org/general/tutorials/turbo-button.html) —— 用于全局 Turbo 绑定和 Turbo Fire 的确切菜单路径、五个 Turbo Fire 选项、mGBA / Gambatte / FCEUmm 的核心内置连发绑定，以及「保存当前配置」「退出时保存配置」的路径。
- [ArkOS — Frequently Asked Questions](https://github.com/christianhaitian/arkos/wiki/Frequently-Asked-Questions---RG351MP) —— 「保存失败」那条答案、ROM 不出现的三条清单、以及 ArkOS 的 SSH 账号与 `Enable Remote Services` 步骤，均为逐字引用。
- [dArkOS](https://github.com/christianhaitian/dArkOS) 与 [(d)ArkOS4Clone](https://github.com/lcdyk0517/arkos4clone) —— 公开仓库。用于 `/home/ark/.config/retroarch/` 这个配置路径、`Reset Retroarch Settings` 脚本正文、启动器对 `retroarch.cfg` 的 `sed` 改动，以及启动器菜单里 `Backup dArkOS Settings` 这一项的存在。
- RetroArch 源码 —— `configuration.c` 里的 `rgui_config_directory`、`config_save_on_exit` 以及 `input_turbo_*` 配置键，`config.def.h` 里的 `DEFAULT_CONFIG_SAVE_ON_EXIT true` 默认值。

**发现有错或想补充，欢迎提 issue 或 PR** —— 尤其是能在当前 ArkOS 版本上确认确切菜单措辞的人。
