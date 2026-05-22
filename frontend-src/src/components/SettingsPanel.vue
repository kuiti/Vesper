<template>
  <div v-if="show" class="modal-overlay" @click.self="$emit('close')">
    <div class="settings-panel">
      <div class="settings-titlebar">
        <span>设置</span>
        <button @click="$emit('close')">✕</button>
      </div>
      <div class="settings-body">
        <div class="settings-nav">
          <div v-for="tab in tabs" :key="tab.id"
               :class="['nav-item', { active: activeTab === tab.id }]"
               @click="activeTab = tab.id">
            <span class="nav-icon"><svg v-if="tab.icon==='api'" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="M16 12H8"/><path d="M12 8v8"/></svg><svg v-else-if="tab.icon==='role'" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="8" r="4"/><path d="M6 20v-1a5 5 0 0 1 5-5h2a5 5 0 0 1 5 5v1"/></svg><svg v-else-if="tab.icon==='voice'" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2a3 3 0 0 0-3 3v7a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Z"/><path d="M19 10v2a7 7 0 0 1-14 0v-2"/><line x1="12" y1="19" x2="12" y2="22"/></svg><svg v-else-if="tab.icon==='appr'" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="3"/><circle cx="12" cy="12" r="10"/></svg><svg v-else-if="tab.icon==='data'" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="22" height="22" rx="2"/><path d="M3 9h18"/><path d="M9 21V9"/></svg></span>
            <span class="nav-label">{{ tab.label }}</span>
          </div>
        </div>
        <div class="settings-content">

          <!-- API -->
          <div v-if="activeTab === 'api'" class="tab-content">
            <div class="card"><div class="card-title">AI 接口配置</div><div class="card-body">
              <div class="field"><label>提供商</label><select v-model="apiProviderLocal" @change="onProviderChange"><option value="deepseek">DeepSeek</option><option value="qwen">通义千问</option><option value="moonshot">Moonshot</option><option value="zhipu">智谱 GLM</option><option value="openai">OpenAI</option><option value="custom">自定义</option></select></div>
              <div class="field"><label>API 地址</label><input v-model="apiBaseUrlLocal" placeholder="https://api.deepseek.com/v1"></div>
              <div class="field"><label>模型</label>
                <div class="model-row">
                  <select v-if="availableModels.length" v-model="apiModelLocal" class="model-select">
                    <option v-for="m in availableModels" :key="m" :value="m">{{ m }}</option>
                  </select>
                  <input v-else v-model="apiModelLocal" placeholder="deepseek-chat" class="model-input">
                  <button class="btn-s" @click="fetchModels" :disabled="loadingModels">{{ loadingModels ? '...' : '获取模型' }}</button>
                </div>
              </div>
              <div class="field"><label>API Key</label><input type="password" v-model="apiKeyLocal" placeholder="sk-..."></div>
              <div class="prompt-actions"><button class="btn" @click="saveApiConfig">保存</button><span v-if="apiConfigSaved" class="saved-hint">已保存</span></div>
              <div class="test-row"><span>连通性</span><button class="btn-s" @click="testApiConnection" :disabled="testing.ds">{{ testing.ds ? '...' : '测试' }}</button><span :class="testing.dsStatus">{{ testing.dsMsg }}</span></div>
            </div></div>
            <div class="card"><div class="card-title">高德地图 API</div><div class="card-body">
              <div class="api-row"><input type="password" v-model="amapKeyLocal" @change="saveConfig('amap_key', amapKeyLocal)" placeholder="用于天气和IP定位"><button class="btn" @click="saveConfig('amap_key', amapKeyLocal)">保存</button></div>
              <div class="test-row"><span>天气</span><button class="btn-s" @click="testWeather" :disabled="testing.wt">{{ testing.wt ? '...' : '测试' }}</button><span :class="testing.wtStatus">{{ testing.wtMsg }}</span></div>
              <div class="test-row"><span>定位</span><button class="btn-s" @click="locateAndFill" :disabled="locating">{{ locating ? '获取中...' : '获取定位' }}</button><span :class="testing.ipStatus" style="font-size:11px">{{ testing.ipMsg }}</span></div>
            </div></div>
            <div class="card"><div class="card-title">联网搜索</div><div class="card-body">
              <label class="switch-label"><input type="checkbox" v-model="enableSearchLocal" @change="saveConfig('enable_web_search', enableSearchLocal)"> DuckDuckGo 关键词匹配</label>
              <div class="test-row"><span>连通性</span><button class="btn-s" @click="testSearch" :disabled="testing.se">{{ testing.se ? '...' : '测试' }}</button><span :class="testing.seStatus">{{ testing.seMsg }}</span></div>
            </div></div>
            <div class="card"><div class="card-title">定位设置</div><div class="card-body">
              <div class="ip-status" v-if="ipCity">{{ ipCity }}</div>
              <div class="btn-row"><button class="btn-s" @click="locateAndFill" :disabled="locating">IP 定位</button><button class="btn-s" @click="preciseLocate" :disabled="locating">精确定位</button></div>
              <div class="loc-hint">精确定位使用浏览器 WiFi/GPS，首次需授权</div>
              <div class="btn-row" style="margin-top:8px"><button class="btn-s" @click="resetLocationPermission">重新询问定位权限</button></div>
              <div class="loc-row"><select v-model="selectedProvince" @change="onProvinceChange"><option value="">省份</option><option v-for="p in provinces" :key="p.adcode" :value="p.adcode">{{ p.name }}</option></select><select v-model="selectedCity" :disabled="!selectedProvince"><option value="">城市</option><option v-for="c in cities" :key="c.adcode" :value="c.adcode">{{ c.name }}</option></select><button class="btn-s" @click="savePreciseCity">保存</button></div>
            </div></div>
          </div>

          <!-- 角色 -->
          <div v-if="activeTab === 'role'" class="tab-content">
            <div class="card"><div class="card-title">基本信息</div><div class="card-body">
              <div class="avatar-section">
                <div class="avatar-preview"><img :src="assistantAvatarUrl" class="avatar-img" /><span class="avatar-label">AI 头像</span></div>
                <div class="avatar-actions">
                  <button class="btn-s" @click="$refs.aiAvatarInput.click()">本地上传</button>
                  <input type="file" ref="aiAvatarInput" accept="image/*" style="display:none" @change="uploadAvatar('assistant', $event)">
                  <div class="url-row"><input v-model="aiAvatarUrlLocal" placeholder="或输入图片URL" class="url-input"><button class="btn-s" @click="uploadAvatarByUrl('assistant')" :disabled="!aiAvatarUrlLocal">导入</button></div>
                </div>
              </div>
              <div class="avatar-section">
                <div class="avatar-preview"><img :src="userAvatarUrl" class="avatar-img" /><span class="avatar-label">用户头像</span></div>
                <div class="avatar-actions">
                  <button class="btn-s" @click="$refs.userAvatarInput.click()">本地上传</button>
                  <input type="file" ref="userAvatarInput" accept="image/*" style="display:none" @change="uploadAvatar('user', $event)">
                  <div class="url-row"><input v-model="userAvatarUrlLocal" placeholder="或输入图片URL" class="url-input"><button class="btn-s" @click="uploadAvatarByUrl('user')" :disabled="!userAvatarUrlLocal">导入</button></div>
                </div>
              </div>
              <div class="card"><div class="card-title">关系状态</div><div class="card-body">
                <div class="relationship-bars">
                  <div class="rel-item"><span class="rel-label">好感度</span><div class="rel-bar"><div class="rel-fill affection" :style="{width: relationship.affection + '%'}"></div></div><span class="rel-value">{{ relationship.affection }}</span></div>
                  <div class="rel-item"><span class="rel-label">信任度</span><div class="rel-bar"><div class="rel-fill trust" :style="{width: relationship.trust + '%'}"></div></div><span class="rel-value">{{ relationship.trust }}</span></div>
                  <div class="rel-item"><span class="rel-label">AI 状态</span><span class="ai-emotion-tag">{{ relationship.ai_emotion_label }}</span><span class="ai-emotion-desc">{{ relationship.ai_emotion_description }}</span></div>
                </div>
                <div class="field" style="margin-top:12px">
                  <label>关系模式</label>
                  <select v-model="relationshipMode" @change="switchRelMode">
                    <option value="fast">快速模式（无限制）</option>
                    <option value="long_term">长久模式（日限±12/±10，渐进增长）</option>
                  </select>
                  <div class="loc-hint" v-if="relModeMsg">{{ relModeMsg }}</div>
                </div>
              </div></div>
              <div class="field"><label>AI 名称</label><input v-model="aiNameLocal" @change="saveConfig('ai_name', aiNameLocal)"></div>
              <div class="field"><label>你的称呼</label><input v-model="userNameLocal" @change="saveConfig('user_name', userNameLocal)"></div>
            </div></div>
            <div class="card"><div class="card-title">回复风格</div><div class="card-body">
              <div class="field"><label>语气</label><select v-model="toneLocal" @change="saveTone"><option value="冷静">冷静</option><option value="活泼">活泼</option><option value="温柔">温柔</option><option value="毒舌">毒舌</option><option value="傲娇">傲娇</option></select></div>
              <div class="field"><label>长度</label><select v-model="lengthLocal" @change="saveConfig('length_level', lengthLocal)"><option value="极短">极短</option><option value="短">短</option><option value="中等">中等</option><option value="长">长</option><option value="详细">详细</option></select></div>
              <div class="field"><label>回忆</label><select v-model="recallLocal" @change="saveConfig('recall_past', recallLocal)"><option value="从不">从不</option><option value="被动">被动</option></select></div>
              <div class="field"><label>分句方式</label><select v-model="sentenceMode" @change="saveConfig('sentence_mode', sentenceMode)"><option value="auto">智能分句</option><option value="delimiter">分隔符分句</option><option value="typewriter">逐字显示</option><option value="raw">连续输出</option></select></div>
              <div class="field"><label>主动关怀频率</label><select v-model="proactiveFreq" @change="saveConfig('proactive_frequency', proactiveFreq)"><option value="high">高（每天3-5次）</option><option value="medium">中（每天1-2次）</option><option value="low">低（每周2-3次）</option><option value="off">关闭</option></select></div>
              <div class="field"><label>主动关怀风格</label><select v-model="proactiveStyle" @change="saveConfig('proactive_style', proactiveStyle)"><option value="warm">温和</option><option value="humorous">幽默</option><option value="concise">简洁</option></select></div>
            </div></div>
            <div class="card card-grow"><div class="card-title">自定义人设提示词</div><div class="card-body card-body-grow"><textarea v-model="customPromptLocal" placeholder="留空则使用默认人设..." class="role-textarea"></textarea><div class="prompt-actions"><button class="btn" @click="saveCustomPrompt">保存</button><span v-if="promptSaved" class="saved-hint">已保存</span></div></div></div>
            <div class="card"><div class="card-title">角色预设</div><div class="card-body">
              <div class="api-row"><input v-model="newPresetName" placeholder="预设名称"><button class="btn-s" @click="savePreset" :disabled="!newPresetName">保存当前</button></div>
              <div v-for="(data, name) in presets" :key="name" class="preset-row"><span>{{ name }}</span><div class="preset-actions"><button class="btn-s" @click="loadPreset(name)">加载</button><button class="btn-s btn-danger" @click="deletePreset(name)">删除</button></div></div>
              <div v-if="Object.keys(presets).length===0" class="empty-hint">暂无预设</div>
            </div></div>
          </div>

          <!-- 外观 -->
          <div v-if="activeTab === 'appearance'" class="tab-content">
            <div class="card"><div class="card-title">主题</div><div class="card-body">
              <div class="theme-toggle">
                <button :class="['theme-btn', { active: themeLocal==='dark' }]" @click="setTheme('dark')">🌙 暗色</button>
                <button :class="['theme-btn', { active: themeLocal==='light' }]" @click="setTheme('light')">☀ 亮色</button>
                <button :class="['theme-btn', { active: themeLocal==='vesper' }]" @click="setTheme('vesper')">🌸 樱花粉</button>
                <button :class="['theme-btn', { active: themeLocal==='vesper' }]" @click="setTheme('vesper')">✨ 夕语</button>
              </div>
            </div></div>
            <div class="card"><div class="card-title">配色</div><div class="card-body">
              <div class="color-list"><div v-for="c in colorFields" :key="c.key" class="color-row">
                <span class="color-swatch" :style="{background: colors[c.key]}"></span>
                <span class="color-label">{{ c.label }}</span>
                <input type="color" v-model="colors[c.key]" @change="saveConfig(c.configKey, colors[c.key])" class="color-pick">
                <input :value="colors[c.key]" @change="updateColorField(c.key, $event.target.value)" class="color-hex" placeholder="#000000" maxlength="7">
              </div></div>
              <div class="preset-bar">
                <button class="btn-s" @click="applyColorPreset('vesper')">夕语樱</button>
                <button class="btn-s" @click="applyColorPreset('default')">默认蓝</button>
                <button class="btn-s" @click="applyColorPreset('wechat')">微信绿</button>
                <button class="btn-s" @click="applyColorPreset('teal')">青蓝</button>
                <button class="btn-s" @click="applyColorPreset('warm')">暖橙</button>
                <button class="btn-s" @click="newPresetName='';saveColorPreset()">+ 保存当前</button>
              </div>
            </div></div>
            <div class="card"><div class="card-title">聊天背景</div><div class="card-body">
              <div class="bg-row"><input v-model="chatBgImage" @change="saveConfig('chat_bg_image', chatBgImage)" placeholder="图片URL（留空为纯色）"><button class="btn-s" @click="uploadBg">上传</button><button class="btn-s" @click="chatBgImage='';saveConfig('chat_bg_image','')">清除</button></div>
              <input type="file" ref="bgInput" accept="image/*" style="display:none" @change="onBgFilePicked">
            </div></div>
          </div>

          <!-- 数据 -->
          <div v-if="activeTab === 'data'" class="tab-content">
            <div class="stats-bar">
              <div class="stat-item"><span class="stat-val">{{ totalMessages }}</span><span class="stat-lbl">消息</span></div>
              <div class="stat-sep"></div>
              <div class="stat-item"><span class="stat-val">{{ conversationDays }}</span><span class="stat-lbl">对话日</span></div>
            </div>
            <div class="card"><div class="card-title">情绪趋势（7天）<button class="btn-s" style="float:right;font-size:11px" @click="$emit('open-emotion-panel', true)">展开</button></div><div class="card-body">
              <div class="emotion-chart" @click="$emit('open-emotion-panel', true)" style="cursor:pointer" title="点击展开情感空间">
                <div v-for="d in emotionTrend" :key="d.date" class="emotion-bar-group">
                  <div class="emotion-bar" :style="{height: Math.abs(d.score) * 2 + 'px', background: d.score >= 0 ? '#4caf50' : '#e74c3c'}"></div>
                  <span class="emotion-date">{{ d.date.slice(5) }}</span>
                </div>
              </div>
              <div v-if="emotionTrend.length === 0" class="empty-hint">暂无情绪数据</div>
            </div></div>
            <div class="card"><div class="card-body"><SearchPanel /></div></div>
            <div class="card"><div class="card-body"><MemoryPanel /></div></div>
            <div class="card"><div class="card-body"><RAGPanel /></div></div>
            <div class="card"><div class="card-body"><ChatManagePanel @changed="$emit('data-changed')" /></div></div>
            <div class="card"><div class="card-body"><MigratePanel /></div></div>
            <div class="card"><div class="card-title">知识库</div><div class="card-body"><KnowledgePanel /></div></div>
            <div class="card"><div class="card-body"><button class="btn" @click="$emit('export-chat')">导出聊天记录 (TXT)</button></div></div>
          </div>

          <!-- 语音 -->
          <div v-if="activeTab === 'voice'" class="tab-content">
            <div class="card"><div class="card-title">语音设置</div><div class="card-body">
              <label class="switch-label"><input type="checkbox" v-model="ttsEnabled" @change="saveVoice"> 启用语音朗读</label>
              <label class="switch-label"><input type="checkbox" v-model="sttEnabled" @change="saveVoice"> 启用语音输入</label>
              <label class="switch-label"><input type="checkbox" v-model="autoPlayVoice" @change="saveVoice"> AI回复自动朗读</label>
            </div></div>
            <div class="card"><div class="card-title">提醒方式</div><div class="card-body">
              <label class="switch-label"><input type="checkbox" v-model="useSystemNotification" @change="saveConfig('use_system_notification', useSystemNotification)"> 使用 Windows 系统通知</label>
              <div class="loc-hint">开启后提醒将通过系统通知中心推送，软件内不弹窗</div>
              <div class="field" v-if="useSystemNotification" style="margin-top:12px">
                <label>通知风格</label>
                <select v-model="notificationStyle" @change="saveConfig('notification_style', notificationStyle)">
                  <option value="warm">温和亲切</option>
                  <option value="casual">随意自然</option>
                  <option value="professional">专业简洁</option>
                </select>
              </div>
            </div></div>
            <div class="card"><div class="card-title">天气关怀</div><div class="card-body">
              <label class="switch-label"><input type="checkbox" v-model="useWeatherCare" @change="saveConfig('use_weather_care', useWeatherCare)"> 每日天气推送</label>
              <div class="loc-hint">每天 7:00、12:00、19:00 自动推送天气信息</div>
            </div></div>
            <div class="card"><div class="card-title">托盘行为</div><div class="card-body">
              <label class="switch-label"><input type="checkbox" v-model="showTrayNotification" @change="saveConfig('show_tray_notification', showTrayNotification)"> 关闭窗口时显示托盘提示</label>
            </div></div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { api } from '../api.js'
