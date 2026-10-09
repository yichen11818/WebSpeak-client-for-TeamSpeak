<template>
  <div :class="['demo-page', 'ws-skin-root', { 'skin-initializing': !skinReady }]" data-ws-part="app" data-ws-page="demo">
    <header class="demo-header" data-ws-part="demo.header">
      <div class="demo-brand" data-ws-part="demo.brand"><span><Icon name="waveform" :size="21" /></span><div><strong>WebSpeak</strong><small>{{ copy.browserClient }}</small></div></div>
      <div class="demo-tools" data-ws-part="demo.header-tools"><span class="demo-badge" data-ws-part="demo.badge">{{ copy.demoBadge }}</span><SkinSwitcher v-model="activeSkinId" data-ws-part="demo.skin-switcher" :menu-label="copy.skinSelector" :options="skinOptions" @change="onSkinChange" /><LanguageSwitcher v-model="language" data-ws-part="demo.language-switcher" :menu-label="copy.languageMenu" @change="persistLanguage" /><a href="/" data-ws-part="demo.home-link">{{ copy.back }}</a></div>
    </header>

    <div v-if="reconnecting" class="demo-reconnect" data-ws-part="demo.reconnect" role="status"><Icon name="refresh" :size="17" /><span>{{ copy.reconnecting }}</span><button type="button" data-ws-part="demo.reconnect.restore" @click="reconnecting = false">{{ copy.restore }}</button></div>
    <main class="demo-shell" data-ws-part="demo.layout">
      <!-- Discord Leftmost Guild Rail (Rendered when in Discord skin) -->
      <nav v-if="activeSkinId === 'builtin.discord'" class="demo-guild-rail" aria-label="Guilds">
        <div class="guild-pill-active"></div>
        <button type="button" class="guild-icon guild-home active" title="私信">
          <Icon name="discord" :size="24" />
        </button>
        <div class="guild-separator"></div>
        <div class="guild-item">
          <button type="button" class="guild-icon guild-server" title="WebSpeak Official">
            <span>WS</span>
          </button>
          <span class="guild-badge">3</span>
        </div>
        <div class="guild-item">
          <button type="button" class="guild-icon guild-server" title="TeamSpeak 交流群">
            <span>TS</span>
          </button>
        </div>
        <div class="guild-item">
          <button type="button" class="guild-icon guild-action" title="模拟添加频道/邀请" @click="toggleSpeaking">
            <Icon name="plus" :size="20" />
          </button>
        </div>
        <div class="guild-item">
          <button type="button" class="guild-icon guild-action" title="交流群与社区" @click="poke = !poke">
            <Icon name="compass" :size="20" />
          </button>
        </div>
        <div class="guild-separator"></div>
        <div class="guild-item">
          <a class="guild-icon guild-action guild-download" href="https://github.com/EchoSixHIYA/WebSpeak-client-for-TeamSpeak" target="_blank" rel="noreferrer" title="查看 GitHub 源码与版本">
            <Icon name="download" :size="20" />
          </a>
        </div>
      </nav>

      <aside class="demo-channel-panel" data-ws-part="demo.channels">
        <!-- Discord Server Header Card -->
        <div v-if="activeSkinId === 'builtin.discord'" class="discord-server-card">
          <div class="discord-server-card-content">
            <span class="discord-server-card-avatar">👾</span>
            <strong>O.P.Neon Games</strong>
            <Icon name="chevron-down" :size="15" class="discord-server-chevron" />
          </div>
        </div>

        <div v-if="activeSkinId !== 'builtin.discord'" class="demo-panel-title" data-ws-part="demo.channels.heading"><span>{{ copy.channels }}</span><strong>{{ selectedChannel.name }}</strong></div>

        <div v-if="activeSkinId === 'builtin.discord'" class="discord-channels-scroll-area">
          <div class="discord-voice-category-heading">
            <Icon name="chevron-down" :size="11" />
            <span>VOICE CHANNELS</span>
          </div>

          <div v-for="channel in channels" :key="channel.id" :class="['demo-channel', { active: selectedChannelId === channel.id }]" data-ws-part="demo.channel" :data-ws-state="selectedChannelId === channel.id ? 'active' : 'idle'">
            <button type="button" data-ws-part="demo.channel.select" @click="selectChannel(channel.id)"><Icon name="volume" :size="16" /><span>{{ channel.name }}</span><small>{{ channel.members.length }}</small></button>
            <div class="demo-channel-members" data-ws-part="demo.channel.members"><span v-for="member in channel.members" :key="member.id" :class="{ speaking: speakingId === member.id }"><i :style="avatarStyle(member.name)">{{ member.name[0] }}</i>{{ member.name }}</span></div>
          </div>
        </div>

        <template v-else>
          <div v-for="channel in channels" :key="channel.id" :class="['demo-channel', { active: selectedChannelId === channel.id }]" data-ws-part="demo.channel" :data-ws-state="selectedChannelId === channel.id ? 'active' : 'idle'">
            <button type="button" data-ws-part="demo.channel.select" @click="selectChannel(channel.id)"><Icon name="volume" :size="16" /><span>{{ channel.name }}</span><small>{{ channel.members.length }}</small></button>
            <div class="demo-channel-members" data-ws-part="demo.channel.members"><span v-for="member in channel.members" :key="member.id" :class="{ speaking: speakingId === member.id }"><i :style="avatarStyle(member.name)">{{ member.name[0] }}</i>{{ member.name }}</span></div>
          </div>
        </template>

        <!-- Discord Voice Connected Strip above User Bar -->
        <div v-if="activeSkinId === 'builtin.discord'" class="discord-connected-strip">
          <div class="discord-connected-left">
            <span class="discord-connected-signal">
              <Icon name="waveform" :size="16" />
            </span>
            <div class="discord-connected-copy">
              <strong>语音已连接</strong>
              <small>{{ selectedChannel.name }} / O.P.Neon</small>
            </div>
          </div>
          <button type="button" class="discord-disconnect-btn" title="断开连接" @click="reconnecting = true">
            <Icon name="door" :size="17" />
          </button>
        </div>

        <!-- Discord Bottom Left Pinned User Profile & Controls Dock -->
        <div v-if="activeSkinId === 'builtin.discord'" class="discord-user-bar">
          <div class="discord-user-info">
            <div class="discord-avatar-wrap">
              <i :style="avatarStyle('illusia')">I</i>
              <span class="discord-status-dot"></span>
            </div>
            <div class="discord-user-text">
              <strong>yichen2218</strong>
              <small>#0001 · 在线</small>
            </div>
          </div>
          <div class="discord-user-actions">
            <button
              type="button"
              class="discord-icon-btn"
              :class="{ muted: isMuted }"
              :title="isMuted ? '取消静音麦克风' : '静音麦克风'"
              @click="toggleMute"
            >
              <Icon :name="isMuted ? 'mic-off' : 'mic'" :size="18" />
            </button>
            <button
              type="button"
              class="discord-icon-btn"
              :class="{ muted: isDeafened }"
              :title="isDeafened ? '取消静音耳机' : '静音耳机'"
              @click="toggleDeafen"
            >
              <Icon :name="isDeafened ? 'volume-off' : 'headset'" :size="18" />
            </button>
            <button
              type="button"
              class="discord-icon-btn"
              title="用户设置"
              @click="showSettingsModal = !showSettingsModal"
            >
              <Icon name="settings" :size="18" />
            </button>
          </div>
        </div>
      </aside>

      <!-- Central Stage: Discord Video & Voice Stage -->
      <section class="demo-main" data-ws-part="demo.main">
        <!-- Discord Stage Header -->
        <div v-if="activeSkinId === 'builtin.discord'" class="discord-stage-header">
          <div class="discord-channel-title">
            <Icon name="volume" :size="20" class="discord-channel-icon" />
            <h2>{{ selectedChannel.name }}</h2>
          </div>
          <div class="discord-stage-header-actions">
            <button
              type="button"
              class="discord-header-btn"
              :class="{ active: activeTab === 'channel' }"
              :title="activeTab === 'channel' ? '收起文字聊天' : '展开文字聊天'"
              @click="activeTab = activeTab === 'channel' ? 'server' : 'channel'"
            >
              <Icon name="message" :size="19" />
            </button>
            <button
              type="button"
              class="discord-header-btn"
              :class="{ active: showMemberList }"
              :title="showMemberList ? '收起成员名单' : '展开成员名单'"
              @click="showMemberList = !showMemberList"
            >
              <Icon name="users" :size="19" />
            </button>
          </div>
        </div>

        <!-- Discord Voice Stage Canvas (Speakers Cards with Green Glow) -->
        <div v-if="activeSkinId === 'builtin.discord'" class="discord-stage-canvas">
          <div class="discord-speakers-grid">
            <!-- Active Speaker 1 (yichen2218 / user) -->
            <div class="discord-speaker-card" :class="{ speaking: speakingId === 'illusia' && !isMuted }">
              <div class="discord-speaker-avatar">
                <i :style="avatarStyle('illusia')">
                  <Icon name="discord" :size="48" />
                </i>
              </div>
              <div class="discord-speaker-tag">
                <span class="discord-speaker-name">yichen2218</span>
              </div>
            </div>

            <!-- Active Speaker 2 / Room Visual Stage Card -->
            <div class="discord-speaker-card stage-artwork-card">
              <div v-if="isSharingScreen" class="discord-screenshare-preview">
                <Icon name="monitor" :size="64" />
                <span>yichen2218 正在共享屏幕</span>
              </div>
              <template v-else>
                <div class="discord-stage-artwork">
                  <span class="discord-trophy-emoji">🏆</span>
                  <span class="discord-shield-emoji">🛡️</span>
                  <span class="discord-gem-emoji">💎</span>
                </div>
                <div class="discord-invite-stage-btn-wrap">
                  <button type="button" class="discord-stage-invite-btn" @click="toggleSpeaking">
                    <Icon name="users" :size="15" />
                    <span>邀请到语音</span>
                  </button>
                </div>
              </template>
            </div>
          </div>

          <!-- Bottom Voice Floating Control Bar in Stage -->
          <div class="discord-floating-controls">
            <button
              type="button"
              class="stage-ctrl-btn"
              :class="{ danger: isMuted }"
              :title="isMuted ? '取消静音麦克风' : '静音麦克风'"
              @click="toggleMute"
            >
              <Icon :name="isMuted ? 'mic-off' : 'mic'" :size="19" />
            </button>
            <button
              type="button"
              class="stage-ctrl-btn"
              :class="{ active: isSharingScreen }"
              :title="isSharingScreen ? '停止屏幕共享' : '共享您的屏幕'"
              @click="toggleScreenShare"
            >
              <Icon name="monitor" :size="19" />
            </button>
            <button
              type="button"
              class="stage-ctrl-btn"
              :class="{ active: poke }"
              title="活动 / 戳一戳互动"
              @click="poke = !poke"
            >
              <Icon name="waveform" :size="19" />
            </button>
            <button
              type="button"
              class="stage-ctrl-btn danger-disconnect"
              title="断开连接"
              @click="reconnecting = true"
            >
              <Icon name="door" :size="19" />
            </button>
          </div>
        </div>

        <!-- Default Theme Fallback (Non-Discord) -->
        <template v-else>
          <div class="demo-hero" data-ws-part="demo.hero"><div><span class="demo-live" data-ws-part="demo.live"><i></i>{{ copy.simulated }}</span><h1 data-ws-part="demo.hero.title"><Icon name="volume" :size="24" /> {{ selectedChannel.name }}</h1><p data-ws-part="demo.hero.description">{{ copy.heroLead }}</p><small data-ws-part="demo.hero.online"><Icon name="users" :size="14" /> {{ selectedChannel.members.length }} {{ copy.online }}</small></div><div class="demo-wave" data-ws-part="demo.wave" aria-hidden="true"><i v-for="bar in bars" :key="bar" :style="{ height: `${bar}px` }"></i></div></div>
          <div class="demo-section-heading" data-ws-part="demo.voice-heading"><span>{{ copy.voiceActivity }}</span><strong>{{ copy.speakingNow }}</strong></div>
          <div class="demo-voice-grid" data-ws-part="demo.voice-grid"><article v-for="member in selectedChannel.members" :key="member.id" :class="['demo-voice-card', { speaking: speakingId === member.id }]" data-ws-part="demo.voice-card" :data-ws-state="speakingId === member.id ? 'speaking' : 'connected'" @click="speakingId = member.id"><i data-ws-part="demo.avatar" :style="avatarStyle(member.name)">{{ member.name[0] }}</i><strong data-ws-part="demo.member-name">{{ member.name }}</strong><span data-ws-part="demo.member-status">{{ speakingId === member.id ? copy.speaking : copy.connected }}</span></article></div>
          <div class="demo-chat-head" data-ws-part="demo.chat.heading"><div><span>{{ copy.textChannel }}</span><strong># {{ selectedChannel.name }} {{ copy.chat }}</strong></div><div class="demo-tabs" data-ws-part="demo.chat.tabs"><button v-for="tab in tabs" :key="tab.id" type="button" :class="{ active: activeTab === tab.id }" data-ws-part="demo.chat.tab" :data-ws-state="activeTab === tab.id ? 'active' : 'idle'" @click="activeTab = tab.id">{{ tab.label }}</button></div></div>
          <div class="demo-messages" data-ws-part="demo.chat.messages"><div v-for="message in visibleMessages" :key="message.id" :class="['demo-message', { mine: message.mine }]" data-ws-part="demo.chat.message" :data-ws-state="message.mine ? 'mine' : 'other'" @dblclick="message.mine = !message.mine"><i :style="avatarStyle(message.author)">{{ message.author[0] }}</i><div><small>{{ message.author }} · {{ message.time }}</small><p>{{ messageText(message) }}</p></div></div><div v-if="!visibleMessages.length" class="demo-empty" data-ws-part="demo.chat.empty"><Icon name="message" :size="22" /><strong>{{ copy.emptyChat }}</strong></div></div>
          <form class="demo-composer" data-ws-part="demo.chat.composer" @submit.prevent="sendMessage"><input v-model="draft" data-ws-part="demo.chat.input" :placeholder="copy.placeholder" :aria-label="copy.placeholder" /><button type="submit" data-ws-part="demo.chat.send" :disabled="!draft.trim()"><Icon name="send" :size="17" /></button></form>
        </template>
      </section>

      <!-- Right Column: Discord Channel Chat Panel (默认频道内聊天) -->
      <aside class="demo-actions-panel" data-ws-part="demo.actions">
        <!-- Discord Chat Header in Right Column -->
        <div v-if="activeSkinId === 'builtin.discord'" class="discord-chat-sidebar-header">
          <div class="discord-chat-title">
            <Icon name="message" :size="18" />
            <strong>{{ selectedChannel.name }}</strong>
          </div>
          <div class="discord-chat-header-actions">
            <button
              type="button"
              class="discord-chat-close-btn"
              title="收起文字聊天"
              @click="activeTab = activeTab === 'channel' ? 'server' : 'channel'"
            >
              <Icon name="close" :size="16" />
            </button>
          </div>
        </div>

        <!-- Discord Messages Stream in Right Column -->
        <div v-if="activeSkinId === 'builtin.discord'" class="discord-channel-chat-stream">
          <div class="discord-chat-welcome">
            <div class="discord-welcome-hash-circle">
              <Icon name="message" :size="32" />
            </div>
            <h2>欢迎来到 {{ selectedChannel.name }}!</h2>
            <p>这是 {{ selectedChannel.name }} 频道的起点。</p>
          </div>

          <div class="discord-date-divider">
            <span>2026年10月9日</span>
          </div>

          <!-- Message Rows -->
          <div v-for="message in visibleMessages" :key="message.id" class="discord-chat-row" :class="{ mine: message.mine }">
            <div class="discord-row-avatar">
              <i :style="avatarStyle(message.author)">{{ message.author[0] }}</i>
            </div>
            <div class="discord-row-content">
              <div class="discord-row-header">
                <span class="discord-row-author">{{ message.author }}</span>
                <span v-if="message.author === 'msicbot'" class="discord-bot-badge">APP</span>
                <span class="discord-row-time">{{ message.time }}</span>
              </div>
              <p class="discord-row-text">{{ messageText(message) }}</p>
            </div>
          </div>
        </div>

        <!-- Discord Composer at Bottom of Right Column -->
        <form v-if="activeSkinId === 'builtin.discord'" class="discord-channel-composer" @submit.prevent="sendMessage">
          <div class="discord-composer-wrap">
            <input v-model="draft" :placeholder="`给 #${selectedChannel.name} 发消息`" />
            <div class="discord-composer-actions">
              <button type="submit" class="discord-composer-send" :disabled="!draft.trim()" title="发送">
                <Icon name="send" :size="16" />
              </button>
            </div>
          </div>
        </form>

        <!-- Default Theme Fallback Simulation Controls -->
        <template v-else>
          <div class="demo-panel-title" data-ws-part="demo.actions.heading"><span>{{ copy.voice }}</span><strong>{{ copy.simulationControls }}</strong></div>
          <button type="button" data-ws-part="demo.action.speaking" @click="toggleSpeaking"><Icon name="mic" :size="17" /> {{ speakingId ? copy.stopSpeaking : copy.simulateSpeaking }}</button>
          <button type="button" data-ws-part="demo.action.poke" @click="poke = !poke"><Icon name="bell" :size="17" /> {{ copy.poke }}</button>
          <button type="button" data-ws-part="demo.action.reconnect" @click="reconnecting = true"><Icon name="refresh" :size="17" /> {{ copy.simulateReconnect }}</button>
          <div class="demo-note" data-ws-part="demo.note"><Icon name="shield" :size="16" /><span>{{ copy.demoNote }}</span></div>
          <div class="demo-user" data-ws-part="demo.user"><i :style="avatarStyle('illusia')">I</i><div><strong>illusia</strong><small>{{ copy.you }} · {{ copy.ready }}</small></div><span class="demo-green-dot"></span></div>
        </template>
      </aside>
    </main>
    <div v-if="poke" class="demo-poke" data-ws-part="demo.poke-notification" role="status"><Icon name="bell" :size="17" /><span><strong>msicbot</strong> {{ copy.pokedYou }}</span><button type="button" data-ws-part="demo.poke.dismiss" @click="poke = false"><Icon name="close" :size="14" /></button></div>

    <!-- Discord User Settings Modal Simulation -->
    <div v-if="showSettingsModal" class="discord-settings-overlay" @click.self="showSettingsModal = false">
      <div class="discord-settings-modal">
        <div class="discord-settings-modal-header">
          <h3>用户设置</h3>
          <button type="button" @click="showSettingsModal = false">
            <Icon name="close" :size="18" />
          </button>
        </div>
        <div class="discord-settings-modal-body">
          <div class="discord-settings-row">
            <div class="discord-settings-row-info">
              <strong>静音麦克风</strong>
              <small>关闭本地麦克风输入</small>
            </div>
            <div class="discord-toggle-switch" :class="{ on: isMuted }" @click="toggleMute">
              <span class="discord-toggle-knob"></span>
            </div>
          </div>
          <div class="discord-settings-row">
            <div class="discord-settings-row-info">
              <strong>静音耳机</strong>
              <small>禁用扬声器声音输出</small>
            </div>
            <div class="discord-toggle-switch" :class="{ on: isDeafened }" @click="toggleDeafen">
              <span class="discord-toggle-knob"></span>
            </div>
          </div>
          <div class="discord-settings-row">
            <div class="discord-settings-row-info">
              <strong>屏幕共享</strong>
              <small>在语音舞台广播当前屏幕画面</small>
            </div>
            <div class="discord-toggle-switch" :class="{ on: isSharingScreen }" @click="toggleScreenShare">
              <span class="discord-toggle-knob"></span>
            </div>
          </div>
        </div>
        <div class="discord-settings-modal-footer">
          <button type="button" class="discord-settings-done-btn" @click="showSettingsModal = false">
            完成
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, shallowRef } from "vue";
import Icon from "../components/Icon.vue";
import LanguageSwitcher from "../components/LanguageSwitcher.vue";
import SkinSwitcher, { type SkinOption } from "../components/SkinSwitcher.vue";
import { listInstalledSkins, loadLocalPreferences, saveLocalPreferences } from "../services/local-persistence.js";
import { BUILTIN_ILLUSIA_SKIN_ID, getPublicDefaultSkinId, isPublicSkinEnabled, listPublicSkins, type SkinCatalogEntry } from "../services/skin-catalog.js";
import { activateSkin, BUILTIN_DARK_SKIN, BUILTIN_LIGHT_SKIN, getStoredSkinId } from "../services/skin-runtime.js";
import type { InstalledSkin } from "../services/skin-pack.js";
import { getStoredTheme, isDarkTheme } from "../services/theme.js";

