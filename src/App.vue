<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref, watch } from "vue";

const categories = ["全部", "老虎機", "捕魚機", "棋牌", "小遊戲"];
const category = ref("全部");
const sort = ref("熱門");
const query = ref("");
const accountOpen = ref(false);
const modal = ref(null);
const nickname = ref("Rin Chen");
const supportTab = ref("submit");
const toast = ref("");
const searchOpen = ref(false);
const showMobileDock = ref(false);
const categoryNav = ref(null);
const nicknameInput = ref(null);
const recordStart = ref("2026-04-18");
const recordEnd = ref("2026-07-18");
const recordCategory = ref("");
const selectedRecordGames = ref([]);
const recordGameMenuOpen = ref(false);
const visibleGameCount = ref(8);
const avatarOptions = [
  { id: "aurora", glyph: "✦", tone: "avatar-aurora", label: "極光" },
  { id: "coral", glyph: "●", tone: "avatar-coral", label: "珊瑚" },
  { id: "sky", glyph: "◈", tone: "avatar-sky", label: "晴空" },
  { id: "violet", glyph: "◆", tone: "avatar-violet", label: "紫晶" },
  { id: "mint", glyph: "✿", tone: "avatar-mint", label: "薄荷" },
  { id: "ember", glyph: "✹", tone: "avatar-ember", label: "燄火" },
  { id: "ocean", glyph: "◌", tone: "avatar-ocean", label: "深海" },
  { id: "sun", glyph: "☼", tone: "avatar-sun", label: "晨光" },
];
const selectedAvatarId = ref("aurora");
const avatarDraftId = ref(selectedAvatarId.value);

const gameCover = (fileName) => `${import.meta.env.BASE_URL}games/${fileName}`;

const games = [
  { id: 1, name: "翡翠天后", category: "老虎機", cover: gameCover("jade-empress.jpg"), icon: "翡", description: "踏入雲上神殿，蒐集翡翠符印，啟動天后祝福與連線獎勵。", volatility: "高", rtp: "96.42%", rtpTrend: "up", multiplier: "x2,000", status: "新上", statusTone: "new" },
  { id: 2, name: "深海龍王", category: "捕魚機", cover: gameCover("azure-dragon.jpg"), icon: "龍", description: "以深海寶藏為目標，鎖定巨龍後可開啟限時加成與團隊獎池。", volatility: "中高", rtp: "97.10%", rtpTrend: "down", multiplier: "x1,500" },
  { id: 3, name: "赤焰麻將", category: "棋牌", cover: gameCover("red-mahjong.jpg"), icon: "萬", description: "經典麻將節奏結合連莊機制，兼具策略與快速對局的爽感。", volatility: "中", rtp: "96.80%", rtpTrend: "up", multiplier: "x800" },
  { id: 4, name: "星際躍升", category: "小遊戲", cover: gameCover("neon-rocket.jpg"), icon: "UP", description: "在倒數結束前選擇撤離時機，穿越星門取得即時倍率。", volatility: "高", rtp: "95.90%", rtpTrend: "down", multiplier: "x10,000", status: "待上", statusTone: "upcoming" },
  { id: 5, name: "天宮寶藏", category: "老虎機", cover: gameCover("jade-empress.jpg"), icon: "寶", description: "集滿三枚天印進入寶藏回合，獲得自由旋轉與隨機倍數。", volatility: "中高", rtp: "96.20%", rtpTrend: "up", multiplier: "x1,200" },
  { id: 6, name: "珊瑚戰線", category: "捕魚機", cover: gameCover("azure-dragon.jpg"), icon: "海", description: "在移動砲台間切換火力，捕捉高價值魚群與海底首領。", volatility: "中", rtp: "97.30%", rtpTrend: "down", multiplier: "x900" },
  { id: 7, name: "金牌十三張", category: "棋牌", cover: gameCover("red-mahjong.jpg"), icon: "13", description: "以手牌布局創造最佳組合，支援快速配桌與好友牌局。", volatility: "中低", rtp: "97.05%", rtpTrend: "up", multiplier: "x300" },
  { id: 8, name: "量子翻翻樂", category: "小遊戲", cover: gameCover("neon-rocket.jpg"), icon: "Q", description: "簡潔的翻牌玩法，每一步都可能打開倍數與驚喜獎勵。", volatility: "中", rtp: "96.10%", rtpTrend: "down", multiplier: "x500", status: "熱門", statusTone: "hot" },
  { id: 9, name: "金曜財神", category: "老虎機", cover: gameCover("jade-empress.jpg"), icon: "金", description: "隨著金元寶連線累積財神加成，解鎖高倍率獎勵回合。", volatility: "高", rtp: "96.66%", rtpTrend: "up", multiplier: "x3,000", status: "維護", statusTone: "maintenance" },
  { id: 10, name: "深藍獵手", category: "捕魚機", cover: gameCover("azure-dragon.jpg"), icon: "鯊", description: "在限時深海關卡鎖定獵物，協作擊破首領並爭取額外分紅。", volatility: "中高", rtp: "96.35%", rtpTrend: "down", multiplier: "x1,000" },
  { id: 11, name: "百搭撲克", category: "棋牌", cover: gameCover("red-mahjong.jpg"), icon: "A", description: "多種牌型快速配對，支援快速配桌與好友牌局。", volatility: "中", rtp: "97.18%", rtpTrend: "up", multiplier: "x600" },
  { id: 12, name: "糖果衝刺", category: "小遊戲", cover: gameCover("neon-rocket.jpg"), icon: "GO", description: "收集糖果能量躲避障礙，將連擊推進到更高的獎勵區間。", volatility: "中低", rtp: "95.80%", rtpTrend: "down", multiplier: "x450" },
];

