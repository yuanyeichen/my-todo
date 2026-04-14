# 📝 提醒事项项目 — 问题总结与知识点笔记

> 适合新手小白的前端入门整理
> 项目地址：https://github.com/yuanyeichen/my-todo

---

## 一、项目开发中遇到的问题 & 解决方法

### 1. 任务添加后计数增加了，但列表里看不到
**现象**：侧边栏显示有 5 个任务，但中间列表是空的。

**原因有两个**：
- **搜索框残留文字**：如果之前搜索过"吃饭"，新添加的"睡觉"不匹配，就会被过滤掉
- **当前不在"全部"视图**：比如在"重要"或"今天"分类里添加普通任务，这个分类本来就不会显示它

**解决方法**：
在"添加任务"的代码最后，自动做两件事：
1. 清空搜索框 `searchInput.value = ''`
2. 切回"全部"视图并重新渲染列表

```javascript
// 添加任务后，确保用户能看到新任务
document.getElementById('search').value = '';
searchQuery = '';
currentFilter = 'all';  // 切回全部
renderList();           // 重新渲染
```

---

### 2. 上传到 GitHub 时连接失败
**现象**：`git push` 时报错 `Failed to connect to github.com port 443`

**原因**：
- GitHub CLI 的 Token 权限不够（缺少 `repo` 和 `read:org` 权限）
- 本地网络到 GitHub 的 HTTPS 连接偶发不稳定

**解决过程**：
1. 安装 `gh`（GitHub 官方命令行工具）
2. 用 Personal Access Token 登录：`echo "token" | gh auth login --with-token`
3. 确保 Token 勾选了 `repo`（管理仓库）和 `read:org`（读取组织）权限
4. 网络问题通过多次重试解决，最终成功推送

**学到的小技巧**：
- 如果 `git push` 被拒绝说 "remote contains work"，先执行 `git pull origin main --rebase` 拉取远程更新，再 `git push`

---

### 3. 深色模式切换后刷新页面又变回去了
**现象**：点了切换深色，F5 刷新后恢复成浅色

**原因**：只改了 CSS 类名，没有把用户的选择记录下来

**解决方法**：
使用 `localStorage` 记住用户的选择：
```javascript
// 切换时保存
localStorage.setItem('theme', 'dark');

// 页面加载时读取
const savedTheme = localStorage.getItem('theme');
if (savedTheme === 'dark') applyTheme(true);
```

---

### 4. 拖拽排序后刷新页面，顺序又乱了
**现象**：拖好的任务顺序，关闭浏览器再打开就恢复了默认排序

**原因**：拖拽只改了页面显示，没有真正修改数组里任务的存储顺序

**解决方法**：
在 `drop` 事件里，用 `splice` 真正调整 `tasks` 数组的顺序，然后 `saveTasks()`：
```javascript
const [moved] = tasks.splice(fromIndex, 1);
tasks.splice(toIndex, 0, moved);
saveTasks();
```

---

## 二、项目涉及的核心知识点（小白版）

### 📘 1. HTML — 网页的骨架
**本项目学到的常用标签**：
- `<div>`：最常用的盒子，用来装内容
- `<input>`：输入框（文字、日期、文件上传都用它）
- `<button>`：按钮
- `<ul>` / `<li>`：无序列表，本项目的任务列表就是用它做的
- `<aside>` / `<main>`：语义化标签，侧边栏和主内容区

**重要属性**：
- `id`：唯一标识符，JS 通过它找到元素
- `class`：样式类名，一个元素可以有多个 class
- `draggable="true"`：让元素可以被拖拽

---

### 🎨 2. CSS — 网页的皮肤和布局
**本项目重点学到的内容**：

#### 2.1 盒模型（Box Model）
每个元素都是一个盒子，由四部分组成：
- `content`（内容）
- `padding`（内边距）
- `border`（边框）
- `margin`（外边距）

**小技巧**：我们在代码开头写了 `* { box-sizing: border-box; }`，这样设置 `width: 100px` 时，就包含了边框和内边距，不会出现"宽高不对"的困惑。

#### 2.2 Flex 布局（最常用！）
本项目几乎全靠 Flex 实现排版：
```css
.container {
    display: flex;
    justify-content: center;  /* 水平居中 */
    align-items: center;      /* 垂直居中 */
    gap: 10px;                /* 子元素间距 */
}
```

#### 2.3 颜色与背景
- `background: linear-gradient(...)`：渐变背景（本项目的紫色渐变）
- `backdrop-filter: blur(20px)`：**毛玻璃效果**，让背后的内容模糊
- `border-radius`：圆角，现代 UI 设计的基础
- `box-shadow`：阴影，增加层次感

#### 2.4 动画
- `@keyframes`：定义动画的关键帧
- `animation: slideIn 0.25s ease`：让新任务从下方滑入
- `transition`：过渡效果，比如鼠标 hover 时颜色渐变

#### 2.5 响应式
```css
@media (max-width: 820px) {
    .sidebar { display: none; }
}
```
意思是：当屏幕宽度小于 820px 时，隐藏侧边栏（适配手机）。

---

### ⚡ 3. JavaScript — 网页的大脑
**本项目学到的 JS 核心知识**：