type Language = "zh" | "en" | "de" | "ru" | "ja";
const illusiaSkinLabels: Record<Language, string> = { zh: "ILLUSIA 风", en: "ILLUSIA style", de: "ILLUSIA-Stil", ru: "Стиль ILLUSIA", ja: "ILLUSIA スタイル" };
interface DemoMessage { id: number; author: string; time: string; text: string; zhText?: string; enText?: string; ruText?: string; jaText?: string; mine: boolean }
type Tab = "channel" | "server";
const storedLanguage = localStorage.getItem("webspeak:language");
const language = ref<Language>(storedLanguage === "en" || storedLanguage === "de" || storedLanguage === "ru" || storedLanguage === "ja" ? storedLanguage : "zh");
const baseCopy = computed(() => language.value === "zh" ? zh : language.value === "de" ? de : language.value === "ru" ? ru : language.value === "ja" ? ja : en);
const activeSkin = shallowRef<InstalledSkin | null>(null);
const storedSkinId = getStoredSkinId();
const activeSkinId = ref(storedSkinId ?? (isDarkTheme(getStoredTheme()) ? BUILTIN_DARK_SKIN : BUILTIN_LIGHT_SKIN));
const skinReady = ref(storedSkinId === BUILTIN_LIGHT_SKIN || storedSkinId === BUILTIN_DARK_SKIN || storedSkinId === "builtin.discord");
const installedSkins = ref<InstalledSkin[]>([]);
const catalogSkins = ref<SkinCatalogEntry[]>([]);
const skinOptions = computed<SkinOption[]>(() => [
  ...catalogSkins.value.map((skin) => ({
    value: skin.id,
    label: skin.id === BUILTIN_LIGHT_SKIN ? copy.value.skinDay : skin.id === BUILTIN_DARK_SKIN ? copy.value.skinNight : skin.id === "builtin.discord" ? "Discord" : skin.id === BUILTIN_ILLUSIA_SKIN_ID ? illusiaSkinLabels[language.value] : skin.name,
    icon: skin.id === BUILTIN_LIGHT_SKIN ? "sun" : skin.id === BUILTIN_DARK_SKIN ? "moon" : skin.id === "builtin.discord" ? "discord" : "compass",
  })),
  ...installedSkins.value.filter((skin) => !catalogSkins.value.some((item) => item.id === skin.id) && isPublicSkinEnabled(skin.id)).map((skin) => ({ value: skin.id, label: skin.name, icon: "compass" })),
]);
const copy = computed(() => {
  const result = { ...baseCopy.value };
  const content = activeSkin.value?.contentData;
  if (!content) return result;
  const fullLocale = language.value === "zh" ? "zh-CN" : language.value === "en" ? "en-US" : language.value === "de" ? "de-DE" : language.value === "ru" ? "ru-RU" : "ja-JP";
  for (const locale of [...new Set([content.defaultLocale, language.value, fullLocale])]) {
    for (const [key, value] of Object.entries(content.locales[locale]?.messages ?? {})) {
      if (!key.startsWith("demo.")) continue;
      const copyKey = key.slice("demo.".length) as keyof typeof result;
      if (Object.prototype.hasOwnProperty.call(result, copyKey) && value.trim()) result[copyKey] = value;
    }
  }
  return result;
});
const selectedChannelId = ref("quiet");
const activeTab = ref<Tab>("channel");
const speakingId = ref("illusia");
const reconnecting = ref(false);
const poke = ref(false);
const draft = ref("");
const isMuted = ref(false);
const isDeafened = ref(false);
const isSharingScreen = ref(false);
const showSettingsModal = ref(false);
const showMemberList = ref(true);