const records = [
  { name: "翡翠天后", category: "老虎機", date: "2026/07/18  04:42", dateValue: "2026-07-18", amount: "+ 2,480" },
  { name: "深海龍王", category: "捕魚機", date: "2026/07/18  03:58", dateValue: "2026-07-18", amount: "- 560" },
  { name: "星際躍升", category: "小遊戲", date: "2026/07/17  23:26", dateValue: "2026-07-17", amount: "+ 1,200" },
  { name: "赤焰麻將", category: "棋牌", date: "2026/07/17  20:10", dateValue: "2026-07-17", amount: "- 300" },
  { name: "天宮寶藏", category: "老虎機", date: "2026/07/17  18:04", dateValue: "2026-07-17", amount: "+ 800" },
];

const selectedGame = ref(games[0]);
const filteredGames = computed(() => {
  const phrase = query.value.trim().toLowerCase();
  const result = games.filter((game) => (category.value === "全部" || game.category === category.value) && (!phrase || game.name.toLowerCase().includes(phrase)));
  return sort.value === "熱門" ? result : [...result].sort((a, b) => b.id - a.id);
});
const visibleGames = computed(() => filteredGames.value.slice(0, visibleGameCount.value));
const hasMoreGames = computed(() => visibleGameCount.value < filteredGames.value.length);
const selectedAvatar = computed(() => avatarOptions.find((avatar) => avatar.id === selectedAvatarId.value) ?? avatarOptions[0]);
const recordGameOptions = computed(() => [...new Set(records.map((record) => record.name))]);
const filteredRecords = computed(() => records.filter((record) => (
  record.dateValue >= recordStart.value
  && record.dateValue <= recordEnd.value
  && (!recordCategory.value || record.category === recordCategory.value)
  && (!selectedRecordGames.value.length || selectedRecordGames.value.includes(record.name))
)));
const recordGameSummary = computed(() => {
  if (!selectedRecordGames.value.length) return "全部遊戲";
  if (selectedRecordGames.value.length === 1) return selectedRecordGames.value[0];
  return `已選 ${selectedRecordGames.value.length} 款`;
});

function openGameInfo(game) { selectedGame.value = game; accountOpen.value = false; modal.value = "game"; }
function showToast(message) { toast.value = message; window.setTimeout(() => { toast.value = ""; }, 2400); }
function focusSearch() { searchOpen.value = true; nextTick(() => document.querySelector(".search-field input")?.focus()); }
function focusNickname() { nicknameInput.value?.focus(); }
function openSupport() { accountOpen.value = false; supportTab.value = "submit"; modal.value = "support"; }
function openHistory() { accountOpen.value = false; recordGameMenuOpen.value = false; modal.value = "history"; }
function submitSupport() { modal.value = null; showToast("問題已送出，客服將盡快回覆"); }
function loadMoreGames() {
  visibleGameCount.value += 4;
  showToast("已加載更多遊戲");
}
function openAvatarPicker() {
  accountOpen.value = false;
  avatarDraftId.value = selectedAvatarId.value;
  modal.value = "avatar";
}
function saveAvatar() {
  selectedAvatarId.value = avatarDraftId.value;
  modal.value = null;
  showToast("頭像已更新");
}
function launchGame(game) {
  if (game.statusTone === "upcoming") {
    selectedGame.value = game;
    modal.value = "coming-soon";
    return;
  }
  modal.value = null;
  showToast(`${game.name} 已準備開啟`);
}