#### 3.1 获取和修改网页元素
```javascript
// 获取元素
document.getElementById('btn')
document.querySelector('.task-item')
document.querySelectorAll('.nav-item')

// 修改内容
element.textContent = '新文字'
element.innerHTML = '<b>加粗</b>'
element.style.color = 'red'
element.classList.add('active')
element.classList.remove('active')
```

#### 3.2 事件监听（用户交互的核心）
```javascript
// 点击按钮时执行代码
button.addEventListener('click', function() {
    alert('被点击了！')
})

// 输入框内容变化时
input.addEventListener('input', function(e) {
    console.log(e.target.value)  // 获取当前输入的文字
})

// 键盘按下时
document.addEventListener('keydown', function(e) {
    if (e.key === 'Enter') console.log('按了回车')
})
```

#### 3.3 数组操作（任务列表的增删改查）
```javascript
let tasks = []

// 添加
tasks.push({ text: '吃饭', completed: false })

// 删除（从第0位删1个）
tasks.splice(0, 1)

// 遍历
 tasks.forEach(task => {
    console.log(task.text)
})

// 过滤
const undone = tasks.filter(t => !t.completed)

// 排序
 tasks.sort((a, b) => a.completed - b.completed)
```

#### 3.4 localStorage — 浏览器本地存储
**什么是 localStorage？**
就像浏览器里有一个小数据库，关闭网页、重启电脑后数据还在。但只存当前设备上，换电脑不会同步。

```javascript
// 保存（必须是字符串）
localStorage.setItem('tasks', JSON.stringify(tasks))

// 读取
const data = localStorage.getItem('tasks')
 tasks = JSON.parse(data) || []
```

**为什么需要 `JSON.stringify`？**
因为 localStorage 只能存文字，对象和数组要先变成 JSON 字符串才能存进去，取出来再用 `JSON.parse` 变回来。

#### 3.5 日期处理
```javascript
const now = new Date()
const isoString = new Date().toISOString()  // 2026-04-14T10:00:00.000Z
```

---

### 🌲 4. Git & GitHub — 代码版本管理
**本项目用到的基础命令**：

| 命令 | 作用 |
|------|------|
| `git init` | 在当前文件夹创建 Git 仓库 |
| `git add 文件名` | 把文件加入"待提交区" |
| `git commit -m "描述"` | 保存一个版本快照 |
| `git push origin main` | 把本地代码推到 GitHub |
| `git pull origin main --rebase` | 拉取远程最新代码并合并 |

**核心概念**：
- **仓库（Repository）**：代码的家
- **提交（Commit）**：保存当前代码的一个快照，可以随时回退
- **分支（Branch）**：本项目的 `main` 就是主分支
- **远程（Remote）**：GitHub 上的仓库地址

---

### 📱 5. PWA — 让网页像 App 一样运行
**什么是 PWA？**
Progressive Web App，渐进式网页应用。可以让网页：
- 添加到手机/电脑桌面
- 离线使用
- 全屏运行，没有浏览器地址栏

**本项目做了什么？**
1. `manifest.json`：App 的配置文件（名字、图标、主题色）
2. `sw.js`（Service Worker）：一个后台脚本，拦截网络请求，缓存网页文件，实现离线访问
3. `index.html` 里注册 Service Worker：
```javascript
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('sw.js')
}
```

---

### 🧠 6. 前端开发思维（进阶一点）

#### 6.1 数据驱动视图
本项目的核心思想：**先改数据（`tasks` 数组），再重新渲染页面（`renderList()`）**

不要直接操作 DOM 去改文字、删元素，而是：
1. 修改 `tasks` 数组
2. 调用 `renderList()` 根据最新数据重新生成 HTML

这样做的好处是：数据（JS）和视图（HTML）永远保持一致，不容易出 bug。

#### 6.2 状态管理
项目中有几个"状态"：
- `tasks`：任务数据
- `currentFilter`：当前选中的分类（全部/今天/重要...）
- `searchQuery`：搜索关键词
- `selectedIds`：批量操作选中的任务 ID

状态一变，就重新渲染。这就是现代前端框架（如 React、Vue）的核心思想。

#### 6.3 用户体验（UX）细节
- **空状态**：没有任务时显示图标和提示，而不是空白
- **Loading / 动画**：任务添加时有滑入动画，完成时有撒花
- **撤销机制**：删除后给 4 秒后悔时间
- **快捷键**：`Cmd+K` 搜索、`N` 新建，提升效率

---

## 三、给新手的建议

1. **不要急着学框架**：先把 HTML + CSS + JS 基础打牢，本项目就是纯原生技术栈
2. **多写 `console.log()`**：调试时打印变量值，是最简单有效的方法
3. **从单文件开始**：像本项目一样，一个 `index.html` 就能做出完整应用，降低入门门槛
4. **善用浏览器开发者工具**：F12 打开，可以查看元素、调试 JS、看 localStorage
5. **Git 尽早学**：哪怕是小项目，养成"修改 → add → commit"的习惯，防止代码丢失

---

## 四、如何继续扩展这个项目

如果你想继续练手，可以尝试：
- 🗂️ **自定义列表**：让用户自己创建"工作""学习"等分组
- 🔔 **浏览器通知**：到截止日期时弹出系统通知
- 🎤 **语音输入**：用 Web Speech API 说话添加任务
- 📊 **数据统计**：顶部显示本周完成率图表

加油！你已经从零做出了一个媲美原生 App 的待办工具！🎉