function toggleMute() {
  isMuted.value = !isMuted.value;
  if (isMuted.value) {
    speakingId.value = "";
  } else if (!isDeafened.value) {
    speakingId.value = "illusia";
  }
}

function toggleDeafen() {
  isDeafened.value = !isDeafened.value;
  if (isDeafened.value) {
    isMuted.value = true;
    speakingId.value = "";
  } else {
    isMuted.value = false;
    speakingId.value = "illusia";
  }
}

function toggleScreenShare() {
  isSharingScreen.value = !isSharingScreen.value;
}
const bars = [15, 31, 21, 42, 25, 50, 32, 21, 38, 27, 45, 18, 33];
const channels = [
  { id: "quiet", name: "Quiet Zone", members: [{ id: "illusia", name: "illusia" }, { id: "msicbot", name: "msicbot" }] },
  { id: "lounge", name: "Lounge", members: [{ id: "alice", name: "Alice" }] },
  { id: "music", name: "Music room", members: [] },
];
const messages = ref<DemoMessage[]>([
  { id: 1, author: "illusia", time: "10:24", text: "", zhText: "大家好，欢迎来到 WebSpeak。", enText: "Hello everyone, welcome to WebSpeak.", mine: true },
  { id: 2, author: "msicbot", time: "10:25", text: "", zhText: "语音和文字频道都已同步。", enText: "Voice and text channels are in sync.", mine: false },
]);
const zh = { languageMenu: "语言", skinSelector: "皮肤", skinDay: "日间", skinNight: "夜间", browserClient: "浏览器语音工作台", demoBadge: "演示 — 模拟数据", back: "返回主页", reconnecting: "连接中断，正在恢复…", restore: "恢复连接", channels: "频道", channel: "频道", simulated: "演示状态", heroLead: "这里展示 WebSpeak 的频道、语音和聊天交互。", online: "人在线", voiceActivity: "语音活动", speakingNow: "正在语音中", speaking: "正在说话…", connected: "已连接", textChannel: "文字频道", chat: "聊天", server: "服务器", emptyChat: "暂无消息", placeholder: "发送一条消息…", voice: "语音", simulationControls: "模拟控制", stopSpeaking: "停止说话", simulateSpeaking: "模拟发言", poke: "戳一戳", simulateReconnect: "模拟重连", demoNote: "此页面不会连接真实 TeamSpeak 服务器。", you: "你", ready: "已就绪", pokedYou: "戳了你一下" };
const en = { languageMenu: "Language", skinSelector: "Skin", skinDay: "Day", skinNight: "Night", browserClient: "Browser voice workspace", demoBadge: "Demo — simulated data", back: "Back home", reconnecting: "Connection interrupted, recovering…", restore: "Restore", channels: "Channels", channel: "Channel", simulated: "Simulation", heroLead: "A preview of WebSpeak channels, voice, and chat interactions.", online: "online", voiceActivity: "VOICE ACTIVITY", speakingNow: "Speaking now", speaking: "Speaking…", connected: "Connected", textChannel: "TEXT CHANNEL", chat: "chat", server: "Server", emptyChat: "No messages yet", placeholder: "Send a message…", voice: "Voice", simulationControls: "Simulation controls", stopSpeaking: "Stop speaking", simulateSpeaking: "Simulate speaking", poke: "Poke", simulateReconnect: "Simulate reconnect", demoNote: "This page never connects to a real TeamSpeak server.", you: "You", ready: "Ready", pokedYou: "poked you" };
const de = { languageMenu: "Sprache", skinSelector: "Design", skinDay: "Tag", skinNight: "Nacht", browserClient: "Sprachbereich im Browser", demoBadge: "Demo — simulierte Daten", back: "Zur Startseite", reconnecting: "Verbindung unterbrochen, Wiederherstellung…", restore: "Wiederherstellen", channels: "Kanäle", channel: "Kanal", simulated: "Simulation", heroLead: "Eine Vorschau auf Kanäle, Sprache und Chat von WebSpeak.", online: "online", voiceActivity: "SPRACHAKTIVITÄT", speakingNow: "Spricht gerade", speaking: "Spricht…", connected: "Verbunden", textChannel: "TEXTKANAL", chat: "Chat", server: "Server", emptyChat: "Noch keine Nachrichten", placeholder: "Nachricht senden…", voice: "Sprache", simulationControls: "Simulation steuern", stopSpeaking: "Sprechen stoppen", simulateSpeaking: "Sprechen simulieren", poke: "Anstupsen", simulateReconnect: "Verbindung simulieren", demoNote: "Diese Seite verbindet sich nie mit einem echten TeamSpeak-Server.", you: "Du", ready: "Bereit", pokedYou: "hat dich angestupst" };
const ru = { languageMenu: "Язык", skinSelector: "Оформление", skinDay: "День", skinNight: "Ночь", browserClient: "Голосовое пространство в браузере", demoBadge: "Демо — симуляция", back: "На главную", reconnecting: "Соединение прервано, восстановление…", restore: "Восстановить", channels: "Каналы", channel: "Канал", simulated: "Симуляция", heroLead: "Предпросмотр каналов, голоса и чата WebSpeak.", online: "онлайн", voiceActivity: "ГОЛОСОВАЯ АКТИВНОСТЬ", speakingNow: "Сейчас говорят", speaking: "Говорит…", connected: "Подключён", textChannel: "ТЕКСТОВЫЙ КАНАЛ", chat: "чат", server: "Сервер", emptyChat: "Сообщений пока нет", placeholder: "Написать сообщение…", voice: "Голос", simulationControls: "Управление симуляцией", stopSpeaking: "Остановить речь", simulateSpeaking: "Симулировать речь", poke: "Толкнуть", simulateReconnect: "Симулировать переподключение", demoNote: "Эта страница не подключается к реальному серверу TeamSpeak.", you: "Вы", ready: "Готово", pokedYou: "толкнул вас" };
const ja = { languageMenu: "言語", skinSelector: "スキン", skinDay: "昼", skinNight: "夜", browserClient: "ブラウザ音声ワークスペース", demoBadge: "デモ — シミュレーション", back: "ホームに戻る", reconnecting: "接続が中断されました。復旧中…", restore: "復旧", channels: "チャンネル", channel: "チャンネル", simulated: "シミュレーション", heroLead: "WebSpeak のチャンネル、音声、チャットのプレビューです。", online: "人がオンライン", voiceActivity: "音声アクティビティ", speakingNow: "発話中", speaking: "発話中…", connected: "接続済み", textChannel: "テキストチャンネル", chat: "チャット", server: "サーバー", emptyChat: "メッセージはありません", placeholder: "メッセージを送信…", voice: "音声", simulationControls: "シミュレーション操作", stopSpeaking: "発話を停止", simulateSpeaking: "発話をシミュレート", poke: "つつく", simulateReconnect: "再接続をシミュレート", demoNote: "このページは実際の TeamSpeak サーバーには接続しません。", you: "あなた", ready: "準備完了", pokedYou: "あなたをつつきました" };
const tabs = computed(() => [{ id: "channel" as const, label: copy.value.channel }, { id: "server" as const, label: copy.value.server }]);
const selectedChannel = computed(() => channels.find((channel) => channel.id === selectedChannelId.value) ?? channels[0]);
const visibleMessages = computed(() => activeTab.value === "channel" ? messages.value : []);