import SearchPanel from './SearchPanel.vue'
import MemoryPanel from './MemoryPanel.vue'
import RAGPanel from './RAGPanel.vue'
import ChatManagePanel from './ChatManagePanel.vue'
import MigratePanel from './MigratePanel.vue'
import KnowledgePanel from './KnowledgePanel.vue'

const COLOR_PRESETS = {
  vesper: { dark: { primary: '#e8929b', bg: '#1a1418', sidebarBg: '#221a1f', chatBg: '#1a1418', userBubble: '#c76e7a', aiBubble: '#251e25' }, light: { primary: '#c76e7a', bg: '#fff5f5', sidebarBg: '#fce8ec', chatBg: '#fff5f5', userBubble: '#e8929b', aiBubble: '#ffffff' } },
  default: { dark: { primary: '#5390d4', bg: '#0d1117', sidebarBg: '#161b22', chatBg: '#0d1117', userBubble: '#2b5278', aiBubble: '#1e2632' }, light: { primary: '#2b6cb0', bg: '#f5f6fa', sidebarBg: '#e8e9ed', chatBg: '#f5f6fa', userBubble: '#2b6cb0', aiBubble: '#ffffff' } },
  wechat: { dark: { primary: '#07c160', bg: '#0d1117', sidebarBg: '#161b22', chatBg: '#0d1117', userBubble: '#1a6840', aiBubble: '#1e2632' }, light: { primary: '#06ad56', bg: '#f0faf0', sidebarBg: '#e0f0e0', chatBg: '#f0faf0', userBubble: '#06ad56', aiBubble: '#ffffff' } },
  teal: { dark: { primary: '#00897b', bg: '#0d1117', sidebarBg: '#161b22', chatBg: '#0d1117', userBubble: '#00695c', aiBubble: '#1e2632' }, light: { primary: '#00897b', bg: '#f0f8f7', sidebarBg: '#e0f0ed', chatBg: '#f0f8f7', userBubble: '#00897b', aiBubble: '#ffffff' } },
  warm: { dark: { primary: '#ff7043', bg: '#0d1117', sidebarBg: '#161b22', chatBg: '#0d1117', userBubble: '#bf360c', aiBubble: '#1e2632' }, light: { primary: '#ff7043', bg: '#fff8f5', sidebarBg: '#fce8e0', chatBg: '#fff8f5', userBubble: '#ff7043', aiBubble: '#ffffff' } },
}

