# 卡点 - 对话式问题引导工具

## 项目概述

"卡点"是一个基于Web的对话式问题引导工具，帮助用户通过与AI的互动逐步理清思路，从混乱中找到解决问题的方向。工具采用简洁的界面设计，提供智能的问题引导和个性化设置功能。

## 功能特性

### 1. 对话式问题引导
- 用户可以描述自己遇到的"卡点"问题
- AI根据用户输入生成引导性问题，帮助用户深入思考
- 支持实时对话，无需页面刷新

### 2. Prompt自定义功能
- 提供"修改Prompt"按钮，可自定义AI的角色设定和引导规则
- 通过模态框界面进行修改，操作简便
- 修改完成后自动发送确认消息

### 3. 语音朗读功能
- 集成浏览器原生Web Speech API，纯前端实现
- 支持多种中文语音选择
- 可调整语速(0.5-2.0倍)和语调(0-2.0)
- 问题生成后自动朗读，提升用户体验
- 提供静音/播放切换按钮

### 4. 响应式设计
- 界面简洁美观，适配不同屏幕尺寸
- 支持移动端访问

## 技术实现

### 1. 核心技术栈
- **HTML5**: 页面结构和语义化标签
- **CSS3**: 样式设计和动画效果
- **JavaScript(ES6+)**: 交互逻辑和API调用

### 2. 架构特点
- **单页面应用(SPA)**: 所有功能集成在一个HTML文件中
- **模块化设计**: 功能模块清晰分离
- **事件驱动**: 基于浏览器事件模型实现交互

### 3. 关键功能实现

#### API集成
```javascript
/**
 * 调用Kimi API获取AI回复
 * @returns {Promise<string>} API返回的响应内容
 */
async function callKimiAPI() {
  // API调用逻辑
  const response = await fetch(API_URL, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${API_KEY}`
    },
    body: JSON.stringify({
      model: 'moonshot-v1-8k',
      messages: conversationHistory,
      temperature: 0.7
    })
  });
  // 处理响应
  if (!response.ok) throw new Error(`API错误: ${response.status}`);
  const data = await response.json();
  return data.choices[0].message.content;
}
```

#### 语音合成
```javascript
/**
 * 语音朗读函数
 * @param {string} text 要朗读的文本
 */
function speakText(text) {
  if (isMuted || !text) return;
  if (currentUtterance) synth.cancel();
  
  currentUtterance = new SpeechSynthesisUtterance(text);
  
  // 设置语音参数
  const selectedVoice = voices.find(voice => voice.name === voiceSelect.value);
  if (selectedVoice) currentUtterance.voice = selectedVoice;
  
  currentUtterance.rate = parseFloat(rateSlider.value);
  currentUtterance.pitch = parseFloat(pitchSlider.value);
  
  synth.speak(currentUtterance);
}
```

#### 模态框实现
```javascript
/**
 * 打开修改Prompt的模态框
 */
function openPromptModal() {
  promptInput.value = SYSTEM_PROMPT;
  promptModal.style.display = 'block';
  promptInput.focus();
}

/**
 * 保存修改后的Prompt
 */
function savePrompt() {
  const newPrompt = promptInput.value.trim();
  if (newPrompt) {
    SYSTEM_PROMPT = newPrompt;
    closePromptModal();
    addMessageToChat('已修改你想要的prompt，快来和我聊聊吧', 'assistant');
  }
}
```

## 使用方法

### 1. 启动项目
1. 在项目目录下启动本地服务器：
   ```bash
   python -m http.server 8000
   ```
2. 在浏览器中访问：`http://localhost:8000/卡点.html`

### 2. 基本操作
1. 在输入框中描述你遇到的问题
2. 点击"发送"按钮或按Enter键
3. 等待AI生成引导性问题
4. 根据AI的问题继续思考并回复

### 3. 自定义设置
1. **修改Prompt**：
   - 点击页面顶部的"修改Prompt"按钮
   - 在模态框中编辑AI的角色设定和引导规则
   - 点击"保存"按钮

2. **语音设置**：
   - 在"修改Prompt"模态框中切换到"语音设置"标签
   - 选择喜欢的语音
   - 调整语速和语调

3. **静音控制**：
   - 点击页面顶部的声音图标(🔊/🔇)切换静音状态

## 配置说明

### API密钥配置
在代码中找到以下部分，替换为你的Kimi API密钥：

```javascript
// Kimi API配置
const API_KEY = 'your_api_key_here';
const API_URL = 'https://api.moonshot.cn/v1/chat/completions';
```

### 默认系统Prompt
系统默认使用以下Prompt引导AI行为：

```javascript
let SYSTEM_PROMPT = `角色设定：
你是"卡点"的引导伙伴，兼具理性分析力与感性觉察力。你像一个温和而敏锐的思考同伴，帮助用户从混乱中逐步理清思路。

核心任务：
根据用户描述的"卡住的情况"，一次只提出一个问题，引导用户逐步深入思考，最终走向可行动状态。

引导原则：
节奏渐进：每次只问一个问题，等待用户回应后再继续。
人设融合：理性面关注事实、逻辑、约束与行动可能；感性面觉察情绪、动机、假设与心理阻碍。
不代答、不建议：你只提问，不提供答案、评价或安慰。
自然承接：每个问题应基于用户之前的回答，形成连贯对话。

问题类型框架：
- 情绪／阻力觉察类
- 目标澄清类
- 现实约束类
- 假设检验类
- 行动探索类

输出格式：
每次只输出一个问题，语言简洁、中性、开放，可适时使用"接下来…"等自然承接语，不在问题前后添加解释或总结。`;
```

## 浏览器兼容性

| 浏览器 | 兼容性 | 注意事项 |
|--------|--------|----------|
| Chrome | ✅ 完全支持 | 推荐使用 |
| Firefox | ✅ 完全支持 | |
| Safari | ✅ 完全支持 | |
| Edge | ✅ 完全支持 | |
| IE | ❌ 不支持 | 不兼容 |

**语音功能**需要浏览器支持Web Speech API，以下是支持情况：
- Chrome/Edge: 完全支持
- Firefox: 部分支持
- Safari: 完全支持

## 核心代码结构

```
卡点.html
├── HTML结构
│   ├── 页面头部(Header)
│   ├── 聊天容器(Chat Container)
│   ├── 输入区域(Input Area)
│   └── 模态框(Modal)
├── CSS样式
│   ├── 全局样式
│   ├── 聊天界面样式
│   ├── 模态框样式
│   └── 语音设置样式
└── JavaScript逻辑
    ├── DOM元素获取
    ├── API配置
    ├── 事件监听器
    ├── 聊天功能
    │   ├── 发送消息
    │   ├── API调用
    │   └── 消息展示
    ├── 模态框功能
    │   ├── 标签页切换
    │   ├── Prompt修改
    │   └── 语音设置
    └── 语音功能
        ├── 语音初始化
        ├── 语音选择
        ├── 语速/语调调整
        └── 语音朗读控制
```

## 未来改进

1. **聊天历史记录**：保存用户对话历史
2. **多语言支持**：添加英文等其他语言选项
3. **主题切换**：支持明暗主题切换
4. **导出功能**：导出对话内容为文本
5. **个性化配置**：保存用户的设置偏好
6. **增强的语音功能**：支持更多语音效果和自定义