function persistLanguage() { localStorage.setItem("webspeak:language", language.value); void saveLocalPreferences({ schemaVersion: 1, language: language.value }); }
async function onSkinChange(skinId: string) {
  localStorage.setItem("webspeak:skin-choice", skinId);
  const catalogSkin = catalogSkins.value.find((skin) => skin.id === skinId);
  activeSkin.value = await activateSkin(skinId, catalogSkin?.version);
  activeSkinId.value = getStoredSkinId() ?? skinId;
  void saveLocalPreferences({ schemaVersion: 1, skinId: activeSkinId.value });
}
onMounted(async () => {
  const catalogPromise = listPublicSkins().catch(() => []);
  try {
    const [preferences, installed, available] = await Promise.all([loadLocalPreferences(), listInstalledSkins(), catalogPromise]);
    installedSkins.value = installed;
    catalogSkins.value = available;
    // Only a deliberate choice should override the instance default. The active
    // skin and local preference also contain automatically applied defaults.
    const savedSkinId = localStorage.getItem("webspeak:skin-choice");
    const selectedId = savedSkinId && isPublicSkinEnabled(savedSkinId) ? savedSkinId : getPublicDefaultSkinId();
    const selectedSkin = available.find((skin) => skin.id === selectedId);
    activeSkinId.value = selectedId;
    activeSkin.value = await activateSkin(selectedId, selectedSkin?.version);
    activeSkinId.value = getStoredSkinId() ?? selectedId;
  } catch {
    activeSkin.value = null;
    activeSkinId.value = isDarkTheme(getStoredTheme()) ? BUILTIN_DARK_SKIN : BUILTIN_LIGHT_SKIN;
    await activateSkin(activeSkinId.value).catch(() => undefined);
  } finally {
    skinReady.value = true;
  }

  void catalogPromise.then(async (available) => {
    catalogSkins.value = available;
    const selected = available.find((skin) => skin.id === activeSkinId.value);
    const installed = installedSkins.value.find((skin) => skin.id === activeSkinId.value);
    if (!selected || !installed || installed.version === selected.version) return;
    activeSkin.value = await activateSkin(selected.id, selected.version);
    if (getStoredSkinId() !== selected.id) return;
    activeSkinId.value = getStoredSkinId() ?? selected.id;
    installedSkins.value = await listInstalledSkins();
  }).catch(() => undefined);
});
function selectChannel(id: string) { selectedChannelId.value = id; activeTab.value = "channel"; }
function toggleSpeaking() { speakingId.value = speakingId.value ? "" : selectedChannel.value.members[0]?.id ?? ""; }
function messageText(message: DemoMessage): string { return language.value === "zh" ? message.zhText ?? message.text : language.value === "ru" ? message.ruText ?? message.enText ?? message.text : language.value === "ja" ? message.jaText ?? message.enText ?? message.text : message.enText ?? message.text; }
function sendMessage() { const text = draft.value.trim(); if (!text) return; messages.value.push({ id: Date.now(), author: "illusia", time: "now", text, mine: true }); draft.value = ""; }
function avatarStyle(name: string) { let hash = 0; for (const char of name) hash = char.charCodeAt(0) + ((hash << 5) - hash); const colors = ["#168f83", "#7b9ed0", "#b99070", "#8b78b9"]; return { background: colors[Math.abs(hash) % colors.length] }; }
</script>