watch([category, sort, query], () => { visibleGameCount.value = 8; });

function updateMobileDock() {
  const nav = categoryNav.value;
  const topbar = document.querySelector(".topbar");
  if (!nav || !topbar || window.innerWidth > 640) {
    showMobileDock.value = false;
    return;
  }
  showMobileDock.value = nav.getBoundingClientRect().top < topbar.getBoundingClientRect().bottom;
}

onMounted(() => {
  updateMobileDock();
  window.addEventListener("scroll", updateMobileDock, { passive: true });
  window.addEventListener("resize", updateMobileDock);
});

onBeforeUnmount(() => {
  window.removeEventListener("scroll", updateMobileDock);
  window.removeEventListener("resize", updateMobileDock);
});
</script>

<template>
  <main class="lobby-shell">
    <div class="ambient ambient-a" />
    <div class="ambient ambient-b" />

    <header class="topbar">
      <a class="brand" href="#" aria-label="LUMEN Lobby 首頁">
        <span class="brand-mark"><i /><i /><i /></span>
        <span><b>LUMEN</b><small>GAME LOBBY</small></span>
      </a>

      <div class="topbar-actions">
        <div class="balance"><span>錢包餘額</span><div class="currency-badge"><span class="coin-token">NT</span><strong>1,000,000</strong><button class="balance-refresh" type="button" aria-label="重新整理錢包餘額" title="重新整理餘額" @click="showToast('餘額已更新')">↻</button></div></div>
        <div class="account-wrap">
          <button class="avatar-button" type="button" @click="accountOpen = !accountOpen" :aria-expanded="accountOpen" aria-label="開啟帳戶選單"><span :class="['avatar', selectedAvatar.tone]">{{ selectedAvatar.glyph }}</span><span class="account-caret">⌄</span></button>
          <section v-if="accountOpen" class="account-menu" aria-label="帳戶選單">
            <div class="account-profile"><button :class="['avatar', 'avatar-edit', selectedAvatar.tone]" type="button" aria-label="更換頭像" @click="openAvatarPicker">{{ selectedAvatar.glyph }}</button><div><small>系統帳號</small><strong>lumen_88321</strong></div></div>
            <label class="nickname-field"><span>暱稱</span><span class="nickname-control"><input ref="nicknameInput" v-model="nickname" maxlength="16" /><button class="edit-icon" type="button" aria-label="修改暱稱" title="修改暱稱" @click="focusNickname">✎</button></span></label>
            <button class="menu-row" type="button" @click="openHistory"><span>遊戲紀錄</span><b>›</b></button>
            <button class="menu-row" type="button" @click="openSupport"><span>客服中心</span><b>›</b></button>
            <a class="menu-row" href="https://www.taggame.online" target="_blank" rel="noreferrer"><span>聯絡我們</span><b>↗</b></a>
            <div class="language-row"><span>語系</span><button class="language-active" type="button">繁中</button><button type="button">EN</button><button type="button">日本語</button></div>
          </section>
        </div>
      </div>
    </header>

    <nav ref="categoryNav" class="category-nav" aria-label="遊戲類型">
      <button v-for="item in categories" :key="item" type="button" :class="{ active: category === item }" @click="category = item">{{ item }}</button>
    </nav>

    <section class="catalogue" aria-label="遊戲列表">
      <div class="catalogue-toolbar">
        <div class="sort-control" aria-label="排序"><button type="button" :class="{ selected: sort === '熱門' }" @click="sort = '熱門'">熱門</button><button type="button" :class="{ selected: sort === '最新' }" @click="sort = '最新'">最新</button></div>
        <label class="search-field" :class="{ 'mobile-search-open': searchOpen }"><span>⌕</span><input v-model="query" placeholder="搜尋遊戲" @blur="searchOpen = false" /><kbd>⌘ K</kbd></label>
      </div>
      <div class="catalogue-heading"><div><span class="eyebrow">GAME COLLECTION</span><h2>{{ category === '全部' ? '精選遊戲' : category }}</h2></div><span>{{ filteredGames.length }} 款遊戲</span></div>
      <div class="game-grid">
        <article v-for="game in visibleGames" :key="game.id" class="game-card">
          <button class="game-visual" type="button" :style="{ backgroundImage: `url(${game.cover})` }" :aria-label="`開啟 ${game.name}`" @click="launchGame(game)"><span v-if="game.status" :class="['game-status', `status-${game.statusTone}`]">{{ game.status }}</span><span class="launch-arrow">↗</span></button>
          <div class="game-panel"><div class="game-title-row"><span class="game-icon">{{ game.icon }}</span><div><h3>{{ game.name }}</h3><p>{{ game.category }}</p></div><button class="info-button" type="button" :aria-label="`查看 ${game.name} 說明`" @click="openGameInfo(game)">!</button></div><p class="game-description">{{ game.description }}</p><div class="game-metrics"><div class="metric"><strong>{{ game.volatility }}</strong><span>波動度</span></div><div class="metric"><strong :class="['rtp-pulse', game.rtpTrend]"><span>{{ game.rtp }}</span><b aria-hidden="true">{{ game.rtpTrend === 'down' ? '↘' : '↗' }}</b></strong><span>RTP</span></div><div class="metric"><strong>{{ game.multiplier }}</strong><span>最高倍率</span></div></div></div>
        </article>
      </div>
      <div v-if="filteredGames.length === 0" class="empty-state">沒有找到相符的遊戲，換個關鍵字試試。</div>
      <div v-else class="load-more-wrap"><button v-if="hasMoreGames" class="load-more-button" type="button" @click="loadMoreGames">加載更多遊戲 <b>↓</b></button><span v-else class="load-complete">已顯示全部 {{ filteredGames.length }} 款遊戲</span></div>
    </section>

    <nav v-show="showMobileDock" class="mobile-dock" aria-label="手機遊戲導航"><div class="mobile-categories"><button v-for="item in categories" :key="item" type="button" :class="{ active: category === item }" @click="category = item">{{ item === '全部' ? '⊞' : item === '老虎機' ? '▦' : item === '捕魚機' ? '≈' : item === '棋牌' ? '♠' : 'MINI' }}<span>{{ item }}</span></button></div><button class="mobile-search" type="button" aria-label="搜尋遊戲" @click="focusSearch">⌕</button></nav>

    <div v-if="modal" class="modal-backdrop" role="presentation" @mousedown.self="modal = null"><section :class="['modal', `${modal}-modal`]" role="dialog" aria-modal="true" aria-label="彈出視窗"><button class="modal-close" type="button" aria-label="關閉" @click="modal = null">×</button>
      <div v-if="modal === 'game'" class="game-info-modal"><div class="modal-cover" :style="{ backgroundImage: `url(${selectedGame.cover})` }" /><div class="modal-content"><span class="eyebrow">{{ selectedGame.category }}</span><h2>{{ selectedGame.name }}</h2><p>{{ selectedGame.description }} 本遊戲採用明確的規則提示與獎勵呈現，適合快速進入遊戲。</p><div class="modal-metrics"><div class="metric"><strong>{{ selectedGame.volatility }}</strong><span>波動度</span></div><div class="metric"><strong :class="['rtp-pulse', selectedGame.rtpTrend]"><span>{{ selectedGame.rtp }}</span><b aria-hidden="true">{{ selectedGame.rtpTrend === 'down' ? '↘' : '↗' }}</b></strong><span>RTP</span></div><div class="metric"><strong>{{ selectedGame.multiplier }}</strong><span>最高倍率</span></div></div><button class="primary-button" type="button" @click="launchGame(selectedGame)">{{ selectedGame.statusTone === 'upcoming' ? '敬請期待' : '立即遊玩' }} <b>{{ selectedGame.statusTone === 'upcoming' ? '✦' : '→' }}</b></button></div></div>
      <div v-if="modal === 'coming-soon'" class="standard-modal coming-soon-modal"><span class="coming-soon-mark">✦</span><span class="eyebrow">COMING SOON</span><h2>{{ selectedGame.name }} 即將上架</h2><p>我們正在為這款遊戲進行最後準備，敬請期待。</p><button class="primary-button" type="button" @click="modal = null">我知道了</button></div>
      <div v-if="modal === 'avatar'" class="standard-modal avatar-modal"><span class="eyebrow">PROFILE AVATAR</span><h2>更換頭像</h2><p>請選擇一個預設頭像。</p><div class="avatar-picker"><button v-for="avatar in avatarOptions" :key="avatar.id" :class="['avatar-choice', { selected: avatarDraftId === avatar.id }]" type="button" :aria-label="`選擇${avatar.label}頭像`" @click="avatarDraftId = avatar.id"><span :class="['avatar', avatar.tone]">{{ avatar.glyph }}</span><small>{{ avatar.label }}</small><b v-if="avatarDraftId === avatar.id">✓</b></button></div><button class="primary-button" type="button" @click="saveAvatar">確認使用</button></div>
      <div v-if="modal === 'history'" class="standard-modal history-modal"><span class="eyebrow">BETTING HISTORY</span><h2>遊戲紀錄</h2><p class="history-notice">僅可查詢最近三個月內的遊戲紀錄。</p><div class="history-tools"><label>起始日期<input v-model="recordStart" type="date" min="2026-04-18" max="2026-07-18" /></label><label>結束日期<input v-model="recordEnd" type="date" min="2026-04-18" max="2026-07-18" /></label><div class="history-filter"><button class="filter-trigger" type="button" :aria-expanded="recordGameMenuOpen" @click="recordGameMenuOpen = !recordGameMenuOpen"><span>遊戲名稱</span><strong>{{ recordGameSummary }}</strong><b>⌄</b></button><div v-if="recordGameMenuOpen" class="filter-menu"><label v-for="name in recordGameOptions" :key="name"><input v-model="selectedRecordGames" type="checkbox" :value="name" /><span>{{ name }}</span></label></div></div><label>遊戲類型<select v-model="recordCategory"><option value="">全部類型</option><option v-for="item in categories.slice(1)" :key="item" :value="item">{{ item }}</option></select></label></div><div v-if="filteredRecords.length" class="record-list"><div v-for="record in filteredRecords" :key="`${record.name}-${record.date}`" class="record"><span class="record-icon">{{ record.name.slice(0, 1) }}</span><div><strong>{{ record.name }}</strong><small>{{ record.category }} · {{ record.date }}</small></div><b :class="record.amount.startsWith('+') ? 'positive' : 'negative'">{{ record.amount }} NT</b></div></div><div v-else class="history-empty">目前篩選條件下沒有遊戲紀錄。</div><div class="pagination"><button type="button">‹</button><span class="current">1</span><span>2</span><span>3</span><span>…</span><span>12</span><button type="button">›</button></div></div>
      <div v-if="modal === 'support'" class="standard-modal support-modal"><span class="eyebrow">SUPPORT CENTER</span><h2>客服中心</h2><div class="support-tabs"><button type="button" :class="{ active: supportTab === 'submit' }" @click="supportTab = 'submit'">提交問題</button><button type="button" :class="{ active: supportTab === 'history' }" @click="supportTab = 'history'">提問紀錄 <i /></button></div><form v-if="supportTab === 'submit'" @submit.prevent="submitSupport"><label>問題類型<select required><option value="" disabled selected>請選擇問題類型</option><option>帳務問題</option><option>遊戲異常</option><option>帳號與安全</option><option>遊戲相關</option><option>意見與檢舉</option></select></label><label>問題描述<textarea maxlength="500" placeholder="請詳細描述您的問題或建議，客服會盡快為您解答。" /></label><div class="upload-box"><span>＋</span><p>上傳附件<small>JPG、JPEG、PNG，15MB 以內</small></p></div><button class="primary-button" type="submit">提交問題</button></form><div v-else class="support-history"><article><small>2026/07/18</small><strong>帳務問題</strong><p>尚有餘額但無法進入遊戲，已嘗試多次重新登入。</p><b>客服已回覆</b></article><article><small>2026/07/12</small><strong>遊戲相關</strong><p>已確認遊戲畫面顯示正常，謝謝您的回報。</p><b class="done">已完成</b></article></div></div>
    </section></div>

    <div v-if="toast" class="toast" role="status">{{ toast }}</div>
  </main>
</template>