export default {
  name: 'SettingsPanel',
  components: { SearchPanel, MemoryPanel, RAGPanel, ChatManagePanel, MigratePanel, KnowledgePanel },
  props: {
    show: Boolean,
    settings: Object,
    themeLocal: String,
    ipCity: String,
    relationship: Object,
    emotionTrend: Array,
    totalMessages: Number,
    conversationDays: Number,
    assistantAvatarUrl: String,
    userAvatarUrl: String,
  },
  emits: ['close', 'config-changed', 'open-emotion-panel', 'export-chat', 'data-changed', 'avatar-updated'],
  data() {
    return {
      activeTab: 'api',
      tabs: [
        { id: 'api', icon: 'api', label: 'API' },
        { id: 'role', icon: 'role', label: '角色' },
        { id: 'appearance', icon: 'appr', label: '外观' },
        { id: 'voice', icon: 'voice', label: '语音' },
        { id: 'data', icon: 'data', label: '数据' },
      ],
      // API
      apiProviderLocal: 'deepseek', apiBaseUrlLocal: '', apiModelLocal: '', apiKeyLocal: '',
      apiConfigSaved: false, availableModels: [], loadingModels: false,
      testing: { ds: false, dsStatus: '', dsMsg: '', wt: false, wtStatus: '', wtMsg: '', se: false, seStatus: '', seMsg: '', ipStatus: '', ipMsg: '' },
      amapKeyLocal: '', enableSearchLocal: true,
      // 角色
      aiNameLocal: '夕语', userNameLocal: '', toneLocal: '冷静', lengthLocal: '短', recallLocal: '从不',
      customPromptLocal: '', promptSaved: false,
      sentenceMode: 'auto', proactiveFreq: 'medium', proactiveStyle: 'warm',
      relationshipMode: 'fast', relModeMsg: '',
      newPresetName: '', presets: {},
      // 外观
      colors: { primary: '#5390d4', bg: '#0d1117', sidebarBg: '#161b22', chatBg: '#0d1117', userBubble: '#2b5278', aiBubble: '#1e2632' },
      colorFields: [
        { key: 'primary', label: '主题色', configKey: 'primary_color' },
        { key: 'bg', label: '背景色', configKey: 'bg_color' },
        { key: 'sidebarBg', label: '侧边栏', configKey: 'sidebar_bg' },
        { key: 'chatBg', label: '聊天区', configKey: 'chat_bg' },
        { key: 'userBubble', label: '用户气泡', configKey: 'user_bubble' },
        { key: 'aiBubble', label: 'AI气泡', configKey: 'ai_bubble' },
      ],
      chatBgImage: '',
      // 语音
      ttsEnabled: true, sttEnabled: true, autoPlayVoice: false,
      useSystemNotification: false, notificationStyle: 'warm',
      useWeatherCare: true, showTrayNotification: true,
      // 头像 URL 输入
      aiAvatarUrlLocal: '', userAvatarUrlLocal: '',
      // 定位
      locating: false,
      provinces: [],
      cities: [], selectedProvince: '', selectedCity: '',
    }
  },
  watch: {
    show(val) {
      if (val) this.initFromSettings()
    },
  },
  methods: {
    initFromSettings() {
      const s = this.settings || {}
      this.apiProviderLocal = this._detectProvider(s.api_base_url || '')
      this.apiBaseUrlLocal = s.api_base_url || 'https://api.deepseek.com/v1'
      this.apiModelLocal = s.api_model || 'deepseek-chat'
      this.apiKeyLocal = s.has_api_key ? '••••••••' : ''
      this.enableSearchLocal = s.enable_web_search !== false
      this.amapKeyLocal = s.has_amap_key ? '••••••••' : ''
      this.aiNameLocal = s.ai_name || '夕语'
      this.userNameLocal = s.user_name || ''
      this.toneLocal = s.personality_tone || '冷静'
      this.lengthLocal = s.length_level || '短'
      this.recallLocal = s.recall_past || '从不'
      this.customPromptLocal = s.custom_system_prompt || ''
      this.sentenceMode = s.sentence_mode || 'auto'
      this.proactiveFreq = s.proactive_frequency || 'medium'
      this.proactiveStyle = s.proactive_style || 'warm'
      this.relationshipMode = s.relationship_mode || 'fast'
      if (s.primary_color) this.colors = {
        primary: s.primary_color, bg: s.bg_color || '#0d1117', sidebarBg: s.sidebar_bg || '#161b22',
        chatBg: s.chat_bg || '#0d1117', userBubble: s.user_bubble || '#2b5278', aiBubble: s.ai_bubble || '#1e2632',
      }
      this.chatBgImage = s.chat_bg_image || ''
      this.ttsEnabled = s.tts_enabled !== false
      this.sttEnabled = s.stt_enabled !== false
      this.autoPlayVoice = s.auto_play_voice || false
      this.useSystemNotification = s.use_system_notification || false
      this.notificationStyle = s.notification_style || 'warm'
      this.useWeatherCare = s.use_weather_care !== false
      this.showTrayNotification = s.show_tray_notification !== false
      this.loadProvinces()
    },

    _detectProvider(url) {
      if (!url) return 'deepseek'
      if (url.includes('deepseek')) return 'deepseek'
      if (url.includes('dashscope') || url.includes('aliyun')) return 'qwen'
      if (url.includes('moonshot')) return 'moonshot'
      if (url.includes('bigmodel') || url.includes('zhipu')) return 'zhipu'
      if (url.includes('openai')) return 'openai'
      return 'custom'
    },

    saveConfig(key, value) {
      this.$emit('config-changed', key, value)
    },

    // ─── API ───
    onProviderChange() {
      const p = { deepseek: { url: 'https://api.deepseek.com/v1', model: 'deepseek-chat' }, qwen: { url: 'https://dashscope.aliyuncs.com/compatible-mode/v1', model: 'qwen-plus' }, moonshot: { url: 'https://api.moonshot.cn/v1', model: 'moonshot-v1-8k' }, zhipu: { url: 'https://open.bigmodel.cn/api/paas/v4', model: 'glm-4-flash' }, openai: { url: 'https://api.openai.com/v1', model: 'gpt-4o-mini' }, custom: { url: '', model: '' } }[this.apiProviderLocal]
      if (p) { this.apiBaseUrlLocal = p.url; this.apiModelLocal = p.model; this.availableModels = [] }
    },
    async saveApiConfig() {
      await this.saveConfig('api_base_url', this.apiBaseUrlLocal)
      await this.saveConfig('api_model', this.apiModelLocal)
      if (this.apiKeyLocal && this.apiKeyLocal !== '••••••••' && this.apiKeyLocal.length >= 30) { await this.saveConfig('api_key', this.apiKeyLocal) }
      this.apiConfigSaved = true; setTimeout(() => { this.apiConfigSaved = false }, 2000); this.fetchModels()
    },
    async testApiConnection() {
      this.testing.ds = true; this.testing.dsStatus = ''; this.testing.dsMsg = ''
      try { const res = await api.get('/test/deepseek'); this.testing.dsStatus = res.data.ok ? 'ok' : 'fail'; this.testing.dsMsg = res.data.message } catch (err) { this.testing.dsStatus = 'fail'; this.testing.dsMsg = '请求失败' } finally { this.testing.ds = false }
    },
    async fetchModels() {
      this.loadingModels = true
      try { const res = await api.get('/test/models'); this.availableModels = res.data.models || [] } catch (e) { console.error(e) } finally { this.loadingModels = false }
    },
    async testWeather() {
      this.testing.wt = true; this.testing.wtStatus = ''
      try { const res = await api.get('/test/weather'); this.testing.wtStatus = res.data.ok ? 'ok' : 'fail'; this.testing.wtMsg = res.data.message || res.data.text || '' } catch (e) { this.testing.wtStatus = 'fail'; this.testing.wtMsg = '请求失败' } finally { this.testing.wt = false }
    },
    async testSearch() {
      this.testing.se = true; this.testing.seStatus = ''
      try { const res = await api.get('/test/search'); this.testing.seStatus = res.data.ok ? 'ok' : 'fail'; this.testing.seMsg = res.data.message || '' } catch (e) { this.testing.seStatus = 'fail'; this.testing.seMsg = '请求失败' } finally { this.testing.se = false }
    },
    async locateAndFill() {
      this.locating = true; this.testing.ipMsg = ''
      try {
        const res = await api.get('/location/ip')
        if (res.data.city) { this.$emit('config-changed', 'precise_city', res.data.city); this.testing.ipStatus = 'ok'; this.testing.ipMsg = res.data.city } else { this.testing.ipStatus = 'fail'; this.testing.ipMsg = res.data.error || '获取失败' }
      } catch (e) { this.testing.ipStatus = 'fail'; this.testing.ipMsg = '请求失败' } finally { this.locating = false }
    },
    async preciseLocate() {
      this.locating = true
      try {
        const pos = await new Promise((resolve, reject) => { navigator.geolocation.getCurrentPosition(resolve, reject, { timeout: 10000 }) })
        const res = await api.post('/location/geo', { lat: pos.coords.latitude, lng: pos.coords.longitude })
        if (res.data.city) { this.$emit('config-changed', 'precise_city', res.data.city); this.testing.ipStatus = 'ok'; this.testing.ipMsg = res.data.city }
      } catch (e) { this.testing.ipMsg = '定位失败，请检查权限' } finally { this.locating = false }
    },
    resetLocationPermission() {
      try { localStorage.removeItem('location_granted'); localStorage.removeItem('location_denied') } catch (e) {}
      this.locateAndFill()
    },
    async loadProvinces() {
      if (this.provinces.length) return
      try { const res = await api.get('/location/provinces'); this.provinces = res.data || [] } catch (e) { console.error(e) }
    },
    async onProvinceChange() {
      if (!this.selectedProvince) { this.cities = []; return }
      try { const res = await api.get(`/location/cities/${this.selectedProvince}`); this.cities = res.data || [] } catch (e) { console.error(e) }
      this.selectedCity = ''
    },
    async savePreciseCity() {
      const prov = this.provinces.find(p => p.adcode === this.selectedProvince)
      const city = this.cities.find(c => c.adcode === this.selectedCity)
      if (city) { await this.saveConfig('precise_city', (prov ? prov.name : '') + city.name); this.$emit('config-changed', 'precise_city', (prov ? prov.name : '') + city.name) }
    },

    // ─── 角色 ───
    saveTone() {
      this.$emit('config-changed', 'personality_tone', this.toneLocal)
    },
    async saveCustomPrompt() {
      await this.saveConfig('custom_system_prompt', this.customPromptLocal); this.promptSaved = true; setTimeout(() => { this.promptSaved = false }, 2000)
    },
    async savePreset() {
      await api.post('/settings/presets', { name: this.newPresetName, data: { tone: this.toneLocal, length: this.lengthLocal, recall: this.recallLocal, prompt: this.customPromptLocal } })
      this.newPresetName = ''; this.loadPresets()
    },
    async loadPresets() {
      try { const res = await api.get('/settings/presets'); this.presets = res.data } catch (e) { console.error(e) }
    },
    async loadPreset(name) {
      const preset = this.presets[name]
      if (preset) {
        if (preset.tone) { this.toneLocal = preset.tone; await this.saveTone() }
        if (preset.length) { this.lengthLocal = preset.length; await this.saveConfig('length_level', preset.length) }
        if (preset.recall) { this.recallLocal = preset.recall; await this.saveConfig('recall_past', preset.recall) }
        if (preset.prompt) { this.customPromptLocal = preset.prompt; await this.saveConfig('custom_system_prompt', preset.prompt) }
      }
    },
    async deletePreset(name) { await api.delete(`/settings/presets/${encodeURIComponent(name)}`); this.loadPresets() },
    async switchRelMode() {
      try {
        const res = await api.post('/settings/relationship-mode', { mode: this.relationshipMode })
        if (res.data.ok) { this.relModeMsg = res.data.clamped ? '已切换（' + res.data.clamp_reason + '）' : '已切换'; setTimeout(() => { this.relModeMsg = '' }, 3000) }
        else { this.relModeMsg = res.data.error; this.relationshipMode = this.relationshipMode === 'fast' ? 'long_term' : 'fast'; setTimeout(() => { this.relModeMsg = '' }, 3000) }
      } catch (e) { console.error(e) }
    },
    async uploadAvatar(role, event) {
      const file = event.target.files[0]; if (!file) return
      const form = new FormData(); form.append('file', file)
      try { const res = await api.post(`/avatar/upload/${role}`, form); this.$emit('avatar-updated', role, res.data.url) } catch (e) { console.error(e) }
    },
    async uploadAvatarByUrl(role) {
      const url = role === 'assistant' ? this.aiAvatarUrlLocal : this.userAvatarUrlLocal
      try { await api.post(`/avatar/upload-url/${role}`, { url }); this.$emit('avatar-updated', role, url) } catch (e) { console.error(e) }
    },

    // ─── 外观 ───
    setTheme(theme) {
      document.documentElement.setAttribute('data-theme', theme)
      const metaColors = { dark: '#0d1117', light: '#f5f6fa', vesper: '#1a1418', vesper: '#12101a' }
      const meta = document.getElementById('theme-color-meta')
      if (meta) meta.content = metaColors[theme] || '#0d1117'
      this.saveConfig('theme', theme)
    },
    updateColorField(key, value) {
      if (/^#[0-9a-fA-F]{6}$/.test(value)) { this.colors[key] = value }
      const map = { primary: 'primary_color', bg: 'bg_color', sidebarBg: 'sidebar_bg', chatBg: 'chat_bg', userBubble: 'user_bubble', aiBubble: 'ai_bubble' }
      if (map[key]) this.saveConfig(map[key], this.colors[key])
    },
    applyColorPreset(name) {
      const preset = COLOR_PRESETS[name]; if (!preset) return
      const v = this.themeLocal === 'light' ? preset.light : preset.dark
      this.colors = { ...v }
      for (const [k, val] of Object.entries(this.colors)) this.updateColorField(k, val)
    },
    async saveColorPreset() {
      const name = this.newPresetName || '自定义'
      await api.post('/settings/presets', { name, data: { colors: this.colors } }); this.newPresetName = ''; this.loadPresets()
    },
    uploadBg() { this.$refs.bgInput.click() },
    async onBgFilePicked(e) {
      const file = e.target.files[0]; if (!file) return
      const form = new FormData(); form.append('file', file)
      try { const res = await api.post('/avatar/upload/bg', form); this.chatBgImage = res.data.url; this.saveConfig('chat_bg_image', res.data.url) } catch (err) { console.error(err) }
    },

    // ─── 语音 ───
    saveVoice() {
      this.saveConfig('voice', { tts_enabled: this.ttsEnabled, stt_enabled: this.sttEnabled, auto_play: this.autoPlayVoice })
    },
  },
}
</script>

<style scoped>
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,.5); backdrop-filter: blur(6px); display: flex; justify-content: center; align-items: center; z-index: 1000; }
.settings-panel { position: fixed; width: 820px; height: 580px; background: var(--bg); border-radius: 10px; display: flex; flex-direction: column; overflow: hidden; box-shadow: 0 16px 48px rgba(0,0,0,.5); z-index: 1001; }
.settings-titlebar { display: flex; justify-content: space-between; align-items: center; padding: 8px 16px; background: var(--sb); cursor: move; user-select: none; border-bottom: 1px solid rgba(128,128,128,.1); }
.settings-titlebar span { color: #ecf0f1; font-size: 14px; font-weight: 600; }
.settings-titlebar button { background: none; border: none; color: #888; font-size: 18px; cursor: pointer; padding: 4px 8px; border-radius: 4px; transition: color .15s; }
.settings-titlebar button:hover { background: rgba(128,128,128,.1); color: #fff; }
.settings-body { display: flex; flex: 1; overflow: hidden; }
.settings-nav { width: 120px; background: var(--sb); padding: 8px 0; display: flex; flex-direction: column; gap: 2px; border-right: 1px solid rgba(128,128,128,.1); }
.nav-item { display: flex; align-items: center; gap: 8px; padding: 8px 14px; cursor: pointer; font-size: 12px; color: #999; transition: all .15s; border-left: 2px solid transparent; }
.nav-item:hover { color: #ddd; background: rgba(255,255,255,.06); }
.nav-item.active { color: #fff; background: rgba(255,255,255,.06); border-left-color: var(--p); }
.nav-icon { font-size: 16px; width: 20px; text-align: center; }
.nav-label { white-space: nowrap; }
.settings-content { flex: 1; overflow-y: auto; padding: 16px 20px; background: var(--bg); scrollbar-width: none; }
.settings-content::-webkit-scrollbar { display: none; }
.tab-content { display: flex; flex-direction: column; gap: 10px; }
.card { background: var(--ab); border-radius: 8px; overflow: hidden; border: 1px solid rgba(255,255,255,.03); }
.card-title { padding: 9px 14px; font-size: 12px; font-weight: 500; color: #bdc3c7; background: rgba(128,128,128,.08); border-bottom: 1px solid rgba(255,255,255,.04); letter-spacing: .3px; }
.card-body { padding: 10px 14px; overflow: hidden; }
.card-grow { flex: 1; display: flex; flex-direction: column; }
.card-body-grow { flex: 1; display: flex; flex-direction: column; }
.field { margin-bottom: 12px; }
.field label { display: block; font-size: 12px; color: #8899aa; margin-bottom: 5px; }
.field input, .field select, .field textarea, .api-row input, .loc-row select { width: 100%; padding: 8px 12px; border-radius: 6px; background: #0f1923; color: #ddd; border: 1px solid rgba(255,255,255,.08); font-size: 13px; outline: none; resize: vertical; font-family: inherit; box-sizing: border-box; }
.field input:focus, .field select:focus, .field textarea:focus, .api-row input:focus, .loc-row select:focus { border-color: var(--p); }
.model-row { display: flex; gap: 6px; align-items: center; }
.model-select { flex: 1; padding: 8px 12px; border-radius: 6px; background: #0f1923; color: #ddd; border: 1px solid rgba(255,255,255,.08); font-size: 13px; outline: none; box-sizing: border-box; }
.model-select:focus { border-color: var(--p); }
.model-input { flex: 1; padding: 8px 12px; border-radius: 6px; background: #0f1923; color: #ddd; border: 1px solid rgba(255,255,255,.08); font-size: 13px; outline: none; box-sizing: border-box; }
.model-input:focus { border-color: var(--p); }
.saved-hint { color: #4caf50; font-size: 11px; margin-top: 6px; }
.prompt-actions { display: flex; align-items: center; gap: 8px; margin-top: 8px; }
.test-row { display: flex; align-items: center; gap: 10px; padding: 4px 0; font-size: 13px; }
.test-row > span:first-child { width: 50px; color: #8899aa; font-size: 12px; flex-shrink: 0; }
.test-row > span:last-child { font-size: 11px; flex: 1; }
.api-row { display: flex; gap: 8px; align-items: center; }
.api-row input { flex: 1; }
.empty-hint { color: #7f8c8d; text-align: center; font-size: 12px; padding: 12px; }
.switch-label { display: flex; align-items: center; gap: 8px; padding: 4px 0; color: #bdc3c7; font-size: 13px; cursor: pointer; }
.switch-label input[type="checkbox"] { accent-color: var(--p); }
.theme-toggle { display: flex; gap: 8px; }
.theme-btn { flex: 1; padding: 10px; background: rgba(255,255,255,.04); color: #999; border: 1px solid rgba(255,255,255,.06); border-radius: 6px; cursor: pointer; font-size: 14px; transition: all .15s; }
.theme-btn:hover { color: #ccc; }
.theme-btn.active { background: var(--p); color: #fff; border-color: var(--p); }
.color-list { display: flex; flex-direction: column; gap: 8px; margin-bottom: 12px; }
.color-row { display: flex; align-items: center; gap: 10px; }
.color-swatch { width: 22px; height: 22px; border-radius: 4px; border: 1px solid rgba(255,255,255,.15); flex-shrink: 0; }
.color-label { width: 56px; font-size: 12px; color: #8899aa; flex-shrink: 0; }
.color-pick { width: 28px; height: 24px; padding: 0; border: 1px solid rgba(255,255,255,.1); border-radius: 4px; cursor: pointer; background: transparent; }
.color-hex { width: 80px; padding: 4px 8px !important; border-radius: 4px; background: #0f1923; color: #ccc; border: 1px solid rgba(255,255,255,.08); font-size: 12px; outline: none; font-family: monospace; }
.preset-bar { display: flex; gap: 6px; flex-wrap: wrap; }
.bg-row { display: flex; gap: 8px; }
.bg-row input { flex: 1; }
.avatar-section { display: flex; gap: 16px; align-items: flex-start; padding: 12px 0; border-bottom: 1px solid rgba(255,255,255,.05); }
.avatar-section:last-of-type { border-bottom: none; }
.avatar-preview { display: flex; flex-direction: column; align-items: center; gap: 6px; }
.avatar-img { width: 64px; height: 64px; border-radius: 50%; object-fit: cover; border: 2px solid rgba(255,255,255,.1); }
.avatar-label { font-size: 11px; color: #8899aa; }
.avatar-actions { flex: 1; display: flex; flex-direction: column; gap: 8px; }
.url-row { display: flex; gap: 6px; }
.url-input { flex: 1; padding: 6px 10px; border-radius: 6px; background: #0f1923; color: #ddd; border: 1px solid rgba(255,255,255,.08); font-size: 12px; outline: none; }
.url-input:focus { border-color: var(--p); }
.relationship-bars { display: flex; flex-direction: column; gap: 12px; }
.rel-item { display: flex; align-items: center; gap: 10px; }
.rel-label { width: 50px; font-size: 12px; color: #8899aa; flex-shrink: 0; }
.rel-bar { flex: 1; height: 8px; background: rgba(255,255,255,.08); border-radius: 4px; overflow: hidden; }
.rel-fill { height: 100%; border-radius: 4px; transition: width .3s ease; }
.rel-fill.affection { background: #e8929b; }
.rel-fill.trust { background: #5390d4; }
.rel-value { width: 30px; font-size: 12px; color: #bdc3c7; text-align: right; }
.ai-emotion-tag { font-size: 12px; color: var(--p); font-weight: 500; }
.ai-emotion-desc { font-size: 11px; color: #8899aa; }
.loc-row { display: flex; gap: 6px; margin-top: 8px; }
.loc-row select { flex: 1; padding: 7px 10px; border-radius: 6px; background: #0f1923; color: #ddd; border: 1px solid rgba(255,255,255,.06); font-size: 12px; outline: none; }
.loc-hint { font-size: 11px; color: #8899aa; margin-top: 6px; }
.stats-bar { display: flex; align-items: center; padding: 14px 18px; background: var(--p); border-radius: 8px; margin-bottom: 10px; }
.stat-item { display: flex; flex-direction: column; align-items: center; flex: 1; }
.stat-val { font-size: 20px; font-weight: 600; color: #fff; }
.stat-lbl { font-size: 11px; color: rgba(255,255,255,.7); margin-top: 2px; }
.stat-sep { width: 1px; height: 28px; background: rgba(255,255,255,.3); }
.ip-status { color: #4caf50; font-size: 13px; margin-bottom: 8px; }
.emotion-chart { display: flex; align-items: flex-end; gap: 8px; height: 80px; padding: 0 4px; }
.emotion-bar-group { display: flex; flex-direction: column; align-items: center; flex: 1; height: 100%; justify-content: flex-end; }
.emotion-bar { width: 100%; min-height: 2px; border-radius: 2px 2px 0 0; transition: height .3s ease; }
.emotion-date { font-size: 10px; color: #8899aa; margin-top: 4px; }
.btn { padding: 8px 18px; background: var(--p); color: #fff; border: none; border-radius: 6px; cursor: pointer; font-size: 13px; white-space: nowrap; }
.btn:hover { opacity: .85; }
.btn:disabled { opacity: .4; cursor: default; }
.btn-s { padding: 5px 14px; background: rgba(255,255,255,.08); color: #ccc; border: none; border-radius: 6px; cursor: pointer; font-size: 12px; }
.btn-s:hover { background: rgba(255,255,255,.14); color: #fff; }
.btn-s:disabled { opacity: .3; }
.btn-danger { background: rgba(231,76,60,.15) !important; color: #e74c3c !important; }
.btn-danger:hover { background: rgba(231,76,60,.3) !important; }
.btn-row { display: flex; gap: 8px; }
.role-textarea { width: 100%; min-height: 120px; resize: vertical; }
.test-row .ok { color: #4caf50; }
.test-row .fail { color: #e74c3c; }
</style>