<style scoped>
.demo-page, .demo-page * { box-sizing: border-box; }
.demo-page.skin-initializing { visibility: hidden; }
.demo-page { min-height: 100dvh; padding: 0 28px 38px; font-family: Inter, ui-sans-serif, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; background: radial-gradient(circle at 72% 12%, rgba(126, 213, 205, .18), transparent 26rem), #f7f9f8; color: #1c2b28; }
.demo-page button, .demo-page input { font: inherit; }
.demo-header { display: flex; align-items: center; justify-content: space-between; width: min(1380px, 100%); min-height: 82px; margin: 0 auto; }
.demo-brand, .demo-tools, .demo-brand > div, .demo-tools { display: flex; align-items: center; }.demo-brand { gap: 10px; }.demo-brand > span { display: grid; place-items: center; width: 39px; height: 39px; color: #fff; background: #006a64; border-radius: 12px; }.demo-brand strong, .demo-brand small { display: block; }.demo-brand strong { color: #006a64; font-size: 18px; }.demo-brand small { margin-top: 2px; color: #84938e; font-size: 10px; }.demo-tools { gap: 15px; color: #68807a; font-size: 12px; }.demo-tools button, .demo-tools a { padding: 7px 10px; color: #006a64; background: #e1f2ee; border: 1px solid #cbe6e0; border-radius: 7px; font-size: 11px; font-weight: 700; text-decoration: none; cursor: pointer; }.demo-badge { color: #2d7d50; font-weight: 700; }
.demo-reconnect { display: flex; align-items: center; gap: 9px; width: min(1380px, 100%); margin: 0 auto 14px; padding: 11px 14px; color: #8b6537; background: #fff8e9; border: 1px solid #efd9aa; border-radius: 10px; }.demo-reconnect span { flex: 1; }.demo-reconnect button { padding: 6px 10px; color: #7d5d35; background: #fff; border: 1px solid #ead0a0; border-radius: 6px; cursor: pointer; }
.demo-shell { display: grid; grid-template-columns: 240px minmax(0, 1fr) 235px; gap: 18px; width: min(1380px, 100%); min-height: 720px; margin: 0 auto; }.demo-channel-panel, .demo-actions-panel, .demo-main { min-width: 0; padding: 20px; border: 1px solid #e0ebe7; border-radius: 16px; background: rgba(255,255,255,.9); box-shadow: 0 12px 30px rgba(26,66,59,.05); }.demo-panel-title { display: grid; gap: 5px; margin-bottom: 20px; }.demo-panel-title span, .demo-section-heading span, .demo-chat-head span { color: #79918c; font-size: 10px; font-weight: 800; letter-spacing: .12em; text-transform: uppercase; }.demo-panel-title strong { color: #263a35; font-size: 18px; }.demo-channel { margin: 4px -8px 12px; padding: 3px 8px 9px; border-radius: 10px; }.demo-channel.active { background: #e2f3ef; }.demo-channel > button { display: flex; align-items: center; gap: 7px; width: 100%; padding: 8px 0; color: #45635b; background: transparent; border: 0; text-align: left; cursor: pointer; }.demo-channel.active > button { color: #006a64; font-weight: 800; }.demo-channel button span { flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }.demo-channel button small { color: #84a19a; }.demo-channel-members { display: grid; gap: 7px; padding-left: 22px; color: #7b8e88; font-size: 11px; }.demo-channel-members span { display: flex; align-items: center; gap: 7px; }.demo-channel-members i, .demo-user > i, .demo-voice-card > i, .demo-message > i { display: grid; place-items: center; flex: 0 0 auto; color: #fff; border-radius: 50%; font-style: normal; font-weight: 800; }.demo-channel-members i { width: 20px; height: 20px; font-size: 9px; }.demo-channel-members span.speaking i { box-shadow: 0 0 0 2px #83e88d, 0 0 10px rgba(83,214,105,.45); }
.demo-main { padding: 27px 32px; }.demo-hero { display: flex; align-items: center; justify-content: space-between; min-height: 190px; padding: 26px 29px; overflow: hidden; border-radius: 14px; background: linear-gradient(112deg, #dff3ef, #f6fbfa); }.demo-live { display: inline-flex; align-items: center; gap: 7px; padding: 5px 9px; color: #2d7e48; background: #d4f3d9; border-radius: 999px; font-size: 10px; font-weight: 700; }.demo-live i, .demo-green-dot { width: 7px; height: 7px; background: #58d476; border-radius: 50%; }.demo-hero h1 { display: flex; align-items: center; gap: 8px; margin: 17px 0 7px; color: #18302c; font-size: 28px; letter-spacing: -.05em; }.demo-hero p { margin: 0 0 15px; color: #64817a; font-size: 12px; }.demo-hero small { display: flex; align-items: center; gap: 6px; color: #52716b; font-size: 11px; }.demo-wave { display: flex; align-items: center; gap: 5px; height: 60px; padding-right: 22px; }.demo-wave i { width: 4px; border-radius: 4px; background: #65cabe; }.demo-wave i:nth-child(3n) { background: #8fe990; }.demo-section-heading { display: grid; gap: 5px; margin: 31px 0 15px; }.demo-section-heading strong { color: #1d2c28; font-size: 21px; }.demo-voice-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(125px, 1fr)); gap: 11px; }.demo-voice-card { display: grid; justify-items: center; gap: 7px; padding: 15px 8px; border: 1px solid #edf1ef; border-radius: 12px; background: #fff; cursor: pointer; }.demo-voice-card > i { width: 56px; height: 56px; font-size: 18px; }.demo-voice-card strong { color: #2c3935; font-size: 12px; }.demo-voice-card span { color: #91a09a; font-size: 10px; }.demo-voice-card.speaking { border-color: #83e88d; box-shadow: 0 0 0 2px rgba(131,232,141,.18), 0 8px 20px rgba(68,173,79,.12); }.demo-voice-card.speaking span { color: #2e8b45; }.demo-chat-head { display: flex; align-items: flex-end; justify-content: space-between; gap: 12px; margin-top: 33px; padding-top: 23px; border-top: 1px solid #edf2f0; }.demo-chat-head strong { display: block; margin-top: 6px; color: #1d2c28; font-size: 20px; }.demo-tabs { display: flex; gap: 5px; }.demo-tabs button { padding: 7px 9px; color: #78908a; background: #f0f6f3; border: 0; border-radius: 7px; cursor: pointer; }.demo-tabs button.active { color: #006a64; background: #dff1ed; font-weight: 700; }.demo-messages { display: grid; align-content: start; gap: 13px; min-height: 215px; max-height: 270px; margin-top: 17px; overflow-y: auto; padding: 7px 3px; }.demo-message { display: flex; align-items: flex-start; gap: 9px; max-width: 76%; }.demo-message.mine { justify-self: end; flex-direction: row-reverse; text-align: right; }.demo-message > i { width: 29px; height: 29px; font-size: 10px; }.demo-message small { color: #9ca9a4; font-size: 9px; }.demo-message p { margin: 4px 0 0; padding: 8px 11px; color: #43534e; background: #f1f5f3; border-radius: 4px 11px 11px 11px; font-size: 12px; }.demo-message.mine p { color: #fff; background: #006a64; border-radius: 11px 4px 11px 11px; }.demo-empty { display: grid; place-content: center; justify-items: center; gap: 8px; min-height: 170px; color: #91a29b; }.demo-empty strong { font-size: 12px; }.demo-composer { display: flex; gap: 8px; padding: 7px 8px 7px 12px; background: #f1f6f4; border-radius: 10px; }.demo-composer input { min-width: 0; flex: 1; border: 0; outline: 0; background: transparent; font-size: 12px; }.demo-composer button { display: grid; place-items: center; width: 34px; height: 34px; color: #fff; background: #006a64; border: 0; border-radius: 8px; cursor: pointer; }.demo-composer button:disabled { opacity: .35; cursor: not-allowed; }
.demo-actions-panel { display: flex; flex-direction: column; gap: 9px; }.demo-actions-panel > button { display: flex; align-items: center; gap: 8px; padding: 10px; color: #4d6a62; background: #f0f6f3; border: 1px solid #e1ece8; border-radius: 8px; text-align: left; cursor: pointer; }.demo-actions-panel > button:hover { color: #006a64; border-color: #b9ddd6; }.demo-note { display: flex; gap: 8px; margin-top: 10px; padding: 11px; color: #68837b; background: #eef8f4; border-radius: 9px; font-size: 10px; line-height: 1.5; }.demo-user { display: flex; align-items: center; gap: 9px; margin-top: auto; padding-top: 15px; border-top: 1px solid #e9efed; }.demo-user > i { width: 35px; height: 35px; font-size: 12px; }.demo-user div { min-width: 0; flex: 1; }.demo-user strong, .demo-user small { display: block; }.demo-user strong { color: #34423d; font-size: 12px; }.demo-user small { margin-top: 4px; color: #8a9a94; font-size: 10px; }.demo-poke { position: fixed; z-index: 10; top: 82px; right: 25px; display: flex; align-items: center; gap: 9px; max-width: min(380px, calc(100% - 32px)); padding: 11px 12px; color: #52645c; background: #fffdf6; border: 1px solid #f0dfbd; border-radius: 9px; box-shadow: 0 8px 22px rgba(88,65,28,.12); font-size: 12px; }.demo-poke span { flex: 1; }.demo-poke strong { color: #8f6532; }.demo-poke button { display: grid; place-items: center; padding: 3px; color: #9b8a6e; background: transparent; border: 0; cursor: pointer; }
@media (max-width: 1080px) { .demo-shell { grid-template-columns: 210px minmax(0, 1fr); }.demo-actions-panel { grid-column: 1 / -1; min-height: 160px; }.demo-actions-panel .demo-panel-title { margin-bottom: 3px; }.demo-actions-panel > button { display: inline-flex; width: fit-content; }.demo-note { margin-top: 4px; }.demo-user { margin-top: 0; } }
@media (max-width: 720px) { .demo-page { padding: 0 14px 24px; }.demo-header { min-height: 68px; }.demo-tools { gap: 6px; font-size: 10px; }.demo-badge { display: none; }.demo-shell { display: block; min-height: 0; }.demo-channel-panel { margin-bottom: 12px; }.demo-channel-panel .demo-channel:not(.active) { display: none; }.demo-actions-panel { margin-top: 12px; }.demo-main { padding: 17px 14px; }.demo-hero { min-height: 170px; padding: 20px; }.demo-wave { padding-right: 0; transform: scale(.75); transform-origin: right center; }.demo-chat-head { display: grid; align-items: start; }.demo-tabs { overflow-x: auto; }.demo-message { max-width: 94%; } }
@media (max-width: 360px) { .demo-tools a { display: none; }.demo-hero h1 { font-size: 23px; }.demo-hero p { font-size: 11px; }.demo-voice-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); } }
/* The whole app is zoomed via #app (zoom:var(--ui-scale)) on large displays;
   divide the fixed poke banner offsets back to CSS pixels so it stays at the
   intended 82px/25px margin instead of drifting by the zoom factor. */
@media (min-width: 851px) { .demo-poke { top: calc(82px / var(--ui-scale)); right: calc(25px / var(--ui-scale)); } }
.demo-page[data-ws-skin="builtin.dark"] { color: #e8f3f0; background: #101918; }
.demo-page[data-ws-skin="builtin.dark"] .demo-channel-panel, .demo-page[data-ws-skin="builtin.dark"] .demo-actions-panel, .demo-page[data-ws-skin="builtin.dark"] .demo-main, .demo-page[data-ws-skin="builtin.dark"] .demo-voice-card { background: #172321; border-color: #30413d; }
.demo-page[data-ws-skin="builtin.dark"] .demo-panel-title strong, .demo-page[data-ws-skin="builtin.dark"] .demo-hero h1, .demo-page[data-ws-skin="builtin.dark"] .demo-section-heading strong, .demo-page[data-ws-skin="builtin.dark"] .demo-chat-head strong, .demo-page[data-ws-skin="builtin.dark"] .demo-voice-card strong, .demo-page[data-ws-skin="builtin.dark"] .demo-user strong { color: #e8f3f0; }
.demo-page[data-ws-skin="builtin.dark"] .demo-hero { background: linear-gradient(112deg, #173e3a, #172321); }
.demo-page[data-ws-skin="builtin.dark"] .demo-message p, .demo-page[data-ws-skin="builtin.dark"] .demo-composer, .demo-page[data-ws-skin="builtin.dark"] .demo-actions-panel > button { background: #202f2c; color: #d7e7e3; }
.demo-page[data-ws-skin="builtin.dark"] .demo-chat-head, .demo-page[data-ws-skin="builtin.dark"] .demo-user { border-color: #30413d; }

.demo-page[data-ws-skin="builtin.discord"] {
  color: #dbdee1;
  background: #1e1f22;
  padding: 0;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.demo-page[data-ws-skin="builtin.discord"] .demo-header {
  width: 100%;
  max-width: 100%;
  padding: 0 16px;
  min-height: 48px;
  height: 48px;
  background: #1e1f22;
  border-bottom: 1px solid #111214;
}

.demo-page[data-ws-skin="builtin.discord"] .demo-shell {
  display: grid;
  grid-template-columns: 72px 240px minmax(0, 1fr) 420px;
  gap: 0;
  width: 100%;
  max-width: 100%;
  flex: 1 1 auto;
  min-height: calc(100vh - 48px);
  height: calc(100vh - 48px);
  border-radius: 0;
  border: none;
  overflow: hidden;
  box-shadow: none;
}
.demo-page[data-ws-skin="builtin.discord"] .demo-channel-panel, .demo-page[data-ws-skin="builtin.discord"] .demo-actions-panel, .demo-page[data-ws-skin="builtin.discord"] .demo-voice-card { background: #2b2d31; border-color: #1f2023; }
.demo-page[data-ws-skin="builtin.discord"] .demo-main { background: #313338; border-color: #1f2023; }
.demo-page[data-ws-skin="builtin.discord"] .demo-panel-title strong, .demo-page[data-ws-skin="builtin.discord"] .demo-hero h1, .demo-page[data-ws-skin="builtin.discord"] .demo-section-heading strong, .demo-page[data-ws-skin="builtin.discord"] .demo-chat-head strong, .demo-page[data-ws-skin="builtin.discord"] .demo-voice-card strong, .demo-page[data-ws-skin="builtin.discord"] .demo-user strong { color: #f2f3f5; }
.demo-page[data-ws-skin="builtin.discord"] .demo-hero { background: linear-gradient(112deg, #2b2d31, #1e1f22); }
.demo-page[data-ws-skin="builtin.discord"] .demo-message p { background: #2b2d31; color: #dbdee1; border-radius: 4px; }
.demo-page[data-ws-skin="builtin.discord"] .demo-message.mine p { background: #5865f2; color: #ffffff; border-radius: 4px; }
.demo-page[data-ws-skin="builtin.discord"] .demo-composer { background: #383a40; }
.demo-page[data-ws-skin="builtin.discord"] .demo-composer input { color: #f2f3f5; }
.demo-page[data-ws-skin="builtin.discord"] .demo-composer button { background: #5865f2; }
.demo-page[data-ws-skin="builtin.discord"] .demo-actions-panel > button { background: #383a40; border-color: #1f2023; color: #dbdee1; }
.demo-page[data-ws-skin="builtin.discord"] .demo-actions-panel > button:hover { background: #35373c; color: #f2f3f5; }
.demo-page[data-ws-skin="builtin.discord"] .demo-chat-head, .demo-page[data-ws-skin="builtin.discord"] .demo-user { border-color: #1f2023; }
.demo-page[data-ws-skin="builtin.discord"] .demo-channel.active { background: #3b3e45; }
.demo-page[data-ws-skin="builtin.discord"] .demo-channel > button { color: #949ba4; }
.demo-page[data-ws-skin="builtin.discord"] .demo-channel.active > button { color: #f2f3f5; }
.demo-page[data-ws-skin="builtin.discord"] .demo-channel-members { color: #949ba4; }
.demo-page[data-ws-skin="builtin.discord"] .demo-channel-members span.speaking i { box-shadow: 0 0 0 2px #23a55a, 0 0 10px rgba(35, 165, 90, 0.65); }
.demo-page[data-ws-skin="builtin.discord"] .demo-voice-card.speaking { border-color: #23a55a; box-shadow: 0 0 0 2px rgba(35, 165, 90, 0.45), 0 8px 20px rgba(0, 0, 0, 0.4); }
.demo-page[data-ws-skin="builtin.discord"] .demo-voice-card.speaking span { color: #23a55a; }
.demo-page[data-ws-skin="builtin.discord"] .demo-tabs button { background: #2b2d31; color: #949ba4; }
.demo-page[data-ws-skin="builtin.discord"] .demo-tabs button.active { background: #383a40; color: #f2f3f5; }

.demo-page[data-ws-skin="builtin.discord"] .demo-tools a {
  background: #2b2d31;
  border-color: #1f2023;
  color: #dbdee1;
}
.demo-page[data-ws-skin="builtin.discord"] .demo-tools a:hover {
  background: #35373c;
  color: #f2f3f5;
}
.demo-page[data-ws-skin="builtin.discord"] .demo-badge {
  color: #23a55a;
}
.demo-page[data-ws-skin="builtin.discord"] .demo-header .demo-brand strong {
  color: #f2f3f5;
}
.demo-page[data-ws-skin="builtin.discord"] .demo-header .demo-brand span {
  background: #5865f2;
}

.demo-page[data-ws-skin="builtin.discord"] .demo-message {
  max-width: 100%;
  padding: 4px 10px;
  border-radius: 4px;
  transition: background-color 0.1s ease;
}
.demo-page[data-ws-skin="builtin.discord"] .demo-message:hover {
  background-color: rgba(255, 255, 255, 0.03);
}
.demo-page[data-ws-skin="builtin.discord"] .demo-message.mine {
  justify-self: start;
  flex-direction: row;
  text-align: left;
}
.demo-page[data-ws-skin="builtin.discord"] .demo-message p,
.demo-page[data-ws-skin="builtin.discord"] .demo-message.mine p {
  background: transparent;
  color: #dbdee1;
  padding: 0;
  border-radius: 0;
  font-size: 14px;
  line-height: 1.4;
}
.demo-page[data-ws-skin="builtin.discord"] .demo-message small {
  color: #949ba4;
  font-size: 11px;
}

/* Discord Guild Navigation Rail (Leftmost column) */
.demo-page[data-ws-skin="builtin.discord"] .demo-guild-rail {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px 0;
  gap: 8px;
  background: #1e1f22;
  border-right: 1px solid #111214;
  overflow-y: auto;
  position: relative;
}

.demo-guild-rail .guild-pill-active {
  position: absolute;
  left: 0;
  top: 16px;
  width: 4px;
  height: 40px;
  background: #ffffff;
  border-radius: 0 4px 4px 0;
}

.demo-guild-rail .guild-item {
  position: relative;
  display: flex;
  justify-content: center;
  width: 100%;
}

.demo-guild-rail .guild-icon {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  border: none;
  background: #313338;
  color: #dbdee1;
  display: grid;
  place-items: center;
  cursor: pointer;
  font-weight: 700;
  font-size: 14px;
  transition: border-radius 0.2s ease, background-color 0.2s ease, color 0.2s ease;
}

.demo-guild-rail .guild-icon:hover {
  border-radius: 16px;
  background: #5865f2;
  color: #ffffff;
}

.demo-guild-rail .guild-icon.active {
  border-radius: 16px;
  background: #5865f2;
  color: #ffffff;
}

.demo-guild-rail .guild-icon.guild-action {
  color: #23a55a;
  background: #313338;
}

.demo-guild-rail .guild-icon.guild-action:hover {
  background: #23a55a;
  color: #ffffff;
}

.demo-guild-rail .guild-separator {
  width: 32px;
  height: 2px;
  background: #35373c;
  border-radius: 1px;
  margin: 4px 0;
}

.demo-guild-rail .guild-badge {
  position: absolute;
  bottom: -2px;
  right: 8px;
  background: #f23f43;
  color: #ffffff;
  font-size: 11px;
  font-weight: 700;
  padding: 1px 5px;
  border-radius: 10px;
  border: 2px solid #1e1f22;
}

/* Discord Sidebar Top Search & Bottom User Bar */
.discord-sidebar-search {
  padding: 8px 10px 14px;
  border-bottom: 1px solid #1f2023;
  margin-bottom: 12px;
}

.discord-search-btn {
  width: 100%;
  height: 32px;
  background: #1e1f22;
  color: #949ba4;
  border: 1px solid #1f2023;
  border-radius: 4px;
  font-size: 12px;
  text-align: left;
  padding: 0 10px;
  cursor: pointer;
  display: flex;
  align-items: center;
  transition: color 0.15s ease, background 0.15s ease;
}

.discord-search-btn:hover {
  color: #dbdee1;
  background: #232428;
}

.discord-user-info {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 0;
  flex: 1;
}

.discord-avatar-wrap {
  position: relative;
  flex: 0 0 auto;
}

.discord-avatar-wrap > i {
  display: grid;
  place-items: center;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  color: #fff;
  font-style: normal;
  font-weight: 700;
  font-size: 13px;
}

.discord-status-dot {
  position: absolute;
  bottom: -2px;
  right: -2px;
  width: 10px;
  height: 10px;
  background: #23a55a;
  border: 2px solid #232428;
  border-radius: 50%;
}

.discord-user-text {
  min-width: 0;
  flex: 1;
  line-height: 1.2;
}

.discord-user-text strong {
  display: block;
  font-size: 13px;
  color: #f2f3f5;
  font-weight: 600;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.discord-user-text small {
  display: block;
  font-size: 10px;
  color: #949ba4;
}

.discord-user-actions {
  display: flex;
  align-items: center;
  gap: 2px;
}

.discord-icon-btn {
  width: 30px;
  height: 30px;
  border-radius: 4px;
  border: none;
  background: transparent;
  color: #b5bac1;
  display: grid;
  place-items: center;
  cursor: pointer;
  transition: background 0.15s ease, color 0.15s ease;
}

.discord-icon-btn:hover {
  background: #35373c;
  color: #dbdee1;
}

.discord-icon-btn.muted {
  color: #f23f43;
}

/* Discord Server Header Card */
.discord-server-card {
  margin: -12px -8px 8px -8px;
  padding: 12px 16px;
  border-bottom: 1px solid #1f2023;
  cursor: pointer;
  transition: background 0.15s ease;
}

.discord-server-card:hover {
  background: rgba(255, 255, 255, 0.04);
}

.discord-server-card-content {
  display: flex;
  align-items: center;
  gap: 8px;
}

.discord-server-card-avatar {
  font-size: 16px;
}

.discord-server-card-content strong {
  flex: 1;
  font-size: 15px;
  font-weight: 700;
  color: #f2f3f5;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.discord-server-chevron {
  color: #949ba4;
}

.discord-voice-category-heading {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 8px 4px 4px 4px;
  color: #949ba4;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.02em;
}

/* Discord Channels Scroll Area */
.discord-channels-scroll-area {
  flex: 1 1 auto;
  overflow-y: auto;
  min-height: 0;
  margin: 0 -4px;
  padding: 0 4px;
}

/* Discord Connected Strip above user bar */
.discord-connected-strip {
  flex: 0 0 auto;
  margin-left: -8px;
  margin-right: -8px;
  padding: 8px 10px;
  background: #111214;
  border-top: 1px solid #1f2023;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.discord-user-bar {
  flex: 0 0 auto;
  margin-left: -8px;
  margin-right: -8px;
  margin-bottom: 0;
  padding: 8px 10px;
  background: #232428;
  border-top: 1px solid #1f2023;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.discord-connected-left {
  display: flex;
  align-items: center;
  gap: 8px;
}

.discord-connected-signal {
  color: #23a55a;
}

.discord-connected-copy strong {
  display: block;
  font-size: 13px;
  font-weight: 700;
  color: #23a55a;
  line-height: 1.2;
}

.discord-connected-copy small {
  display: block;
  font-size: 10px;
  color: #949ba4;
  line-height: 1.2;
}

.discord-disconnect-btn {
  background: transparent;
  border: none;
  color: #f23f43;
  width: 28px;
  height: 28px;
  border-radius: 4px;
  display: grid;
  place-items: center;
  cursor: pointer;
  transition: background 0.15s ease;
}

.discord-disconnect-btn:hover {
  background: rgba(242, 63, 67, 0.15);
}

.demo-page[data-ws-skin="builtin.discord"] .demo-channel-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
  min-height: 0;
  border-radius: 0;
  border-right: 1px solid #1f2023;
  border-top: none;
  border-bottom: none;
  border-left: none;
  padding: 12px 8px 0;
  background: #2b2d31;
}

/* Discord Center Voice Stage Styles */
.demo-page[data-ws-skin="builtin.discord"] .demo-main {
  display: flex;
  flex-direction: column;
  border-radius: 0;
  border: none;
  background: #111214;
  position: relative;
  overflow: hidden;
  height: 100%;
}

.discord-stage-header {
  height: 48px;
  min-height: 48px;
  padding: 0 16px;
  background: #313338;
  border-bottom: 1px solid #1f2023;
  display: flex;
  align-items: center;
  justify-content: space-between;
  z-index: 10;
}

.discord-channel-title {
  display: flex;
  align-items: center;
  gap: 8px;
}

.discord-channel-title .discord-channel-icon {
  color: #80848e;
}

.discord-channel-title h2 {
  font-size: 15px;
  font-weight: 700;
  color: #f2f3f5;
  margin: 0;
}

.discord-stage-header-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.discord-header-btn {
  width: 32px;
  height: 32px;
  border-radius: 4px;
  border: none;
  background: transparent;
  color: #b5bac1;
  display: grid;
  place-items: center;
  cursor: pointer;
}

.discord-header-btn:hover {
  background: #35373c;
  color: #dbdee1;
}

.discord-header-btn.active {
  color: #f2f3f5;
}

.discord-stage-canvas {
  flex: 1 1 auto;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 24px;
  position: relative;
  background: #111214;
  min-height: 0;
}

.discord-speakers-grid {
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr 1fr;
  gap: 16px;
  width: 100%;
  max-width: 820px;
  height: 80%;
  max-height: 640px;
}

.discord-speaker-card {
  position: relative;
  background: linear-gradient(145deg, #2b3a70 0%, #1e2646 100%);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
  transition: box-shadow 0.2s ease;
}

.discord-speaker-card.speaking {
  box-shadow: 0 0 0 2px #23a55a, 0 0 16px rgba(35, 165, 90, 0.6);
}

.discord-speaker-avatar i {
  display: grid;
  place-items: center;
  width: 88px;
  height: 88px;
  border-radius: 50%;
  background: #5865f2;
  color: #ffffff;
}

.discord-speaker-tag {
  position: absolute;
  bottom: 12px;
  left: 12px;
  background: rgba(0, 0, 0, 0.6);
  padding: 4px 10px;
  border-radius: 4px;
  backdrop-filter: blur(4px);
}

.discord-speaker-name {
  font-size: 13px;
  font-weight: 600;
  color: #f2f3f5;
}

.stage-artwork-card {
  background: linear-gradient(145deg, #2b1836 0%, #181122 100%);
  flex-direction: column;
  gap: 16px;
}

.discord-stage-artwork {
  font-size: 42px;
  display: flex;
  gap: 12px;
}

.discord-stage-invite-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #f2f3f5;
  border-radius: 4px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  backdrop-filter: blur(4px);
  transition: background 0.15s ease;
}

.discord-stage-invite-btn:hover {
  background: rgba(255, 255, 255, 0.1);
}

.discord-floating-controls {
  position: absolute;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: center;
  gap: 12px;
  background: #1e1f22;
  padding: 8px 16px;
  border-radius: 28px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.6);
  border: 1px solid #2b2d31;
}

.stage-ctrl-btn {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: none;
  background: #313338;
  color: #dbdee1;
  display: grid;
  place-items: center;
  cursor: pointer;
  transition: background 0.15s ease, color 0.15s ease;
}

.stage-ctrl-btn:hover {
  background: #3b3e45;
  color: #ffffff;
}

.stage-ctrl-btn.danger {
  color: #f23f43;
}

.stage-ctrl-btn.danger-disconnect {
  background: #da373c;
  color: #ffffff;
}

.stage-ctrl-btn.danger-disconnect:hover {
  background: #f23f43;
}

/* Discord Right Column: Channel Chat Panel */
.demo-page[data-ws-skin="builtin.discord"] .demo-actions-panel {
  display: flex;
  flex-direction: column;
  border-radius: 0;
  border-left: 1px solid #1f2023;
  border-top: none;
  border-bottom: none;
  border-right: none;
  padding: 0;
  background: #313338;
  height: 100%;
  min-height: 0;
}

.discord-chat-sidebar-header {
  height: 48px;
  min-height: 48px;
  padding: 0 16px;
  border-bottom: 1px solid #1f2023;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #313338;
}

.discord-chat-title {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #f2f3f5;
  font-size: 15px;
}

.discord-chat-close-btn {
  background: transparent;
  border: none;
  color: #949ba4;
  cursor: pointer;
  display: grid;
  place-items: center;
}

.discord-chat-close-btn:hover {
  color: #f2f3f5;
}

.discord-channel-chat-stream {
  flex: 1 1 auto;
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.discord-chat-welcome {
  margin-top: 16px;
  margin-bottom: 8px;
}

.discord-welcome-hash-circle {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  background: #404249;
  color: #ffffff;
  display: grid;
  place-items: center;
  margin-bottom: 8px;
}

.discord-chat-welcome h2 {
  font-size: 24px;
  font-weight: 700;
  color: #f2f3f5;
  margin: 0 0 4px 0;
}

.discord-chat-welcome p {
  font-size: 13px;
  color: #949ba4;
  margin: 0;
}

.discord-date-divider {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin: 12px 0;
}

.discord-date-divider::before {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  height: 1px;
  background: #3f4147;
}

.discord-date-divider span {
  position: relative;
  background: #313338;
  padding: 0 8px;
  font-size: 11px;
  font-weight: 600;
  color: #949ba4;
}

.discord-chat-row {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 4px 8px;
  border-radius: 4px;
  transition: background 0.1s ease;
}

.discord-chat-row:hover {
  background: rgba(255, 255, 255, 0.03);
}

.discord-row-avatar i {
  display: grid;
  place-items: center;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  color: #fff;
  font-style: normal;
  font-weight: 700;
  font-size: 15px;
}

.discord-row-content {
  flex: 1 1 auto;
  min-width: 0;
}

.discord-row-header {
  display: flex;
  align-items: baseline;
  gap: 6px;
  margin-bottom: 2px;
}

.discord-row-author {
  font-size: 14px;
  font-weight: 600;
  color: #f2f3f5;
}

.discord-bot-badge {
  background: #5865f2;
  color: #ffffff;
  font-size: 9px;
  font-weight: 700;
  padding: 1px 4px;
  border-radius: 3px;
}

.discord-row-time {
  font-size: 11px;
  color: #949ba4;
}

.discord-row-text {
  font-size: 14px;
  color: #dbdee1;
  margin: 0;
  line-height: 1.4;
  word-break: break-word;
}

.discord-channel-composer {
  padding: 12px 16px 20px;
  background: #313338;
}

.discord-composer-wrap {
  display: flex;
  align-items: center;
  background: #383a40;
  border-radius: 8px;
  padding: 0 12px;
  height: 44px;
}

.discord-composer-plus {
  background: #4e5058;
  border: none;
  border-radius: 50%;
  color: #313338;
  width: 24px;
  height: 24px;
  display: grid;
  place-items: center;
  cursor: pointer;
  margin-right: 12px;
}

.discord-composer-wrap input {
  flex: 1 1 auto;
  background: transparent;
  border: none;
  outline: none;
  color: #f2f3f5;
  font-size: 14px;
}

.discord-composer-wrap input::placeholder {
  color: #80848e;
}

.discord-composer-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.discord-composer-btn {
  background: transparent;
  border: none;
  color: #b5bac1;
  cursor: pointer;
  display: grid;
  place-items: center;
}

.discord-composer-btn:hover {
  color: #dbdee1;
}

.discord-composer-send {
  background: #5865f2;
  border: none;
  border-radius: 4px;
  color: #ffffff;
  width: 28px;
  height: 28px;
  display: grid;
  place-items: center;
  cursor: pointer;
}

.discord-composer-send:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

/* Discord screenshare preview card in stage */
.discord-screenshare-preview {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  width: 100%;
  height: 100%;
  color: #5865f2;
}

.discord-screenshare-preview span {
  font-size: 13px;
  font-weight: 600;
  color: #dbdee1;
}

/* Discord Settings Modal Simulation */
.discord-settings-overlay {
  position: fixed;
  inset: 0;
  z-index: 100;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
}

.discord-settings-modal {
  width: min(560px, calc(100% - 32px));
  background: #313338;
  border-radius: 8px;
  box-shadow: 0 16px 36px rgba(0, 0, 0, 0.6);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.discord-settings-modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid #1f2023;
}

.discord-settings-modal-header h3 {
  margin: 0;
  font-size: 17px;
  font-weight: 700;
  color: #f2f3f5;
}

.discord-settings-modal-header button {
  background: transparent;
  border: none;
  color: #b5bac1;
  cursor: pointer;
  display: grid;
  place-items: center;
  padding: 4px;
  border-radius: 4px;
}

.discord-settings-modal-header button:hover {
  color: #f2f3f5;
  background: #35373c;
}

.discord-settings-modal-body {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.discord-settings-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 14px;
  background: #2b2d31;
  border-radius: 6px;
}

.discord-settings-row-info strong {
  display: block;
  font-size: 14px;
  color: #f2f3f5;
  margin-bottom: 2px;
}

.discord-settings-row-info small {
  display: block;
  font-size: 12px;
  color: #949ba4;
}

.discord-toggle-switch {
  width: 44px;
  height: 24px;
  background: #80848e;
  border-radius: 12px;
  position: relative;
  cursor: pointer;
  transition: background 0.2s ease;
}

.discord-toggle-switch.on {
  background: #23a55a;
}

.discord-toggle-knob {
  position: absolute;
  top: 3px;
  left: 3px;
  width: 18px;
  height: 18px;
  background: #ffffff;
  border-radius: 50%;
  transition: transform 0.2s ease;
}

.discord-toggle-switch.on .discord-toggle-knob {
  transform: translateX(20px);
}

.discord-settings-modal-footer {
  padding: 14px 20px;
  background: #2b2d31;
  display: flex;
  justify-content: flex-end;
  border-top: 1px solid #1f2023;
}

.discord-settings-done-btn {
  background: #5865f2;
  color: #ffffff;
  border: none;
  border-radius: 4px;
  padding: 8px 18px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s ease;
}

.discord-settings-done-btn:hover {
  background: #4752c4;
}

/* Responsive overrides specifically for Discord Skin */
@media (max-width: 1080px) {
  .demo-page[data-ws-skin="builtin.discord"] .demo-shell {
    grid-template-columns: 72px 220px minmax(0, 1fr) 340px;
  }
}

@media (max-width: 900px) {
  .demo-page[data-ws-skin="builtin.discord"] .demo-shell {
    grid-template-columns: 72px 220px minmax(0, 1fr);
  }
  .demo-page[data-ws-skin="builtin.discord"] .demo-actions-panel {
    display: none;
  }
}

@media (max-width: 640px) {
  .demo-page[data-ws-skin="builtin.discord"] .demo-shell {
    grid-template-columns: 60px minmax(0, 1fr);
  }
  .demo-page[data-ws-skin="builtin.discord"] .demo-channel-panel {
    display: none;
  }
}

.demo-page :where(button, a, input, select):focus-visible { outline: 3px solid #69d2c7; outline-offset: 2px; }
.demo-page[data-ws-skin="builtin.discord"] :where(button, a, input, select):focus-visible { outline-color: #5865f2; }
</style>
