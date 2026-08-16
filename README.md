# Photo2
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<title>フォトムービー作成</title>
<script src="https://cdn.jsdelivr.net/npm/heic2any@0.0.4/dist/heic2any.min.js"></script>
<style>
  :root{
    --bg:#14161b; --surface:#1e222a; --surface2:#272c36;
    --text:#f2f0ea; --muted:#9aa0ad; --accent:#ecb45e; --accent-ink:#2a1f0d;
    --radius:14px;
  }
  *{box-sizing:border-box; -webkit-tap-highlight-color:transparent; touch-action:manipulation}
  html{-webkit-text-size-adjust:100%}
  body{
    margin:0; background:var(--bg); color:var(--text);
    font-family:-apple-system,"Hiragino Sans","Yu Gothic",sans-serif;
    padding:20px 16px calc(40px + env(safe-area-inset-bottom));
    max-width:560px; margin-inline:auto;
  }
  h1{font-size:22px; font-weight:700; margin:4px 0 2px; letter-spacing:.02em}
  .sub{color:var(--muted); font-size:13px; margin:0 0 20px}
  .card{background:var(--surface); border-radius:var(--radius); padding:16px; margin-bottom:14px}
  .step{font-size:12px; color:var(--accent); font-weight:700; letter-spacing:.08em; margin-bottom:8px}
  button, .btn{
    display:block; width:100%; border:none; border-radius:12px;
    padding:14px; font-size:16px; font-weight:700; cursor:pointer;
    font-family:inherit;
  }
  .btn-accent{background:var(--accent); color:var(--accent-ink)}
  .btn-accent:disabled{opacity:.35}
  .btn-ghost{background:var(--surface2); color:var(--text); margin-top:8px}
  input[type=file]{display:none}
  #photoList{display:flex; flex-direction:column; gap:10px; margin-top:12px}
  .row{display:flex; gap:8px; background:var(--surface2); border-radius:10px; padding:8px; align-items:center}
  .pick{width:20px; height:20px; accent-color:var(--accent); flex-shrink:0}
  .thumbWrap{position:relative; flex-shrink:0}
  .row img{width:96px; height:54px; object-fit:contain; border-radius:6px; display:block; background:#000}
  .vbadge{
    position:absolute; right:3px; bottom:3px; background:rgba(0,0,0,.7); color:#fff;
    font-size:10px; border-radius:4px; padding:1px 4px;
  }
  .ng{width:96px; height:54px; border-radius:6px; background:#3a2a2a; color:#e08787;
      display:flex; align-items:center; justify-content:center; font-size:20px; flex-shrink:0}
  .row input[type=text]{
    flex:1; background:var(--bg); border:1px solid #3a4150; border-radius:8px;
    color:var(--text); padding:10px; font-size:15px; min-width:0; font-family:inherit;
  }
  .row input::placeholder{color:#6b7280}
  .info{flex:1; display:flex; flex-direction:column; gap:6px; min-width:0}
  .trim{display:flex; align-items:center; gap:6px; font-size:12px; color:var(--muted); flex-wrap:nowrap}
  .fillSeg{display:inline-flex; background:var(--bg); border:1px solid #3a4150;
    border-radius:8px; overflow:hidden; flex-shrink:0}
  .fillSeg button{width:38px; height:30px; padding:0; border-radius:0; background:transparent;
    display:flex; align-items:center; justify-content:center; border-right:1px solid #3a4150}
  .fillSeg button:last-child{border-right:none}
  .fillSeg svg{opacity:0.5; transition:opacity .15s}
  .fillSeg button.on{background:var(--accent)}
  .fillSeg button.on svg{opacity:1}
  .fillSeg svg{display:block}
  .trim input[type=number]{
    width:64px; background:var(--bg); border:1px solid #3a4150; border-radius:6px;
    color:var(--text); padding:6px 4px; font-size:14px; font-family:inherit; text-align:center;
  }
  .ctrl{display:flex; flex-direction:column; gap:4px; flex-shrink:0}
  .ctrl button{
    width:36px; height:26px; padding:0; font-size:13px; font-weight:400;
    border-radius:6px; background:var(--surface); color:var(--text);
  }
  .ctrl button:disabled{opacity:.25}
  .ctrl .del{color:#e08787}
  .ctrl .ug{color:var(--accent)}
  .settings{display:flex; align-items:center; justify-content:space-between; gap:12px; margin-top:4px}
  .settings label{font-size:15px}
  select{
    background:var(--surface2); color:var(--text); border:1px solid #3a4150;
    border-radius:8px; padding:8px 10px; font-size:15px; font-family:inherit;
  }
  .bgm-btn{
    background:var(--surface2); color:var(--text); border-radius:8px;
    padding:8px 14px; font-size:14px; cursor:pointer;
  }
  #bgmName{font-size:13px; color:var(--accent); margin-top:8px; display:none}
  #groupBtn{display:none}
  #progressWrap{display:none; margin-top:12px}
  #progressBar{height:8px; background:var(--surface2); border-radius:4px; overflow:hidden}
  #progressFill{height:100%; width:0%; background:var(--accent); transition:width .2s}
  #status{font-size:13px; color:var(--muted); margin-top:6px; text-align:center}
  #result{display:none}
  video{width:100%; border-radius:10px; background:#000; margin-bottom:10px}
  .warn{font-size:12px; color:var(--muted); margin-top:10px; line-height:1.6}
  #count{color:var(--muted); font-size:13px; margin-top:10px}
  body.busy #photoList, body.busy #bulkBar, body.busy .card:nth-of-type(1){
    pointer-events:none; opacity:0.5}
  #lightbox{
    position:fixed; inset:0; background:rgba(0,0,0,.92); display:none;
    flex-direction:column; align-items:center; justify-content:center;
    gap:14px; padding:20px; z-index:50;
  }
  #lightbox.show{display:flex}
  #lbContent{max-width:100%; display:flex; justify-content:center}
  #lbContent img, #lbContent video{max-width:100%; max-height:78vh; border-radius:8px}
  #lbClose{width:auto; padding:10px 32px; background:var(--surface2); color:var(--text);
    border-radius:10px; font-size:15px; font-weight:400}
  #projModal{position:fixed; inset:0; background:rgba(0,0,0,.7); display:none;
    align-items:center; justify-content:center; z-index:60; padding:20px}
  #projModal.show{display:flex}
  #projPanel{background:var(--surface); border-radius:var(--radius); padding:16px;
    width:100%; max-width:420px; max-height:80vh; overflow-y:auto}
  .projRow{display:flex; align-items:center; gap:8px; background:var(--surface2);
    border-radius:10px; padding:10px; margin-bottom:8px}
  .projRow .pn{flex:1; min-width:0; font-size:15px}
  .projRow .pn > div:first-child{overflow:hidden; text-overflow:ellipsis; white-space:nowrap}
  .projRow .pd{font-size:11px; color:var(--muted)}
  .projRow button{width:auto; padding:8px 12px; font-size:13px; font-weight:400;
    border-radius:8px; background:var(--surface); color:var(--text)}
  .projRow .pdel{color:#e08787}
  #cropModal{position:fixed; inset:0; background:rgba(0,0,0,.85); display:none;
    align-items:center; justify-content:center; z-index:70; padding:16px}
  #cropModal.show{display:flex}
  #cropPanel{background:var(--surface); border-radius:var(--radius); padding:16px; width:100%; max-width:460px}
  #cropCv{width:100%; border-radius:8px; background:#0b0d11; touch-action:none; display:block}
  #cropHint{font-size:12px; color:var(--muted); margin:10px 0 4px; line-height:1.6; text-align:center}
  #cropInfo{font-size:12px; color:var(--accent); text-align:center; margin-bottom:12px; min-height:16px}
  .cropBtns{display:flex; gap:8px}
  .cropBtns button{flex:1; padding:12px; font-size:15px}
</style>
</head>
<body>

<h1>フォトムービー作成</h1>
<p class="sub">写真や動画を選んでコメントを付けると、1本のムービー(MP4)を作れます</p>

<div class="card">
  <div class="step">プロジェクト</div>
  <div class="settings">
    <label id="projName" style="overflow:hidden; text-overflow:ellipsis; white-space:nowrap">無題</label>
    <button id="projBtn" style="width:auto; padding:8px 14px; font-size:14px; font-weight:400; background:var(--surface2); color:var(--text)">一覧 / 切替</button>
  </div>
</div>

<div class="card">
  <div class="step">STEP 1 — 写真・動画を選ぶ</div>
  <label class="btn btn-accent" for="fileInput">ライブラリから選択</label>
  <input type="file" id="fileInput" accept="image/jpeg,image/png,video/mp4,video/quicktime" multiple>
  <div id="count"></div>
  <div id="photoList"></div>
  <div id="bulkBar" style="display:none">
    <button class="btn-ghost" id="groupBtn">選択した写真を1枚にまとめる</button>
    <button class="btn-ghost" id="bulkFillBtn">選択した写真の余白をそろえる</button>
    <button class="btn-ghost" id="bulkDelBtn" style="color:#e08787">選択した写真を削除</button>
  </div>
  <div class="warn">写真左のチェックを2〜4枚に付けて「まとめる」を押すと1枚のコラージュになります。繰り返せば何枚でも作れます(「解」で元に戻す)。「ライブラリから選択」は何度押しても既存のリストに追加されます。作業内容は自動保存され、次回開いたとき復元できます(動画素材を除く)。<br>「余白」の3つのアイコンは左から<b>ぼかし背景</b>・<b>画面いっぱい</b>・<b>埋めない(黒帯)</b>です。サムネイルは完成動画と同じ見え方で表示されます(タップで拡大)。</div>
</div>

<div class="card">
  <div class="step">STEP 2 — 設定</div>
  <div class="settings">
    <label>写真1枚の表示時間</label>
    <select id="secSelect">
      <option value="2">2秒</option>
      <option value="3" selected>3秒</option>
      <option value="4">4秒</option>
      <option value="5">5秒</option>
    </select>
  </div>
  <div class="settings" style="margin-top:14px">
    <label>切替エフェクト</label>
    <select id="fxSelect">
      <option value="none">なし</option>
      <option value="fade" selected>フェード(しっとり)</option>
      <option value="slide">スライド(テンポよく)</option>
      <option value="circle">サークル(遊び心)</option>
      <option value="zoom">ズーム(ドラマチック)</option>
      <option value="blinds">ブラインド(スタイリッシュ)</option>
    </select>
  </div>
  <div class="settings" style="margin-top:14px">
    <label>写真の動き</label>
    <select id="motionSelect">
      <option value="none" selected>なし(標準)</option>
      <option value="kenburns">ゆっくりズーム</option>
    </select>
  </div>
  <div class="settings" style="margin-top:14px">
    <label>文字デザイン</label>
    <select id="textStyleSelect">
      <option value="cinema" selected>シネマ(標準)</option>
      <option value="tegaki">てがき(あたたかい)</option>
      <option value="elegant">エレガント(上品)</option>
      <option value="fukidashi">ふきだし(楽しい)</option>
    </select>
  </div>
  <canvas id="stylePreview" width="1280" height="720"
          style="width:100%; border-radius:10px; margin-top:10px; background:#000"></canvas>
  <div class="settings" style="margin-top:14px">
    <label>BGM(任意)</label>
    <label class="bgm-btn" for="bgmInput">音楽ファイルを選択</label>
    <input type="file" id="bgmInput" accept="audio/*" multiple>
  </div>
  <div id="bgmName"></div>
  <div class="warn">BGMは「ファイル」Appの音楽ファイル(mp3 / m4a)を複数選べます。選んだ順に再生され、動画が長い場合は先頭から繰り返します。Apple Musicの曲は使えません。動画素材はその動画の長さだけ再生され、音は入りません(BGMが流れます)。<br>BGMは端末の保存容量の都合でプロジェクトに保存できないことがあります。その場合は次回開いたときに自動で解除され、選び直しをお知らせします。</div>
</div>

<div class="card">
  <div class="step">STEP 3 — 動画を作る</div>
  <div id="estimate" style="font-size:13px; color:var(--muted); margin-bottom:10px; text-align:center"></div>
  <button class="btn-accent" id="makeBtn" disabled>動画を作成</button>
  <div id="progressWrap">
    <div id="progressBar"><div id="progressFill"></div></div>
    <div id="status"></div>
  </div>
  <div class="warn">完成する動画の長さと同じ時間がかかります。作成中に他のアプリへ切り替えたり画面を消したりした場合は、自動的に一時停止し、戻ってきたときに続きから再開します。</div>
</div>

<div class="card" id="liveWrap" style="display:none">
  <div class="step">作成中プレビュー</div>
  <canvas id="cv" width="1280" height="720" style="width:100%; border-radius:10px; background:#000"></canvas>
</div>

<div class="card" id="result">
  <div class="step">完成!</div>
  <video id="preview" controls playsinline></video>
  <button class="btn-accent" id="shareBtn">「写真」に保存 / 共有</button>
  <a class="btn btn-ghost" id="dlLink" download="album.mp4" style="text-align:center; text-decoration:none">ファイルとしてダウンロード</a>
</div>

<div id="lightbox">
  <div id="lbContent"></div>
  <button id="lbClose">閉じる</button>
</div>

<div id="projModal">
  <div id="projPanel">
    <div class="step">プロジェクト一覧</div>
    <div id="projList"></div>
    <button class="btn-accent" id="projNew" style="margin-top:10px">新しいプロジェクトを作る</button>
    <button class="btn-ghost" id="projRename">現在のプロジェクト名を変更</button>
    <button class="btn-ghost" id="projClose">閉じる</button>
  </div>
</div>

<div id="cropModal">
  <div id="cropPanel">
    <div class="step">トリミング範囲の指定</div>
    <canvas id="cropCv"></canvas>
    <div id="cropHint">4本の線を指でドラッグして範囲を決めてください</div>
    <div id="cropInfo"></div>
    <div class="cropBtns">
      <button class="btn-ghost" id="cropReset" style="margin-top:0">全体に戻す</button>
      <button class="btn-accent" id="cropDone">完了</button>
    </div>
  </div>
</div>

<script>
const W = 1280, H = 720, FPS = 30, TR = 0.6;  // TR: エフェクトの長さ(秒)
let items = [];   // 画像/動画: {file, url, kind, comment, state, ...} グループ: {kind:'group', children:[...], comment, ...}
let bgmFiles = [];      // BGM(複数曲を順番に再生)
let resultBlob = null;
let genDiag = null;  // 生成中の診断情報(描画回数・展開失敗回数)

const fileInput = document.getElementById('fileInput');
const photoList = document.getElementById('photoList');
const makeBtn   = document.getElementById('makeBtn');
const countEl   = document.getElementById('count');
const groupBtn  = document.getElementById('groupBtn');

// 余白の埋め方 3モードとアイコン(16:9の枠に写真がどう入るかを図示)
// 中央の明るい部分=写真、左右=余白の処理方法
const FILL_MODES = [
  ['blur', 'ぼかし背景', `<svg width="26" height="16" viewBox="0 0 26 16">
    <rect x="0" y="0" width="26" height="16" rx="2" fill="#6b7482"/>
    <rect x="0" y="0" width="26" height="16" rx="2" fill="#9aa3b2" opacity="0.55"/>
    <rect x="4" y="0" width="3" height="16" fill="#5d6674" opacity="0.5"/>
    <rect x="19" y="0" width="3" height="16" fill="#5d6674" opacity="0.5"/>
    <rect x="9" y="1" width="8" height="14" rx="1" fill="#f4f6fa"/>
  </svg>`],
  ['crop', '画面いっぱい', `<svg width="26" height="16" viewBox="0 0 26 16">
    <rect x="0" y="0" width="26" height="16" rx="2" fill="#f4f6fa"/>
  </svg>`],
  ['none', '埋めない(黒帯)', `<svg width="26" height="16" viewBox="0 0 26 16">
    <rect x="0" y="0" width="26" height="16" rx="2" fill="#101216"/>
    <rect x="9" y="1" width="8" height="14" rx="1" fill="#f4f6fa"/>
  </svg>`]
];


// ダブルタップによる意図しない拡大を防ぐ(CSSのtouch-actionが効かない環境の保険)
(() => {
  let lastTouch = 0;
  document.addEventListener('touchend', e => {
    const now = Date.now();
    if (now - lastTouch < 350) e.preventDefault();   // 素早い2度目のタップは無効化
    lastTouch = now;
  }, {passive: false});
  // ピンチによる拡大は残しつつ、ジェスチャーによる拡大は抑止する
  ['gesturestart', 'gesturechange'].forEach(ev =>
    document.addEventListener(ev, e => e.preventDefault()));
})();

fileInput.addEventListener('change', () => {
  const files = [...fileInput.files];
  if (!files.length) return;
  const newItems = files.map(f => ({
    file: f, url: URL.createObjectURL(f),
    kind: f.type.startsWith('video') ? 'video' : 'image',
    comment: '', state: 'loading', retried: false,
    duration: 0, thumbUrl: null, checked: false,
    trimStart: 0, trimEnd: null, fill: 'blur', crop: null
  }));
  items = items.concat(newItems);  // 既存のリスト(コラージュ含む)を保持したまま追加
  renderList();
  updateCount();
  newItems.forEach(it => it.kind === 'video' ? prepareVideoItem(it) : queueImage(it));
  fileInput.value = '';  // 同じ写真をもう一度選べるようにリセット
});

function prepareVideoItem(it){
  const v = document.createElement('video');
  v.muted = true;
  v.playsInline = true;
  v.preload = 'metadata';
  v.onloadedmetadata = () => {
    it.duration = v.duration;
    if (it.trimEnd == null) it.trimEnd = Math.round(v.duration * 10) / 10;
    v.currentTime = Math.min(0.1, v.duration);
  };
  v.onseeked = () => {
    const c = document.createElement('canvas');
    c.width = PV_W; c.height = PV_H;
    drawFrame(c.getContext('2d'), v, it.fill, it, PV_W, PV_H);
    it.thumbUrl = c.toDataURL('image/jpeg', 0.8);
    c.width = 0; c.height = 0;
    it.state = 'ok';
    renderList();
    updateCount();
  };
  v.onerror = () => {
    it.state = 'failed';
    renderList();
    updateCount();
  };
  it.videoEl = v;  // 生成時に再利用(元ファイル参照が切れても動くように)
  v.src = it.url;
}

// 画像は1枚ずつ順番に処理してメモリのピークを抑える
let imageQueue = Promise.resolve();
function queueImage(it){
  imageQueue = imageQueue.then(() => prepareImageItem(it));
}

// 画像の読み込み: HEICなら変換 → 長辺1600pxの縮小版だけを保持し、原寸データは即解放
async function prepareImageItem(it){
  try {
    let img;
    try {
      img = await loadImage(it.url);
    } catch (e){
      if (!window.heic2any) throw e;
      // HEICの可能性 → JPEGへ変換して再試行
      const jpg = await heic2any({blob: it.file, toType: 'image/jpeg', quality: 0.9});
      it.url = URL.createObjectURL(jpg);
      img = await loadImage(it.url);
    }
    // 縮小版(動画生成・拡大表示に十分な解像度)
    const s = Math.min(1, 1600 / Math.max(img.naturalWidth, img.naturalHeight));
    const mc = document.createElement('canvas');
    mc.width = Math.round(img.naturalWidth * s);
    mc.height = Math.round(img.naturalHeight * s);
    mc.getContext('2d').drawImage(img, 0, 0, mc.width, mc.height);
    const midBlob = await new Promise(r => mc.toBlob(r, 'image/jpeg', 0.93));
    mc.width = 0; mc.height = 0;  // canvasメモリを即解放
    if (!midBlob) throw new Error('縮小に失敗しました');
    it.midUrl = URL.createObjectURL(midBlob);
    it.midBlob = midBlob;  // 一時保存用に保持
    // リスト表示用サムネイル(完成動画と同じ構図で表示する)
    it.thumbUrl = await makePreviewThumb(it, img);
    img.src = '';  // 原寸データの解放を促す
    it.state = 'ok';
  } catch (e){
    it.state = 'failed';
  }
  renderList();
  updateCount();
}

// 完成予定の長さを表示(作成前に見通しが立つように)
function updateEstimate(){
  const el = document.getElementById('estimate');
  if (!el) return;
  const list = items.filter(i => i.state === 'ok');
  if (!list.length){ el.textContent = ''; return; }
  const sec = Number(document.getElementById('secSelect').value);
  let total = 0;
  list.forEach(it => {
    if (it.kind === 'video'){
      const d = it.duration || 1;
      let s = Number(it.trimStart) || 0, e = Number(it.trimEnd);
      if (!isFinite(e) || e <= s) e = d;
      total += Math.max(Math.min(e, d) - s, 0.5);
    } else total += sec;
  });
  const m = Math.floor(total / 60), s2 = Math.round(total % 60);
  el.textContent = `${list.length}件 / 完成予定 ` + (m ? `${m}分${s2}秒` : `${s2}秒`) +
                   `(作成にも同じ時間がかかります)`;
}

function updateCount(){
  const loading = items.filter(i => i.state === 'loading').length;
  const ok = items.filter(i => i.state === 'ok').length;
  const failed = items.filter(i => i.state === 'failed').length;
  if (!items.length){
    countEl.textContent = '';
  } else if (loading){
    countEl.textContent = `読み込み中... ${ok + failed}/${items.length}件`;
  } else {
    countEl.textContent = failed
      ? `✓ ${ok}件の読み込み完了(${failed}件は読み込めず除外)`
      : `✓ ${ok}件すべて読み込み完了`;
  }
  makeBtn.disabled = !(loading === 0 && ok > 0);
  updateEstimate();
  if (loading === 0 && ok > 0) renderStylePreview();
}

function updateGroupBar(){
  const sel = items.filter(i => i.checked && i.kind === 'image' && i.state === 'ok');
  const bar = document.getElementById('bulkBar');
  if (!sel.length){ bar.style.display = 'none'; return; }
  bar.style.display = 'block';
  groupBtn.style.display = (sel.length >= 2 && sel.length <= 4) ? 'block' : 'none';
  groupBtn.textContent = `選択した${sel.length}枚を1枚にまとめる`;
  document.getElementById('bulkFillBtn').textContent = `選択した${sel.length}枚の余白をそろえる`;
  document.getElementById('bulkDelBtn').textContent = `選択した${sel.length}枚を削除`;
}

// 選択した写真の余白設定を、先頭の写真に合わせてそろえる
document.getElementById('bulkFillBtn').addEventListener('click', async () => {
  const sel = items.filter(i => i.checked && i.kind === 'image' && i.state === 'ok');
  if (!sel.length) return;
  const mode = sel[0].fill || 'blur';
  for (const it of sel){
    if (it.fill !== mode){
      it.fill = mode;
      if (it.bgCache){ it.bgCache.width = 0; it.bgCache = null; }
      try { it.thumbUrl = await makePreviewThumb(it); } catch (e){}
    }
    it.checked = false;
  }
  renderList();
  renderStylePreview();
  scheduleSave();
});

// 選択した写真をまとめて削除(2段階タップで誤操作を防ぐ)
let bulkDelArmed = false, bulkDelTimer = null;
document.getElementById('bulkDelBtn').addEventListener('click', () => {
  const sel = items.filter(i => i.checked && i.kind === 'image' && i.state === 'ok');
  if (!sel.length) return;
  const btn = document.getElementById('bulkDelBtn');
  if (!bulkDelArmed){
    bulkDelArmed = true;
    btn.textContent = `本当に${sel.length}枚削除しますか?`;
    bulkDelTimer = setTimeout(() => { bulkDelArmed = false; updateGroupBar(); }, 4000);
    return;
  }
  clearTimeout(bulkDelTimer);
  bulkDelArmed = false;
  items = items.filter(i => !sel.includes(i));
  renderList();
  updateCount();
  renderStylePreview();
  scheduleSave();
});

groupBtn.addEventListener('click', () => {
  const sel = items.filter(i => i.checked && i.kind === 'image' && i.state === 'ok');
  if (sel.length < 2) return;
  if (sel.length > 4){ alert('1枚にまとめられるのは最大4枚までです'); return; }
  const idx = items.indexOf(sel[0]);
  sel.forEach(s => s.checked = false);
  const g = {
    kind: 'group', children: sel, comment: '',
    state: 'loading', thumbUrl: null, duration: 0, checked: false
  };
  items = items.filter(i => !sel.includes(i));
  items.splice(Math.min(idx, items.length), 0, g);
  renderList();
  updateCount();
  updateGroupBar();
  buildGroupThumb(g);
});

async function buildGroupThumb(g){
  try {
    g.thumbUrl = await makePreviewThumb(g);
    g.state = 'ok';
  } catch (e){
    g.state = 'failed';
  }
  renderList();
  updateCount();
}

function renderList(){
  photoList.innerHTML = '';
  items.forEach((it, i) => {
    const row = document.createElement('div');
    row.className = 'row';

    // コメント入力(エラー処理から参照するため先に作る)
    const input = document.createElement('input');
    input.type = 'text';
    input.value = it.comment;

    // まとめ用チェックボックス(通常の写真のみ)
    if (it.kind === 'image' && it.state !== 'failed'){
      const cb = document.createElement('input');
      cb.type = 'checkbox';
      cb.className = 'pick';
      cb.checked = it.checked;
      cb.onchange = () => { it.checked = cb.checked; updateGroupBar(); };
      row.append(cb);
    }

    // サムネイル(保存済みのサムネイル画像を使い、元データへの参照に依存しない)
    let thumb;
    if (it.state === 'failed'){
      thumb = document.createElement('div');
      thumb.className = 'ng';
      thumb.textContent = '✕';
    } else if (it.kind === 'video' || it.kind === 'group'){
      thumb = document.createElement('div');
      thumb.className = 'thumbWrap';
      const im = document.createElement('img');
      if (it.thumbUrl) im.src = it.thumbUrl;
      const badge = document.createElement('span');
      badge.className = 'vbadge';
      badge.textContent = it.kind === 'group'
        ? '田 ' + it.children.length + '枚'
        : (it.duration ? '▶ ' + Math.round(it.duration) + '秒' : '▶');
      thumb.append(im, badge);
    } else {
      thumb = document.createElement('img');
      if (it.thumbUrl) thumb.src = it.thumbUrl;
    }
    if (it.state === 'ok'){
      thumb.style.cursor = 'zoom-in';
      thumb.onclick = () => openLightbox(it);
    }

    if (it.state === 'failed'){
      input.disabled = true;
      input.placeholder = '読み込めません' + (it.file ? ' [' + (it.file.type || '形式不明') + ']' : '');
    } else {
      input.placeholder = 'コメント(任意)';
      input.addEventListener('input', () => { it.comment = input.value; renderStylePreview(); scheduleSave(); });
    }

    // 並べ替え・削除・グループ解除
    const ctrl = document.createElement('div');
    ctrl.className = 'ctrl';
    const up = document.createElement('button');
    up.textContent = '↑';
    up.disabled = (i === 0);
    up.onclick = () => { [items[i-1], items[i]] = [items[i], items[i-1]]; renderList(); };
    const down = document.createElement('button');
    down.textContent = '↓';
    down.disabled = (i === items.length - 1);
    down.onclick = () => { [items[i], items[i+1]] = [items[i+1], items[i]]; renderList(); };
    const del = document.createElement('button');
    del.textContent = '✕';
    del.className = 'del';
    del.onclick = () => { items.splice(i, 1); renderList(); updateCount(); updateGroupBar(); };
    ctrl.append(up, down, del);
    if (it.kind === 'group'){
      const ug = document.createElement('button');
      ug.textContent = '解';
      ug.className = 'ug';
      ug.onclick = () => {
        items.splice(i, 1, ...it.children);
        renderList();
        updateCount();
        updateGroupBar();
      };
      ctrl.append(ug);
    }

    const info = document.createElement('div');
    info.className = 'info';
    info.append(input);
    if ((it.kind === 'image' || it.kind === 'video') && it.state === 'ok'){
      const fillRow = document.createElement('div');
      fillRow.className = 'trim';
      const seg = document.createElement('div');
      seg.className = 'fillSeg';
      const segBtns = {};
      FILL_MODES.forEach(([key, name, icon]) => {
        const b = document.createElement('button');
        b.innerHTML = icon;
        b.title = name;
        b.setAttribute('aria-label', name);
        b.onclick = () => {
          if (it.fill === key) return;
          it.fill = key;
          syncSeg();
          refreshPreview(it);      // 一覧のプレビューを完成構図で作り直す
          renderStylePreview();
          scheduleSave();
        };
        segBtns[key] = b;
        seg.append(b);
      });
      const adj = document.createElement('button');
      adj.style.cssText = 'width:100%; padding:8px; font-size:12px; font-weight:400;' +
        'border-radius:6px; background:var(--surface); color:var(--text)';
      function syncSeg(){
        const cur = it.fill || 'blur';
        FILL_MODES.forEach(([key]) => segBtns[key].classList.toggle('on', key === cur));
        adj.textContent = it.crop ? '✓ トリミング済み' : 'トリミング範囲を指定';
        adj.style.color = it.crop ? 'var(--accent)' : 'var(--text)';
      }
      syncSeg();
      adj.onclick = () => openCropEditor(it);
      fillRow.append(document.createTextNode('余白'), seg);
      info.append(fillRow);
      if (it.kind === 'image'){
        const adjRow = document.createElement('div');
        adjRow.className = 'trim';
        adjRow.append(adj);
        info.append(adjRow);
      }
    }
    if (it.kind === 'video' && it.state === 'ok'){
      const trim = document.createElement('div');
      trim.className = 'trim';
      const ts = document.createElement('input');
      ts.type = 'number'; ts.min = 0; ts.step = 0.5; ts.inputMode = 'decimal';
      ts.value = it.trimStart || 0;
      ts.oninput = () => { it.trimStart = Number(ts.value); scheduleSave(); updateEstimate(); };
      const te = document.createElement('input');
      te.type = 'number'; te.min = 0; te.step = 0.5; te.inputMode = 'decimal';
      te.value = it.trimEnd != null ? it.trimEnd : Math.round(it.duration * 10) / 10;
      te.oninput = () => { it.trimEnd = Number(te.value); scheduleSave(); updateEstimate(); };
      trim.append(document.createTextNode('使用区間'), ts,
                  document.createTextNode('〜'), te, document.createTextNode('秒'));
      info.append(trim);
    }
    row.append(thumb, info, ctrl);
    photoList.append(row);
  });
  updateGroupBar();
  scheduleSave();
}

// ---------- BGM ----------
const bgmNameEl = document.getElementById('bgmName');
let bgmMissingName = null;   // 復元時に実体が失われていたBGMの名前

// 選択中の曲を一覧表示する(順番に再生され、足りなければ先頭に戻って繰り返す)
function renderBgmList(){
  bgmNameEl.innerHTML = '';
  if (bgmMissingName){
    bgmNameEl.style.display = 'block';
    bgmNameEl.style.color = '#e0a87a';
    bgmNameEl.textContent = '⚠ 前回のBGM「' + bgmMissingName + '」は保存できないため解除しました。選び直してください';
    return;
  }
  if (!bgmFiles.length){ bgmNameEl.style.display = 'none'; return; }
  bgmNameEl.style.display = 'block';
  bgmNameEl.style.color = 'var(--accent)';
  bgmFiles.forEach((f, i) => {
    const row = document.createElement('div');
    row.style.cssText = 'display:flex; align-items:center; gap:8px; margin-top:4px';
    const nm = document.createElement('span');
    nm.style.cssText = 'flex:1; min-width:0; overflow:hidden; text-overflow:ellipsis; white-space:nowrap';
    nm.textContent = (i + 1) + '. ' + (f.name || 'BGM');
    const up = document.createElement('button');
    up.textContent = '↑';
    up.style.cssText = 'width:auto; padding:4px 8px; font-size:12px; font-weight:400;' +
      'border-radius:6px; background:var(--surface2); color:var(--text)';
    up.disabled = (i === 0);
    up.onclick = () => { [bgmFiles[i-1], bgmFiles[i]] = [bgmFiles[i], bgmFiles[i-1]]; renderBgmList(); scheduleSave(); };
    const del = document.createElement('button');
    del.textContent = '✕';
    del.style.cssText = 'width:auto; padding:4px 8px; font-size:12px; font-weight:400;' +
      'border-radius:6px; background:var(--surface2); color:#e08787';
    del.onclick = () => { bgmFiles.splice(i, 1); renderBgmList(); scheduleSave(); };
    row.append(nm, up, del);
    bgmNameEl.append(row);
  });
}

document.getElementById('bgmInput').addEventListener('change', e => {
  const added = [...e.target.files];
  if (added.length){
    bgmFiles = bgmFiles.concat(added);   // 選ぶたびに追加できる
    bgmMissingName = null;
  }
  document.getElementById('bgmInput').value = '';  // 同じ曲も再選択できるように
  renderBgmList();
  scheduleSave();
});

// ---------- 拡大表示 ----------
const lightbox = document.getElementById('lightbox');
const lbContent = document.getElementById('lbContent');
async function openLightbox(it){
  lbContent.innerHTML = '';
  if (it.kind === 'video'){
    const v = document.createElement('video');
    v.src = it.url;
    v.controls = true;
    v.playsInline = true;
    v.muted = true;
    v.autoplay = true;
    lbContent.append(v);
  } else if (it.kind === 'group'){
    try {
      const imgs = [];
      for (const ch of it.children){
        imgs.push(await loadImage(ch.midUrl || ch.url));
      }
      const c = document.createElement('canvas');
      c.width = W; c.height = H;
      drawCollage(c.getContext('2d'), imgs, W, H);
      imgs.forEach(im => { im.src = ''; });
      const im = document.createElement('img');
      im.src = c.toDataURL('image/jpeg', 0.85);
      lbContent.append(im);
    } catch (e){ /* 表示できない場合は何もしない */ }
  } else {
    const im = document.createElement('img');
    im.src = it.midUrl || it.url;
    lbContent.append(im);
  }
  lightbox.classList.add('show');
}
function closeLightbox(){
  const v = lbContent.querySelector('video');
  if (v){ try { v.pause(); } catch(e){} }
  lightbox.classList.remove('show');
}
document.getElementById('lbClose').addEventListener('click', closeLightbox);
lightbox.addEventListener('click', e => { if (e.target === lightbox) closeLightbox(); });

// ---------- トリミング範囲の指定(4本の境界線をドラッグ) ----------
const cropModal = document.getElementById('cropModal');
const cropCv = document.getElementById('cropCv');
const cropInfoEl = document.getElementById('cropInfo');
let cropTarget = null, cropImg = null;
let cropRect = null;          // {l, t, r, b} 元画像に対する 0〜1 の割合
let activeLine = null;        // 'l' | 't' | 'r' | 'b'
const MIN_SPAN = 0.08;        // 線どうしが重ならない最小幅

function defaultRect(){ return {l: 0, t: 0, r: 1, b: 1}; }

// 画像を canvas 内に収めるための配置(letterbox)を求める
function cropLayout(){
  const cw = cropCv.width, ch = cropCv.height;
  const s = Math.min(cw / cropImg.naturalWidth, ch / cropImg.naturalHeight);
  const w = cropImg.naturalWidth * s, h = cropImg.naturalHeight * s;
  return {x: (cw - w) / 2, y: (ch - h) / 2, w, h};
}

function cropRender(){
  if (!cropImg || !cropRect) return;
  const ctx = cropCv.getContext('2d');
  const cw = cropCv.width, ch = cropCv.height;
  const L = cropLayout();
  ctx.clearRect(0, 0, cw, ch);
  ctx.fillStyle = '#0b0d11';
  ctx.fillRect(0, 0, cw, ch);
  ctx.drawImage(cropImg, L.x, L.y, L.w, L.h);

  // 選択範囲(画面座標)
  const x0 = L.x + L.w * cropRect.l, x1 = L.x + L.w * cropRect.r;
  const y0 = L.y + L.h * cropRect.t, y1 = L.y + L.h * cropRect.b;

  // 範囲の外側を暗くする
  ctx.fillStyle = 'rgba(0,0,0,0.62)';
  ctx.fillRect(0, 0, cw, y0);
  ctx.fillRect(0, y1, cw, ch - y1);
  ctx.fillRect(0, y0, x0, y1 - y0);
  ctx.fillRect(x1, y0, cw - x1, y1 - y0);

  // 三分割ガイド(構図の目安)
  ctx.strokeStyle = 'rgba(255,255,255,0.22)';
  ctx.lineWidth = 1;
  ctx.beginPath();
  for (let k = 1; k < 3; k++){
    ctx.moveTo(x0 + (x1 - x0) * k / 3, y0); ctx.lineTo(x0 + (x1 - x0) * k / 3, y1);
    ctx.moveTo(x0, y0 + (y1 - y0) * k / 3); ctx.lineTo(x1, y0 + (y1 - y0) * k / 3);
  }
  ctx.stroke();

  // 4本の境界線(操作中の線は太く強調)
  const lines = [
    ['l', x0, y0, x0, y1], ['r', x1, y0, x1, y1],
    ['t', x0, y0, x1, y0], ['b', x0, y1, x1, y1]
  ];
  lines.forEach(([key, ax, ay, bx, by]) => {
    const on = (activeLine === key);
    ctx.strokeStyle = on ? '#ecb45e' : '#ffffff';
    ctx.lineWidth = on ? 5 : 3;
    ctx.beginPath();
    ctx.moveTo(ax, ay); ctx.lineTo(bx, by);
    ctx.stroke();
    // 指でつまむ位置の目印(線の中央)
    const mx = (ax + bx) / 2, my = (ay + by) / 2;
    const vertical = (key === 'l' || key === 'r');
    ctx.fillStyle = on ? '#ecb45e' : 'rgba(255,255,255,0.92)';
    ctx.beginPath();
    if (vertical) roundRect(ctx, mx - 5, my - 26, 10, 52, 5);
    else roundRect(ctx, mx - 26, my - 5, 52, 10, 5);
    ctx.fill();
  });

  // 四隅の目印
  ctx.strokeStyle = '#ecb45e';
  ctx.lineWidth = 5;
  const c = 26;
  ctx.beginPath();
  ctx.moveTo(x0, y0 + c); ctx.lineTo(x0, y0); ctx.lineTo(x0 + c, y0);
  ctx.moveTo(x1 - c, y0); ctx.lineTo(x1, y0); ctx.lineTo(x1, y0 + c);
  ctx.moveTo(x0, y1 - c); ctx.lineTo(x0, y1); ctx.lineTo(x0 + c, y1);
  ctx.moveTo(x1 - c, y1); ctx.lineTo(x1, y1); ctx.lineTo(x1, y1 - c);
  ctx.stroke();

  // 指で画像が隠れても現在位置が分かるよう数値でも示す
  const pw = Math.round((cropRect.r - cropRect.l) * cropImg.naturalWidth);
  const ph = Math.round((cropRect.b - cropRect.t) * cropImg.naturalHeight);
  const names = {l: '左', r: '右', t: '上', b: '下'};
  cropInfoEl.textContent = '範囲 ' + pw + ' × ' + ph + ' px' +
    (activeLine ? ' / ' + names[activeLine] + 'の線を操作中' : '');
}

async function openCropEditor(it){
  cropTarget = it;
  try {
    cropImg = await loadImage(it.midUrl || it.url);
  } catch (e){
    alert('この写真を読み込めませんでした');
    return;
  }
  // canvasの内部解像度を画像の縦横比に合わせる(最大900px)
  const maxW = 900, maxH = 640;
  const s = Math.min(maxW / cropImg.naturalWidth, maxH / cropImg.naturalHeight, 1);
  cropCv.width = Math.max(Math.round(cropImg.naturalWidth * s), 200);
  cropCv.height = Math.max(Math.round(cropImg.naturalHeight * s), 150);
  cropRect = it.crop ? {...it.crop} : defaultRect();
  activeLine = null;
  cropRender();
  cropModal.classList.add('show');
}

function closeCropEditor(save){
  if (save && cropTarget && cropRect){
    const full = (cropRect.l < 0.001 && cropRect.t < 0.001 && cropRect.r > 0.999 && cropRect.b > 0.999);
    cropTarget.crop = full ? null : {...cropRect};   // 全体ならトリミングなし扱い
    refreshPreview(cropTarget);
    renderStylePreview();
    scheduleSave();
  }
  if (cropImg){ cropImg.src = ''; cropImg = null; }
  cropTarget = null;
  activeLine = null;
  cropModal.classList.remove('show');
}

// 画面座標 → canvas内部座標
function toCanvasPos(e){
  const r = cropCv.getBoundingClientRect();
  return {x: (e.clientX - r.left) * (cropCv.width / r.width),
          y: (e.clientY - r.top) * (cropCv.height / r.height),
          scale: cropCv.width / r.width};
}

// 指の位置から最も近い境界線を選ぶ(線の周囲に広いタッチ領域を確保)
function pickLine(p){
  const L = cropLayout();
  const x0 = L.x + L.w * cropRect.l, x1 = L.x + L.w * cropRect.r;
  const y0 = L.y + L.h * cropRect.t, y1 = L.y + L.h * cropRect.b;
  const tol = 34 * p.scale;   // 指で押さえやすい太さ(実寸で約34px)
  const cands = [
    ['l', Math.abs(p.x - x0), p.y >= y0 - tol && p.y <= y1 + tol],
    ['r', Math.abs(p.x - x1), p.y >= y0 - tol && p.y <= y1 + tol],
    ['t', Math.abs(p.y - y0), p.x >= x0 - tol && p.x <= x1 + tol],
    ['b', Math.abs(p.y - y1), p.x >= x0 - tol && p.x <= x1 + tol]
  ].filter(c => c[2] && c[1] <= tol);
  if (!cands.length) return null;
  cands.sort((a, b) => a[1] - b[1]);   // 線が近接していても最も近い方を選ぶ
  return cands[0][0];
}

function moveLine(key, p){
  const L = cropLayout();
  const fx = Math.min(Math.max((p.x - L.x) / L.w, 0), 1);
  const fy = Math.min(Math.max((p.y - L.y) / L.h, 0), 1);
  // 反対側の線を越えないよう常に有効な長方形を保つ
  if (key === 'l') cropRect.l = Math.min(fx, cropRect.r - MIN_SPAN);
  else if (key === 'r') cropRect.r = Math.max(fx, cropRect.l + MIN_SPAN);
  else if (key === 't') cropRect.t = Math.min(fy, cropRect.b - MIN_SPAN);
  else if (key === 'b') cropRect.b = Math.max(fy, cropRect.t + MIN_SPAN);
  cropRect.l = Math.max(cropRect.l, 0); cropRect.t = Math.max(cropRect.t, 0);
  cropRect.r = Math.min(cropRect.r, 1); cropRect.b = Math.min(cropRect.b, 1);
}

cropCv.addEventListener('pointerdown', e => {
  if (!cropRect) return;
  e.preventDefault();
  const p = toCanvasPos(e);
  const key = pickLine(p);
  if (!key) return;                    // 線以外は反応しない(誤操作で画像は動かない)
  activeLine = key;
  cropCv.setPointerCapture(e.pointerId);
  moveLine(key, p);
  cropRender();
});
cropCv.addEventListener('pointermove', e => {
  if (!activeLine) return;
  e.preventDefault();
  moveLine(activeLine, toCanvasPos(e));
  cropRender();
});
['pointerup', 'pointercancel'].forEach(ev => cropCv.addEventListener(ev, () => {
  activeLine = null;
  cropRender();
}));

document.getElementById('cropReset').addEventListener('click', () => {
  cropRect = defaultRect();
  cropRender();
});
document.getElementById('cropDone').addEventListener('click', () => closeCropEditor(true));
cropModal.addEventListener('click', e => { if (e.target === cropModal) closeCropEditor(true); });

// ---------- 描画 ----------
function loadImage(url){
  return new Promise((res, rej) => {
    const im = new Image();
    im.onload = () => res(im);
    im.onerror = () => rej(new Error('画像を読み込めませんでした'));
    im.src = url;
  });
}

// Blobを安定した形式(dataURL)へ変換する
// (保存領域から復元したBlobは一時参照(blob URL)経由だとiOSで読めないことがある)
function blobToDataURL(blob){
  return new Promise((res, rej) => {
    const fr = new FileReader();
    fr.onload = () => res(fr.result);
    fr.onerror = () => rej(new Error('画像データを読み出せませんでした'));
    fr.readAsDataURL(blob);
  });
}

// 動画を保持する見えない置き場所
// (画面に配置されていない動画はiOSが再生を止めてしまうため、極小サイズで配置する)
const videoHolder = document.createElement('div');
videoHolder.style.cssText = 'position:fixed; left:0; top:0; width:1px; height:1px;' +
  'opacity:0.01; pointer-events:none; overflow:hidden; z-index:-1';
document.body.appendChild(videoHolder);

function loadVideo(url){
  return new Promise((res, rej) => {
    const v = document.createElement('video');
    v.muted = true;
    v.defaultMuted = true;
    v.controls = false;
    v.autoplay = false;
    // 全画面表示にならないようにする(iOSは指定がないと再生時に全画面へ切り替える)
    v.playsInline = true;
    v.setAttribute('playsinline', '');
    v.setAttribute('webkit-playsinline', 'true');
    v.setAttribute('x-webkit-airplay', 'deny');
    v.setAttribute('disablepictureinpicture', '');
    v.disablePictureInPicture = true;
    v.setAttribute('muted', '');
    v.preload = 'auto';
    v.style.cssText = 'width:1px; height:1px';
    v.onloadeddata = () => res(v);
    v.onerror = () => rej(new Error('動画を読み込めませんでした'));
    // 保険: 全画面に切り替わってしまった場合はすぐ戻す
    v.addEventListener('webkitbeginfullscreen', () => {
      try { v.webkitExitFullscreen(); } catch (e){}
    });
    v.src = url;
    videoHolder.appendChild(v);
    v.load();
  });
}

function mediaEl(it){ return it.kind === 'video' ? it.videoEl : it.img; }

// トリミング範囲を安全な値に補正して [開始, 終了] を返す
function videoRange(it){
  const d = it.duration || 1;
  let s = Number(it.trimStart);
  if (!isFinite(s) || s < 0) s = 0;
  s = Math.min(s, Math.max(d - 0.5, 0));
  let e = Number(it.trimEnd);
  if (!isFinite(e) || e <= s) e = d;
  e = Math.min(e, d);
  if (e - s < 0.5) e = Math.min(s + 0.5, d);
  return [s, e];
}
function mediaW(el){ return el.videoWidth || el.naturalWidth || el.width; }
function mediaH(el){ return el.videoHeight || el.naturalHeight || el.height; }

// トリミング範囲(0〜1の割合)から元画像の切り出し矩形を求める
function sourceRect(el, crop){
  const mw = mediaW(el), mh = mediaH(el);
  if (!crop) return {sx: 0, sy: 0, sw: mw, sh: mh};
  const l = Math.min(Math.max(crop.l, 0), 1), t = Math.min(Math.max(crop.t, 0), 1);
  const r = Math.min(Math.max(crop.r, 0), 1), b = Math.min(Math.max(crop.b, 0), 1);
  return {sx: mw * l, sy: mh * t,
          sw: Math.max(mw * (r - l), 1), sh: Math.max(mh * (b - t), 1)};
}

// 指定領域いっぱいに切り抜いて表示(src: 元画像の切り出し矩形、省略時は全体)
function drawCover(ctx, img, x, y, w, h, src){
  const s0 = src || sourceRect(img, null);
  const s = Math.max(w / s0.sw, h / s0.sh);
  const sw = w / s, sh = h / s;
  ctx.drawImage(img, s0.sx + (s0.sw - sw) / 2, s0.sy + (s0.sh - sh) / 2, sw, sh, x, y, w, h);
}

// 2〜4枚のコラージュを描く(imgs: 読み込み済み画像要素の配列)
function drawCollage(ctx, imgs, w, h){
  ctx.fillStyle = '#000';
  ctx.fillRect(0, 0, w, h);
  const g = Math.max(Math.round(w * 0.006), 2);  // 写真間の隙間
  const n = imgs.length;
  let cells;
  if (n === 2){
    const cw = (w - g) / 2;
    cells = [[0, 0, cw, h], [cw + g, 0, cw, h]];
  } else if (n === 3){
    const cw = (w - g) / 2, ch = (h - g) / 2;
    cells = [[0, 0, cw, h], [cw + g, 0, cw, ch], [cw + g, ch + g, cw, ch]];
  } else {
    const cw = (w - g) / 2, ch = (h - g) / 2;
    cells = [[0, 0, cw, ch], [cw + g, 0, cw, ch], [0, ch + g, cw, ch], [cw + g, ch + g, cw, ch]];
  }
  imgs.forEach((im, i) => {
    if (i < cells.length) drawCover(ctx, im, ...cells[i]);
  });
}

// 折り返し(日本語対応: 1文字単位、最大3行)
function wrapLines(ctx, comment, font, maxW){
  ctx.font = font;
  const lines = [];
  let line = '';
  for (const ch of comment){
    if (ctx.measureText(line + ch).width <= maxW) line += ch;
    else { lines.push(line); line = ch; if (lines.length === 3) break; }
  }
  if (line && lines.length < 3) lines.push(line);
  return lines;
}

function roundRect(ctx, x, y, w, h, r){
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.arcTo(x + w, y, x + w, y + h, r);
  ctx.arcTo(x + w, y + h, x, y + h, r);
  ctx.arcTo(x, y + h, x, y, r);
  ctx.arcTo(x, y, x + w, y, r);
  ctx.closePath();
}

// コメントを描く(文字デザイン4種)
function drawComment(ctx, comment){
  comment = (comment || '').trim();
  if (!comment) return;
  const style = document.getElementById('textStyleSelect').value;

  if (style === 'cinema'){
    // 半透明の黒帯 + 白ゴシック
    const font = '28px -apple-system, "Hiragino Sans", sans-serif';
    const lines = wrapLines(ctx, comment, font, W - 120);
    const lineH = 40, bandH = lineH * lines.length + 24;
    ctx.fillStyle = 'rgba(0,0,0,0.6)';
    ctx.fillRect(0, H - bandH, W, bandH);
    ctx.fillStyle = '#fff';
    ctx.textAlign = 'center';
    lines.forEach((ln, i) => ctx.fillText(ln, W / 2, H - bandH + 12 + lineH * (i + 0.75)));
    ctx.textAlign = 'left';

  } else if (style === 'tegaki'){
    // 白い角丸ラベル + 丸ゴシックの黒文字
    const font = '30px "Hiragino Maru Gothic ProN", "Hiragino Sans", sans-serif';
    const lines = wrapLines(ctx, comment, font, W - 260);
    const lineH = 42, pad = 22;
    const tw = Math.max(...lines.map(l => ctx.measureText(l).width));
    const bw = tw + pad * 2, bh = lineH * lines.length + pad * 1.4;
    const bx = (W - bw) / 2, by = H - bh - 46;
    roundRect(ctx, bx + 4, by + 5, bw, bh, 16);
    ctx.fillStyle = 'rgba(0,0,0,0.25)';
    ctx.fill();
    roundRect(ctx, bx, by, bw, bh, 16);
    ctx.fillStyle = 'rgba(255,253,245,0.94)';
    ctx.fill();
    ctx.font = font;
    ctx.fillStyle = '#3c3228';
    ctx.textAlign = 'center';
    lines.forEach((ln, i) => ctx.fillText(ln, W / 2, by + pad * 0.5 + lineH * (i + 0.75)));
    ctx.textAlign = 'left';

  } else if (style === 'elegant'){
    // 明朝体の白文字 + 金色のライン(帯なし・下部にごく淡い陰影で可読性を確保)
    const font = '38px "Hiragino Mincho ProN", "Hiragino Sans", serif';
    const lines = wrapLines(ctx, comment, font, W - 200);
    const lineH = 54;
    const baseY = H - 88 - lineH * (lines.length - 1);
    const grad = ctx.createLinearGradient(0, H - 260, 0, H);
    grad.addColorStop(0, 'rgba(0,0,0,0)');
    grad.addColorStop(1, 'rgba(0,0,0,0.55)');
    ctx.fillStyle = grad;
    ctx.fillRect(0, H - 260, W, 260);
    ctx.save();
    ctx.font = font;
    ctx.shadowColor = 'rgba(0,0,0,0.8)';
    ctx.shadowBlur = 10;
    ctx.shadowOffsetY = 2;
    ctx.fillStyle = '#ffffff';
    ctx.textAlign = 'center';
    lines.forEach((ln, i) => ctx.fillText(ln, W / 2, baseY + lineH * i));
    ctx.restore();
    ctx.font = font;
    const tw = Math.max(...lines.map(l => ctx.measureText(l).width));
    ctx.fillStyle = '#e8c584';
    ctx.fillRect((W - tw * 0.7) / 2, baseY + lineH * (lines.length - 1) + 20, tw * 0.7, 3);

  } else if (style === 'fukidashi'){
    // 白いふきだし + 黒文字
    const font = '30px -apple-system, "Hiragino Sans", sans-serif';
    const lines = wrapLines(ctx, comment, font, W - 280);
    const lineH = 42, pad = 26;
    const tw = Math.max(...lines.map(l => ctx.measureText(l).width));
    const bw = tw + pad * 2, bh = lineH * lines.length + pad * 1.3;
    const bx = (W - bw) / 2, by = H - bh - 78;
    ctx.fillStyle = 'rgba(255,255,255,0.96)';
    roundRect(ctx, bx, by, bw, bh, 26);
    ctx.fill();
    const cx = W / 2 + bw * 0.18;
    ctx.beginPath();
    ctx.moveTo(cx, by + bh - 2);
    ctx.lineTo(cx + 42, by + bh - 2);
    ctx.lineTo(cx + 12, by + bh + 30);
    ctx.closePath();
    ctx.fill();
    ctx.font = font;
    ctx.fillStyle = '#28282d';
    ctx.textAlign = 'center';
    lines.forEach((ln, i) => ctx.fillText(ln, W / 2, by + pad * 0.45 + lineH * (i + 0.75)));
    ctx.textAlign = 'left';
  }
}

// 背景ぼかし用の作業用canvas(使い回してメモリを節約)
const bgFull = document.createElement('canvas');    // 元解像度でぼかす用
const bgStep = document.createElement('canvas');    // 段階縮小の途中経過用

// canvasのぼかし(filter)が使えるかを一度だけ判定する
const CAN_FILTER = (() => {
  try {
    const c = document.createElement('canvas').getContext('2d');
    c.filter = 'blur(2px)';
    return c.filter !== 'none' && c.filter !== '';
  } catch (e){ return false; }
})();

// 写真自身を拡大・ぼかして背景に敷く(左右の黒帯を自然に埋める)
// 元の解像度のままぼかすことで、粗い拡大によるモザイク状のムラを防ぐ
// 余白の広さ(0〜1)からぼかしの強さを決める
// 余白が広いほど大きくぼかして背景を目立たせず、狭いほどはっきり残して一体感を保つ
function blurStrength(marginRatio, w){
  const m = Math.min(Math.max(marginRatio || 0, 0), 0.6);
  const t = Math.min(m / 0.5, 1);                        // 余白50%以上で最大
  // 余白の広さにほぼ比例させる(狭ければ弱く、広ければ強く)
  // 画面幅の1.0%〜37.5%(1280px幅換算で 13px〜480px)
  const ratio = 0.010 + (0.375 - 0.010) * Math.pow(t, 0.85);
  return Math.max(w * ratio, 3);
}

function drawBlurBackdrop(ctx, el, src, w, h, marginRatio){
  w = Math.round(w || W); h = Math.round(h || H);
  const over = 1.16;                       // 端のにじみを隠すため少し大きめに描く
  const bw = Math.round(w * over), bh = Math.round(h * over);
  bgFull.width = bw; bgFull.height = bh;
  const bc = bgFull.getContext('2d');
  bc.imageSmoothingEnabled = true;
  bc.imageSmoothingQuality = 'high';

  if (CAN_FILTER){
    const radius = blurStrength(marginRatio, w);
    // 仕上げに全体を軽くぼかし、拡大時のカクつきや階調の段差を消す
    const finish = Math.min(Math.max(w / 110, 4), 16);
    // ぼかしが強いときは縮小してから処理する(細部が残らないため見た目は変わらない)
    // ただし縮小しすぎると粗くなるので、十分な作業解像度を確保する
    const base = Math.min(bw, Math.max(Math.round(w * 0.60), 480));
    const k = (radius > finish * 1.5) ? Math.max(bw / base, 1) : 1;
    if (k > 1){
      const rw = Math.max(Math.round(bw / k), 32), rh = Math.max(Math.round(bh / k), 18);
      // 仕上げのぼかしと合わせて狙いの強さになるよう逆算する
      const sigma = Math.sqrt(Math.max(radius * radius - finish * finish, 1)) / k;
      bgStep.width = rw; bgStep.height = rh;
      const sc2 = bgStep.getContext('2d');
      sc2.imageSmoothingEnabled = true;
      sc2.imageSmoothingQuality = 'high';
      sc2.filter = 'blur(' + sigma + 'px)';
      drawCover(sc2, el, 0, 0, rw, rh, src);
      sc2.filter = 'none';
      bc.filter = 'blur(' + finish + 'px)';
      bc.drawImage(bgStep, 0, 0, rw, rh, 0, 0, bw, bh);
      bc.filter = 'none';
      bgStep.width = 0; bgStep.height = 0;
    } else {
      // 元画像をそのままの精細さで描いてから、まとめてぼかす(最も滑らか)
      bc.filter = 'blur(' + radius + 'px)';
      drawCover(bc, el, 0, 0, bw, bh, src);
      bc.filter = 'none';
    }
  } else {
    // ぼかし未対応の環境: 半分ずつ縮小 → 半分ずつ拡大して滑らかにする
    let cw = bw, ch = bh;
    drawCover(bc, el, 0, 0, bw, bh, src);
    const steps = [];
    // ぼかしの強さに応じて縮小の下限を変える(強いほど小さくまで縮める)
    const limitW = Math.max(Math.round(bw / (blurStrength(marginRatio, w) * 1.6)), 12);
    while (cw > limitW && ch > 8){                  // 段階的に縮小
      cw = Math.max(Math.round(cw / 2), limitW);
      ch = Math.max(Math.round(ch / 2), 8);
      steps.push([cw, ch]);
      bgStep.width = cw; bgStep.height = ch;
      const sc = bgStep.getContext('2d');
      sc.imageSmoothingEnabled = true;
      sc.imageSmoothingQuality = 'high';
      sc.drawImage(bgFull, 0, 0, cw, ch);
      bc.clearRect(0, 0, bw, bh);
      bc.drawImage(bgStep, 0, 0, cw, ch);
    }
    for (let i = steps.length - 2; i >= 0; i--){    // 段階的に拡大して戻す
      const [pw, ph] = steps[i];
      bgStep.width = pw; bgStep.height = ph;
      const sc = bgStep.getContext('2d');
      sc.imageSmoothingEnabled = true;
      sc.imageSmoothingQuality = 'high';
      sc.drawImage(bgFull, 0, 0, cw, ch, 0, 0, pw, ph);
      bc.clearRect(0, 0, bw, bh);
      bc.drawImage(bgStep, 0, 0, pw, ph);
      cw = pw; ch = ph;
    }
    bgStep.width = 0; bgStep.height = 0;
  }

  ctx.save();
  ctx.imageSmoothingEnabled = true;
  ctx.imageSmoothingQuality = 'high';
  ctx.drawImage(bgFull, 0, 0, bw, bh, (w - bw) / 2, (h - bh) / 2, bw, bh);
  ctx.restore();
  ctx.fillStyle = 'rgba(0,0,0,0.28)';  // 少し沈めて主役の写真を引き立てる
  ctx.fillRect(0, 0, w, h);
  applyGrain(ctx, w, h);                // 階調の段差(縞・ブロック)を目立たなくする
  bgFull.width = 0; bgFull.height = 0;  // 使い終わったら解放
}

// ごく薄い粒子(ノイズ)のパターンを一度だけ作る
let grainPattern = null;
function getGrainPattern(ctx){
  if (grainPattern) return grainPattern;
  const size = 96;
  const nc = document.createElement('canvas');
  nc.width = size; nc.height = size;
  const nx = nc.getContext('2d');
  const img = nx.createImageData(size, size);
  for (let i = 0; i < img.data.length; i += 4){
    const v = Math.random() < 0.5 ? 0 : 255;
    img.data[i] = img.data[i+1] = img.data[i+2] = v;
    img.data[i+3] = Math.random() * 12;    // ほとんど透明(見た目にはほぼ分からない量)
  }
  nx.putImageData(img, 0, 0);
  grainPattern = ctx.createPattern(nc, 'repeat');
  return grainPattern;
}

// なめらかな背景は圧縮で縞やブロックが出やすいため、微細な粒子で散らす
function applyGrain(ctx, w, h){
  try {
    const p = getGrainPattern(ctx);
    if (!p) return;
    ctx.save();
    ctx.globalAlpha = 0.35;
    ctx.fillStyle = p;
    ctx.fillRect(0, 0, w, h);
    ctx.restore();
  } catch (e){ /* 粒子は演出目的なので失敗しても続行 */ }
}

// fill: 'blur'(既定) = ぼかし背景 / 'crop' = 画面いっぱい / 'none' = 埋めない(黒帯)
// it.crop があれば、その範囲だけを切り出して表示する
function drawMedia(ctx, el, comment, fill, it){
  drawFrame(ctx, el, fill, it, W, H);
  drawComment(ctx, comment);
}

// 完成動画と同じ構図を任意サイズに描く(一覧のプレビューでも使う)
function drawFrame(ctx, el, fill, it, cw, ch){
  const mode = fill || 'blur';
  const s0 = sourceRect(el, it && it.crop);
  const fits = Math.abs(s0.sw / s0.sh - cw / ch) < 0.01;  // 比率が合えば帯は出ない
  ctx.fillStyle = '#000';
  ctx.fillRect(0, 0, cw, ch);
  if (mode === 'crop'){
    drawCover(ctx, el, 0, 0, cw, ch, s0);
  } else {
    if (mode === 'blur' && !fits){
      // 写真が実際に表示される大きさから、余白がどれだけ広いかを求める
      const sc = Math.min(cw / s0.sw, ch / s0.sh);
      const marginRatio = Math.max((cw - s0.sw * sc) / cw, (ch - s0.sh * sc) / ch);
      if (it && it.kind === 'video'){
        // 動画は毎フレームぼかすと重いので、背景を一度だけ作って使い回す
        if (!it.bgCache || it.bgCache.width !== cw || it.bgCache.height !== ch){
          it.bgCache = document.createElement('canvas');
          it.bgCache.width = cw; it.bgCache.height = ch;
          drawBlurBackdrop(it.bgCache.getContext('2d'), el, s0, cw, ch, marginRatio);
        }
        ctx.drawImage(it.bgCache, 0, 0);
      } else {
        drawBlurBackdrop(ctx, el, s0, cw, ch, marginRatio);
      }
    }
    const scale = Math.min(cw / s0.sw, ch / s0.sh);
    const w = s0.sw * scale, h = s0.sh * scale;
    ctx.drawImage(el, s0.sx, s0.sy, s0.sw, s0.sh, (cw - w) / 2, (ch - h) / 2, w, h);
  }
}

// 一覧用に「完成時と同じ構図」のサムネイルを作る
const PV_W = 320, PV_H = 180;
async function makePreviewThumb(it, el){
  const c = document.createElement('canvas');
  c.width = PV_W; c.height = PV_H;
  const cx = c.getContext('2d');
  if (it.kind === 'group'){
    const imgs = [];
    for (const ch of it.children) imgs.push(await loadImage(ch.midUrl || ch.url));
    drawCollage(cx, imgs, PV_W, PV_H);
    imgs.forEach(im => { im.src = ''; });
  } else {
    const own = !el;
    const im = el || await loadImage(it.midUrl || it.url);
    drawFrame(cx, im, it.fill, it, PV_W, PV_H);
    if (own) im.src = '';
  }
  const url = c.toDataURL('image/jpeg', 0.8);
  c.width = 0; c.height = 0;
  return url;
}

// 設定変更後にプレビューを作り直す
async function refreshPreview(it){
  // 余白の設定が変わったら、動画用にキャッシュした背景は作り直す必要がある
  if (it.bgCache){ it.bgCache.width = 0; it.bgCache = null; }
  try { it.thumbUrl = await makePreviewThumb(it); } catch (e){ /* 失敗時は既存のまま */ }
  renderList();
}

// 文字デザインのライブプレビュー
const previewCv = document.getElementById('stylePreview');
const previewBg = {src: null, img: null};
async function renderStylePreview(){
  const ctx = previewCv.getContext('2d');
  // 背景: 読み込み済みの最初の写真、なければサンプルのグラデーション
  let bgImg = null;
  const first = items.find(i => i.kind === 'image' && i.state === 'ok');
  if (first){
    try {
      const src = first.midUrl || first.url;
      if (previewBg.src !== src){
        previewBg.img = await loadImage(src);
        previewBg.src = src;
      }
      bgImg = previewBg.img;
    } catch (e){ /* 背景なしで続行 */ }
  }
  if (bgImg){
    drawMedia(ctx, bgImg, null, first ? first.fill : 'blur', first);
  } else {
    const g = ctx.createLinearGradient(0, 0, 0, H);
    g.addColorStop(0, '#5689bd');
    g.addColorStop(1, '#d6e4eb');
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, W, H);
  }
  const withComment = items.find(i => i.comment && i.comment.trim());
  drawComment(ctx, withComment ? withComment.comment : 'サンプルの文字です');
}
document.getElementById('textStyleSelect').addEventListener('change', renderStylePreview);
renderStylePreview();  // 初期表示

let motionNow = 0;  // 現在の再生時刻(Ken Burns用)

// 1コマ描画: 展開済みのコマ画像を転写(動画素材は生の映像を描画)
// 「写真の動き」がオンのときは写真だけをゆっくり動かし、コメントは固定で上描きする
function drawSeg(ctx, seg){
  if (seg.it.kind === 'video'){
    if (!seg.it.videoEl){ prepareSeg(seg); return; }   // 読み込み中は前のコマを維持
    drawMedia(ctx, seg.it.videoEl, seg.it.comment, seg.it.fill, seg.it);
    if (genDiag) genDiag.segDraws[seg.idx] = (genDiag.segDraws[seg.idx] || 0) + 1;
    return;
  }
  if (!seg.frameImg){
    ensureFrame(seg);  // 未展開なら自己修復(展開できるまで前のフレームを維持)
    return;
  }
  const motion = document.getElementById('motionSelect').value;
  if (motion === 'kenburns'){
    const p = Math.min(Math.max((motionNow - seg.start) / seg.dur, 0), 1);
    const v = seg.idx % 4;  // コマごとに動きを変えて単調にしない
    let s, dx = 0, dy = 0;
    if (v === 0){ s = 1 + 0.08 * p; }                       // 寄っていく
    else if (v === 1){ s = 1.08 - 0.08 * p; }               // 引いていく
    else if (v === 2){ s = 1.08; dx = (p - 0.5) * (s - 1) * W; }  // 横に流れる
    else { s = 1.08; dy = (p - 0.5) * (s - 1) * H; }        // 縦に流れる
    ctx.save();
    ctx.translate(W / 2 - dx, H / 2 - dy);
    ctx.scale(s, s);
    ctx.translate(-W / 2, -H / 2);
    ctx.drawImage(seg.frameImg, 0, 0);
    ctx.restore();
  } else {
    ctx.drawImage(seg.frameImg, 0, 0);
  }
  drawComment(ctx, seg.it.comment);
  if (genDiag) genDiag.segDraws[seg.idx] = (genDiag.segDraws[seg.idx] || 0) + 1;
}

// 1コマ分の素材を必要になる直前に用意する
// (全素材を先に読み込むとメモリが枚数に比例して増え、端末が落ちるため)
let prepareBusy = false;   // 準備は一度に1件だけ行い、描画の妨げを最小にする
async function prepareSeg(seg){
  if (!seg || seg.prepared || seg.preparing) return;
  if (prepareBusy){ return; }        // 他の準備中は次の機会に回す(描画を優先)
  seg.preparing = true;
  prepareBusy = true;
  try {
    await new Promise(r => requestAnimationFrame(r));   // 描画1回分の順番を譲る
    if (seg.it.kind === 'video'){
      // 動画は再生する直前に読み込む(同時に開く本数を抑える)
      if (!seg.it.videoEl) seg.it.videoEl = await loadVideo(seg.it.url);
      // 頭出しまで済ませておく(再生開始時に映像が止まるのを防ぐ)
      const v = seg.it.videoEl;
      const startAt = videoRange(seg.it)[0];
      if (Math.abs(v.currentTime - startAt) > 0.05){
        await new Promise(r => {
          let done = false;
          const fin = () => { if (!done){ done = true; v.removeEventListener('seeked', fin); r(); } };
          v.addEventListener('seeked', fin);
          setTimeout(fin, 2500);          // 応答がなくても先へ進む
          try { v.currentTime = startAt; } catch (e){ fin(); }
        });
      }
    } else if (!seg.frameImg){
      const c = document.createElement('canvas');
      c.width = W; c.height = H;
      const cx = c.getContext('2d');
      if (seg.it.kind === 'group'){
        const imgs = [];
        for (const ch of seg.it.children) imgs.push(await loadImage(ch.midUrl || ch.url));
        drawCollage(cx, imgs, W, H);
        imgs.forEach(im => { im.src = ''; });
      } else {
        const im = await loadImage(seg.it.midUrl || seg.it.url);
        drawMedia(cx, im, null, seg.it.fill, seg.it);  // コメントは再生時に描く
        im.src = '';
      }
      // 同時に保持するのは数コマだけなので、圧縮せずそのまま持つ
      // (PNG化と再展開は時間がかかり、録画中の映像がカクつく原因になる)
      if (window.createImageBitmap){
        seg.frameImg = await createImageBitmap(c);
        c.width = 0; c.height = 0;   // 元canvasは解放
      } else {
        seg.frameImg = c;            // 未対応環境はcanvasのまま使う
      }
    }
    seg.prepared = true;
  } catch (e){
    seg.prepareError = e;          // 描画時に前のコマを保持したまま次で再試行される
  } finally {
    seg.preparing = false;
    prepareBusy = false;
  }
}

// 使い終わった素材をすべて解放する(メモリを枚数に依存させない)
function releaseSeg(seg){
  if (!seg) return;
  freeFrame(seg);
  if (seg.it.bgCache){ seg.it.bgCache.width = 0; seg.it.bgCache = null; }
  if (seg.it.kind === 'video' && seg.it.videoEl){
    const v = seg.it.videoEl;
    try { v.pause(); } catch (e){}
    try { v.removeAttribute('src'); v.load(); } catch (e){}
    try { if (v.parentNode) v.parentNode.removeChild(v); } catch (e){}
    seg.it.videoEl = null;
  }
  seg.prepared = false;
  seg.started = false;
}

// コマが未用意なら用意を促す(できるまでは前のコマを表示し続ける)
function ensureFrame(seg){
  if (!seg || seg.it.kind === 'video' || seg.frameImg) return Promise.resolve();
  if (genDiag && !seg.preparing) genDiag.decodeFail++;
  return prepareSeg(seg);
}

// コマ画像の即時解放(ImageBitmapはclose()で確実にメモリが戻る)
function freeFrame(seg){
  if (!seg || !seg.frameImg) return;
  if (seg.frameImg.close) seg.frameImg.close();          // ImageBitmap
  else if (seg.frameImg.getContext){ seg.frameImg.width = 0; seg.frameImg.height = 0; }  // canvas
  else seg.frameImg.src = '';
  seg.frameImg = null;
  seg.frameLoading = false;
}

// 切替エフェクト(p: 0→1)
function drawTransition(ctx, prevSeg, seg, p, fx){
  p = p * p * (3 - 2 * p);  // なめらかに(イージング)
  drawSeg(ctx, prevSeg);

  if (fx === 'fade'){
    ctx.globalAlpha = p;
    drawSeg(ctx, seg);
    ctx.globalAlpha = 1;

  } else if (fx === 'slide'){
    ctx.save();
    ctx.translate(W * (1 - p), 0);
    drawSeg(ctx, seg);
    ctx.restore();

  } else if (fx === 'circle'){
    ctx.save();
    ctx.beginPath();
    const maxR = Math.hypot(W, H) / 2;
    ctx.arc(W / 2, H / 2, maxR * p, 0, Math.PI * 2);
    ctx.clip();
    drawSeg(ctx, seg);
    ctx.restore();

  } else if (fx === 'zoom'){
    ctx.save();
    ctx.globalAlpha = p;
    const s = 0.6 + 0.4 * p;
    ctx.translate(W / 2, H / 2);
    ctx.scale(s, s);
    ctx.translate(-W / 2, -H / 2);
    drawSeg(ctx, seg);
    ctx.restore();
    ctx.globalAlpha = 1;

  } else if (fx === 'blinds'){
    const N = 8, stripW = W / N;
    ctx.save();
    ctx.beginPath();
    for (let k = 0; k < N; k++){
      ctx.rect(k * stripW, 0, stripW * p, H);
    }
    ctx.clip();
    drawSeg(ctx, seg);
    ctx.restore();
  }
}

// 録画形式を選ぶ。BGMがある場合は「音声コーデックを含む形式」を優先する
// (映像だけの指定にすると、音声トラックが記録されず無音になる)
function pickMimeType(hasAudio){
  const withAudio = [
    'video/mp4;codecs=avc1,mp4a.40.2',
    'video/mp4;codecs=avc1.42E01E,mp4a.40.2',
    'video/mp4',
    'video/webm;codecs=vp9,opus',
    'video/webm;codecs=vp8,opus',
    'video/webm'
  ];
  const videoOnly = ['video/mp4;codecs=avc1', 'video/mp4', 'video/webm;codecs=vp9', 'video/webm'];
  for (const t of (hasAudio ? withAudio : videoOnly)){
    if (MediaRecorder.isTypeSupported(t)) return t;
  }
  return '';
}

// ---------- 動画生成 ----------
makeBtn.addEventListener('click', async () => {
  // 復元でBGMが失われている場合は、無音のまま作ってしまわないよう確認する
  if (!bgmFiles.length && bgmMissingName){
    if (!confirm('前回のBGM「' + bgmMissingName + '」は保存されていないため、このまま作ると音なしになります。\nこのまま続けますか?')){
      return;
    }
    bgmMissingName = null;
    renderBgmList();
    scheduleSave();
  }
  makeBtn.disabled = true;
  document.body.classList.add('busy');   // 生成中は素材の編集を無効化
  document.getElementById('result').style.display = 'none';
  document.getElementById('progressWrap').style.display = 'block';
  document.getElementById('liveWrap').style.display = 'block';
  const status = document.getElementById('status');
  const fill = document.getElementById('progressFill');
  status.textContent = '素材を読み込み中...';

  let actx = null, bgmSrcs = [], bgmTrack = null;
  // iOS対策: 音声機能(AudioContext)はタップ操作の直後に、待機を挟まずに作る必要がある
  // (先に他の待機処理を入れると「操作直後」と見なされず、無音になる)
  if (bgmFiles.length){
    try {
      actx = new (window.AudioContext || window.webkitAudioContext)();
      actx.resume();
    } catch (e){ actx = null; }
  }
  // 生成中の画面消灯・省電力によるフリーズを防ぐ
  let wakeLock = null;
  try { if (navigator.wakeLock) wakeLock = await navigator.wakeLock.request('screen'); } catch (e){}
  let segsRef = [];
  let frameLoop = null;
  const genCleanup = [];        // 生成終了時に片づける処理
  try {
    const list = items.filter(it => it.state === 'ok');
    if (!list.length) throw new Error('使用できる素材がありません');

    const sec = Number(document.getElementById('secSelect').value);
    const fx = document.getElementById('fxSelect').value;

    // タイムライン: 写真・コラージュ=設定秒数、動画=トリミング後の長さ
    // 動画の長さはメタデータから分かるため、この時点で読み込む必要はない
    const segs = [];
    let start = 0;
    for (const it of list){
      let dur = sec;
      if (it.kind === 'video'){
        const [ts, te] = videoRange(it);
        dur = te - ts;
      }
      segs.push({it, start, dur, started: false, idx: segs.length});
      start += dur;
    }
    const total = start;

    const cv = document.getElementById('cv');
    const ctx = cv.getContext('2d');
    // 先頭の2コマ分だけ先に用意する(残りは再生に合わせて順次用意する)
    status.textContent = '素材を準備中...';
    await prepareSeg(segs[0]);
    await prepareSeg(segs[1]);
    prepareSeg(segs[2]);
    segsRef = segs;
    drawSeg(ctx, segs[0]);

    const stream = cv.captureStream(FPS);

    // BGM(ループ+末尾1.5秒フェードアウト)
    if (bgmFiles.length && actx){
      try {
        status.textContent = 'BGMを読み込み中...';
        const dest = actx.createMediaStreamDestination();
        const gain = actx.createGain();
        gain.connect(dest);

        // 選んだ曲を順番に読み込む(読めない曲は飛ばす)
        const buffers = [];
        for (let bi = 0; bi < bgmFiles.length; bi++){
          status.textContent = `BGMを読み込み中... ${bi + 1}/${bgmFiles.length}`;
          try {
            const arr = await bgmFiles[bi].arrayBuffer();
            // Safariの新旧両方の書き方に対応し、15秒で応答がなければ打ち切る
            const decode = new Promise((res, rej) => {
              const r = actx.decodeAudioData(arr, res, rej);
              if (r && r.then) r.then(res).catch(rej);
            });
            const timeout = new Promise((_, rej) =>
              setTimeout(() => rej(new Error('読み込みに時間がかかりすぎました')), 15000));
            buffers.push(await Promise.race([decode, timeout]));
          } catch (e){ /* この曲は飛ばす */ }
        }
        if (!buffers.length) throw new Error('BGMを読み込めませんでした');

        // 曲を順番につなげ、動画の長さに足りなければ先頭から繰り返す
        const t0a = actx.currentTime + 0.06;
        let at = 0, guard = 0;
        while (at < total && guard < 500){
          for (const buf of buffers){
            if (at >= total) break;
            const s = actx.createBufferSource();
            s.buffer = buf;
            s.connect(gain);
            const startIn = t0a + at;
            const remain = total - at;
            s.start(startIn, 0, Math.min(buf.duration, remain + 0.5));
            bgmSrcs.push(s);
            at += buf.duration;
            guard++;
          }
        }

        // 末尾1.5秒でフェードアウト
        gain.gain.setValueAtTime(1, t0a);
        gain.gain.setValueAtTime(1, t0a + Math.max(total - 1.5, 0));
        gain.gain.linearRampToValueAtTime(0.001, t0a + total);

        bgmTrack = dest.stream.getAudioTracks()[0];
        if (actx.state === 'suspended') await actx.resume();   // 停止状態なら再開して無音を防ぐ
        if (!bgmTrack || bgmTrack.readyState !== 'live'){
          bgmTrack = null;
          status.textContent = 'BGMを設定できないため音なしで作成します';
        } else if (buffers.length < bgmFiles.length){
          status.textContent = `${bgmFiles.length - buffers.length}曲は読み込めませんでした(残りで作成します)`;
        }
      } catch (e){
        status.textContent = 'BGMを読み込めないため音なしで作成します';
        bgmSrcs = [];
      }
    }

    // 映像と音声は新しいMediaStreamに合成してから録画する
    // (canvasのストリームに音声を直接addTrackすると、iOSで映像だけ途中停止する事例への対策)
    const recStream = bgmTrack
      ? new MediaStream([...stream.getVideoTracks(), bgmTrack])
      : stream;

    genDiag = {decodeFail: 0, segDraws: {}};

    const mime = pickMimeType(!!bgmTrack);
    let rec;
    try {
      rec = new MediaRecorder(recStream, mime ? {mimeType: mime, videoBitsPerSecond: 14_000_000} : undefined);
    } catch (e) {
      rec = new MediaRecorder(recStream);
    }
    const chunks = [];
    rec.ondataavailable = e => { if (e.data && e.data.size) chunks.push(e.data); };
    rec.onerror = e => { status.textContent = '録画エラー: ' + (e.error ? e.error.message : '不明'); };

    const done = new Promise(res => {
      rec.onstop = res;
      rec.addEventListener('dataavailable', () => { if (rec.state === 'inactive') res(); });
    });
    rec.start(1000);  // 1秒ごとにデータを受け取り、長尺でも巨大な単一バッファに依存しない

    let t0 = performance.now();
    let segIdx = 0;
    // 他のアプリに切り替えた場合、映像の記録は止まってしまう
    // そこで録画・音楽・時間の進みをまとめて一時停止し、戻ったときに続きから再開する
    let pausedAt = 0;
    const onVisibility = () => {
      if (document.hidden){
        if (pausedAt) return;
        pausedAt = performance.now();
        try { if (rec.state === 'recording') rec.pause(); } catch (e){}
        try { if (actx && actx.state === 'running') actx.suspend(); } catch (e){}
        segs.forEach(s => { if (s.it.videoEl){ try { s.it.videoEl.pause(); } catch (e){} } });
      } else if (pausedAt){
        t0 += performance.now() - pausedAt;   // 止まっていた分だけ時間をずらす
        pausedAt = 0;
        try { if (rec.state === 'paused') rec.resume(); } catch (e){}
        try { if (actx && actx.state === 'suspended') actx.resume(); } catch (e){}
        const cur = segs[segIdx];
        if (cur && cur.it.kind === 'video' && cur.it.videoEl && cur.started){
          try { cur.it.videoEl.play(); } catch (e){}
        }
        requestAnimationFrame(frameLoop);     // 停止していた描画を再開する
      }
    };
    document.addEventListener('visibilitychange', onVisibility);
    genCleanup.push(() => document.removeEventListener('visibilitychange', onVisibility));
    await new Promise(resolve => {
      frameLoop = function frame(now){
        if (pausedAt) return;                 // 画面を離れている間は進めない
        const t = (now - t0) / 1000;
        motionNow = t;
        if (t >= total){ resolve(); return; }
        while (segIdx < segs.length - 1 && t >= segs[segIdx + 1].start){
          segIdx++;
          releaseSeg(segs[segIdx - 2]);    // 通り過ぎた素材はまとめて解放
          ensureFrame(segs[segIdx]);       // 現在のコマ(未用意なら)
          prepareSeg(segs[segIdx + 1]);    // 次の素材を用意
          prepareSeg(segs[segIdx + 2]);    // さらに先読みして余裕を持たせる
          prepareSeg(segs[segIdx + 3]);
        }
        const seg = segs[segIdx];

        // 動画素材はセグメント開始時に再生を始める
        if (seg.it.kind === 'video' && seg.it.videoEl){
          const v = seg.it.videoEl;
          if (!seg.started){
            seg.started = true;
            const startAt = videoRange(seg.it)[0];
            if (Math.abs(v.currentTime - startAt) > 0.3){
              try { v.currentTime = startAt; } catch (e){}
            }
            try { const p = v.play(); if (p && p.catch) p.catch(() => {}); } catch (e){}
          } else if (v.paused && !document.hidden){
            // 何らかの理由で止まっていたら再開する(映像が固まるのを防ぐ)
            try { const p = v.play(); if (p && p.catch) p.catch(() => {}); } catch (e){}
          }
        }

        // 準備が他の処理と重なって見送られた場合に備え、毎フレーム軽く促す
        // (用意済み・準備中なら即座に戻るため負荷はほとんどない)
        prepareSeg(segs[segIdx + 1]);
        prepareSeg(segs[segIdx + 2]);

        const local = t - seg.start;
        if (fx !== 'none' && segIdx > 0 && local < TR){
          drawTransition(ctx, segs[segIdx - 1], seg, local / TR, fx);
        } else {
          drawSeg(ctx, seg);
        }

        fill.style.width = (t / total * 100).toFixed(1) + '%';
        status.textContent = `作成中... 残り${Math.ceil(total - t)}秒`;
        requestAnimationFrame(frameLoop);
      };
      requestAnimationFrame(frameLoop);
    });

    rec.stop();
    await Promise.race([done, new Promise(r => setTimeout(r, 3000))]);

    const totalBytes = chunks.reduce((s, c) => s + c.size, 0);
    if (!totalBytes){
      throw new Error('録画データが空でした(形式: ' + (rec.mimeType || mime || '既定') + ')。iOSを最新にして再度お試しください');
    }

    const outType = rec.mimeType || mime || 'video/mp4';
    const type = outType.indexOf('mp4') >= 0 ? 'video/mp4' : 'video/webm';
    resultBlob = new Blob(chunks, {type});
    const url = URL.createObjectURL(resultBlob);
    const ext = type === 'video/mp4' ? 'mp4' : 'webm';

    document.getElementById('preview').src = url;
    const dl = document.getElementById('dlLink');
    dl.href = url;
    dl.download = 'album.' + ext;
    document.getElementById('liveWrap').style.display = 'none';
    document.getElementById('result').style.display = 'block';
    fill.style.width = '100%';
    status.textContent = '完了!';
    // 診断: 一度も描画されなかったコマや展開失敗があれば報告する
    const notDrawn = segs.map((s, i) => (genDiag.segDraws[i] ? null : i + 1)).filter(v => v);
    if (notDrawn.length || genDiag.decodeFail){
      alert('診断情報\n描画できなかったコマ: ' + (notDrawn.join(', ') || 'なし') +
            '\nコマ展開の失敗: ' + genDiag.decodeFail + '回\n' +
            '動画に映らない写真があった場合、この内容を開発側に伝えてください');
    }
    document.getElementById('result').scrollIntoView({behavior:'smooth'});
  } catch (err){
    status.textContent = 'エラー: ' + err.message;
  } finally {
    genCleanup.forEach(fn => { try { fn(); } catch (e){} });
    document.body.classList.remove('busy');
    items.forEach(it => { if (it.bgCache){ it.bgCache.width = 0; it.bgCache = null; } });
    genDiag = null;
    if (wakeLock){ try { wakeLock.release(); } catch (e){} }
    // コマ画像と展開中のデータを解放
    segsRef.forEach(releaseSeg);
    items.forEach(it => { it.frameBlob = null; });
    bgmSrcs.forEach(s => { try { s.stop(); } catch(e){} });
    if (actx){ try { actx.close(); } catch(e){} }
    makeBtn.disabled = false;
  }
});

// ---------- プロジェクト保存(自動保存 + 複数プロジェクト) ----------
// ローカルファイルとして開いた環境ではIndexedDBが使えないことがあるため、
// 起動時に使える保存先を判定し、使えなければlocalStorageへ退避する
const KEY_PREFIX = 'photoMovie_';
let storageBackend = null;  // 'idb' | 'ls' | null
let currentProjId = null;
let currentProjName = '無題';

function idbOpen(){
  return new Promise((res, rej) => {
    const req = indexedDB.open('photoMovieDB', 1);
    req.onupgradeneeded = () => req.result.createObjectStore('session');
    req.onsuccess = () => res(req.result);
    req.onerror = () => rej(req.error);
  });
}
function idbTx(mode, fn){
  return idbOpen().then(db => new Promise((res, rej) => {
    const tx = db.transaction('session', mode);
    const out = fn(tx.objectStore('session'));
    tx.oncomplete = () => { db.close(); res(out && out.result); };
    tx.onerror = () => { db.close(); rej(tx.error); };
  }));
}

async function detectStorage(){
  try {
    await idbTx('readwrite', st => st.put('1', '__test'));
    await idbTx('readwrite', st => st.delete('__test'));
    storageBackend = 'idb';
    return;
  } catch (e){ /* IndexedDB不可 → localStorageを試す */ }
  try {
    localStorage.setItem('__t', '1');
    localStorage.removeItem('__t');
    storageBackend = 'ls';
  } catch (e){
    storageBackend = null;
  }
}

// バックエンド共通のキー単位読み書き
async function storGet(key){
  if (storageBackend === 'idb'){
    try { return await idbTx('readonly', st => st.get(key)); } catch (e){ return null; }
  }
  if (storageBackend === 'ls'){
    try {
      const raw = localStorage.getItem(KEY_PREFIX + key);
      return raw ? JSON.parse(raw) : null;
    } catch (e){ return null; }
  }
  return null;
}
async function storSet(key, val){
  if (storageBackend === 'idb') return idbTx('readwrite', st => st.put(val, key));
  if (storageBackend === 'ls') localStorage.setItem(KEY_PREFIX + key, JSON.stringify(val));
}
async function storDel(key){
  try {
    if (storageBackend === 'idb') return await idbTx('readwrite', st => st.delete(key));
    if (storageBackend === 'ls') localStorage.removeItem(KEY_PREFIX + key);
  } catch (e){}
}

// localStorage用: 保存サイズを抑えた画像(長辺1024px)を作りキャッシュする
async function makeSaveData(it){
  const im = await loadImage(it.midUrl);
  const s = Math.min(1, 1024 / Math.max(im.naturalWidth, im.naturalHeight));
  const c = document.createElement('canvas');
  c.width = Math.round(im.naturalWidth * s);
  c.height = Math.round(im.naturalHeight * s);
  c.getContext('2d').drawImage(im, 0, 0, c.width, c.height);
  const d = c.toDataURL('image/jpeg', 0.72);
  c.width = 0; c.height = 0;
  im.src = '';
  return d;
}

let saveTimer = null, restoring = false;
function scheduleSave(){
  if (restoring || !storageBackend || !currentProjId) return;
  clearTimeout(saveTimer);
  saveTimer = setTimeout(saveSession, 800);
}

async function buildPayload(){
  const ser = async it => {
    const s = {kind: 'image', comment: it.comment, thumbUrl: it.thumbUrl, fill: it.fill || 'blur',
               crop: it.crop || null};
    if (storageBackend === 'idb'){
      s.midBlob = it.midBlob || null;
    } else {
      try {
        if (!it.saveData && it.midUrl) it.saveData = await makeSaveData(it);
      } catch (e){ /* 画像なしで保存 */ }
      s.midData = it.saveData || null;
    }
    return s;
  };
  const out = [];
  for (const it of items){
    if (it.kind === 'video' || it.state !== 'ok') continue;
    if (it.kind === 'group'){
      const children = [];
      for (const c of it.children) children.push(await ser(c));
      out.push({kind: 'group', comment: it.comment, thumbUrl: it.thumbUrl, children});
    } else {
      out.push(await ser(it));
    }
  }
  return {
    savedAt: Date.now(),
    settings: {
      sec: document.getElementById('secSelect').value,
      fx: document.getElementById('fxSelect').value,
      textStyle: document.getElementById('textStyleSelect').value,
      motion: document.getElementById('motionSelect').value
    },
    // BGMは保存先によって実体を保持できない。その場合は名前だけ残し、復元時に選び直しを促す
    bgm: bgmFiles.length
      ? (storageBackend === 'idb'
          ? {name: bgmFiles[0].name || 'BGM', count: bgmFiles.length,
             blobs: bgmFiles.slice(), names: bgmFiles.map(f => f.name || 'BGM')}
          : {name: bgmFiles[0].name || 'BGM', count: bgmFiles.length,
             blobs: null, names: bgmFiles.map(f => f.name || 'BGM'), notSaved: true})
      : null,
    videoCount: items.filter(i => i.kind === 'video').length,
    items: out
  };
}

async function loadIndex(){
  return (await storGet('index')) || [];
}
async function saveIndexEntry(){
  // 保存したプロジェクトを常に先頭へ(日時が同じでも順序が保証される)
  const idx = (await loadIndex()).filter(p => p.id !== currentProjId);
  const okCount = items.filter(i => i.state === 'ok').length;
  idx.unshift({id: currentProjId, name: currentProjName, savedAt: Date.now(), count: okCount});
  await storSet('index', idx);
}

async function saveSession(){
  if (!storageBackend || !currentProjId) return;
  if (deletedIds.has(currentProjId)) return;  // 削除済みは書き戻さない
  try {
    const payload = await buildPayload();
    if (storageBackend === 'idb'){
      await storSet('proj_' + currentProjId, payload);
    } else if (storageBackend === 'ls'){
      try {
        await storSet('proj_' + currentProjId, payload);
      } catch (e){
        // 容量超過: 画像を除いた最小構成(コメント・並び・設定)で保存
        payload.items.forEach(s => {
          if (s.kind === 'group') s.children.forEach(c => { c.midData = null; });
          else s.midData = null;
        });
        payload.imagesDropped = true;
        try { await storSet('proj_' + currentProjId, payload); } catch (e2){ /* 断念 */ }
      }
    }
    await saveIndexEntry();
  } catch (e){ /* 保存失敗は作業継続を優先して無視 */ }
}

// 保存データを現在の作業状態へ展開する
// 復元したBlobはblob URLだと読めないことがあるため、ここでdataURLに変換して持つ
async function applyPayload(data){
  restoring = true;
  const reviveImg = async s => {
    let url = null, midBlob = null, saveData = null;
    if (s.midBlob){
      midBlob = s.midBlob;
      try { url = await blobToDataURL(s.midBlob); }
      catch (e){ url = null; }
    } else if (s.midData){
      url = s.midData;
      saveData = s.midData;
    }
    return {
      kind: 'image', file: null, url: url, midUrl: url, midBlob: midBlob, saveData: saveData,
      comment: s.comment || '', state: url ? 'ok' : 'failed', thumbUrl: s.thumbUrl,
      duration: 0, checked: false, retried: false, trimStart: 0, trimEnd: null,
      fill: s.fill || 'blur', crop: s.crop || null
    };
  };
  const out = [];
  for (const s of (data.items || [])){
    if (s.kind === 'group'){
      const children = [];
      for (const c of (s.children || [])) children.push(await reviveImg(c));
      out.push({kind: 'group', comment: s.comment || '', thumbUrl: s.thumbUrl, state: 'ok',
                children: children, duration: 0, checked: false});
    } else {
      out.push(await reviveImg(s));
    }
  }
  items = out;
  if (data.settings){
    document.getElementById('secSelect').value = data.settings.sec || '3';
    document.getElementById('fxSelect').value = data.settings.fx || 'fade';
    document.getElementById('textStyleSelect').value = data.settings.textStyle || 'cinema';
    document.getElementById('motionSelect').value = data.settings.motion || 'none';
  }
  // BGM(複数曲)を復元する。実体が保存されていない場合は解除して選び直しを促す
  const savedBgm = data.bgm || null;
  bgmFiles = (savedBgm && Array.isArray(savedBgm.blobs) ? savedBgm.blobs.filter(Boolean)
             : (savedBgm && savedBgm.blob ? [savedBgm.blob] : []));   // 旧形式も読み込める
  document.getElementById('bgmInput').value = '';
  bgmMissingName = null;
  if (!bgmFiles.length && savedBgm && savedBgm.name){
    const n = savedBgm.count > 1 ? savedBgm.name + ' ほか' + (savedBgm.count - 1) + '曲' : savedBgm.name;
    bgmMissingName = n;
  }
  renderBgmList();
  restoring = false;
  renderList();
  updateCount();
  renderStylePreview();
  if (data.imagesDropped){
    alert('保存容量の都合で写真の画像データは保存できませんでした。コメントと設定は復元済みなので、写真を選び直してください');
  } else if (data.videoCount){
    alert('動画素材(' + data.videoCount + '件)は容量の都合で保存されません。必要ならもう一度選択してください');
  }
}

function newProjectState(name){
  currentProjId = Date.now().toString(36) + Math.random().toString(36).slice(2, 6);
  currentProjName = name || ('プロジェクト ' + new Date().toLocaleDateString('ja-JP'));
  document.getElementById('projName').textContent = currentProjName;
  restoring = true;
  items = [];
  bgmFiles = [];
  bgmMissingName = null;
  document.getElementById('bgmInput').value = '';
  renderBgmList();
  restoring = false;
  renderList();
  updateCount();
  renderStylePreview();
}

async function openProject(id){
  await saveSession();  // 現在の作業を保存してから切り替える
  const data = await storGet('proj_' + id);
  if (!data){
    alert('このプロジェクトのデータが見つかりませんでした');
    return;
  }
  const idx = await loadIndex();
  const meta = idx.find(p => p.id === id);
  currentProjId = id;
  currentProjName = (meta && meta.name) || '無題';
  document.getElementById('projName').textContent = currentProjName;
  await applyPayload(data);
}

// 削除済みIDを記録し、進行中の自動保存が復活させないようにする
const deletedIds = new Set();

async function deleteProject(id){
  clearTimeout(saveTimer);            // 進行中の自動保存を打ち消す
  deletedIds.add(id);
  try {
    await storDel('proj_' + id);
    const idx = (await loadIndex()).filter(p => p.id !== id);
    await storSet('index', idx);
    // 実際に消えたか(一覧・データ本体の両方)を確認する
    const stillListed = (await loadIndex()).some(p => p.id === id);
    const stillStored = await storGet('proj_' + id);
    if (stillListed || stillStored) throw new Error('保存領域から削除できませんでした');
  } catch (e){
    alert('削除に失敗しました: ' + e.message);
    deletedIds.delete(id);
    renderProjList();
    return;
  }
  if (id === currentProjId) newProjectState();
  renderProjList();
}

async function renderProjList(){
  const listEl = document.getElementById('projList');
  listEl.innerHTML = '';
  const idx = await loadIndex();
  if (!idx.length){
    listEl.innerHTML = '<div class="warn">保存されたプロジェクトはまだありません</div>';
    return;
  }
  idx.forEach(p => {
    const row = document.createElement('div');
    row.className = 'projRow';
    const info = document.createElement('div');
    info.className = 'pn';
    const nm = document.createElement('div');
    nm.textContent = p.name + (p.id === currentProjId ? ' (編集中)' : '');
    const d = new Date(p.savedAt);
    const pd = document.createElement('div');
    pd.className = 'pd';
    pd.textContent = (p.count || 0) + '件 / ' + (d.getMonth() + 1) + '/' + d.getDate() +
                     ' ' + d.getHours() + ':' + String(d.getMinutes()).padStart(2, '0');
    info.append(nm, pd);
    const open = document.createElement('button');
    open.textContent = '開く';
    open.onclick = async () => { await openProject(p.id); closeProjModal(); };
    const del = document.createElement('button');
    del.textContent = '削除';
    del.className = 'pdel';
    let armed = false, armTimer = null;
    del.onclick = () => {
      if (!armed){
        // 1回目は確認状態にする(確認ダイアログが出ない環境でも確実に動く)
        armed = true;
        del.textContent = '本当に削除?';
        armTimer = setTimeout(() => { armed = false; del.textContent = '削除'; }, 4000);
        return;
      }
      clearTimeout(armTimer);
      del.disabled = true;
      del.textContent = '削除中...';
      deleteProject(p.id);
    };
    row.append(info, open, del);
    listEl.append(row);
  });
}

const projModal = document.getElementById('projModal');
function closeProjModal(){ projModal.classList.remove('show'); }
document.getElementById('projBtn').addEventListener('click', () => {
  renderProjList();
  projModal.classList.add('show');
});
document.getElementById('projClose').addEventListener('click', closeProjModal);
projModal.addEventListener('click', e => { if (e.target === projModal) closeProjModal(); });
document.getElementById('projNew').addEventListener('click', async () => {
  await saveSession();
  const name = prompt('新しいプロジェクトの名前', 'プロジェクト ' + new Date().toLocaleDateString('ja-JP'));
  if (name === null) return;
  newProjectState(name || undefined);
  closeProjModal();
});
document.getElementById('projRename').addEventListener('click', async () => {
  const name = prompt('プロジェクト名', currentProjName);
  if (!name) return;
  currentProjName = name;
  document.getElementById('projName').textContent = name;
  await saveIndexEntry();
  renderProjList();
});

async function startup(){
  await detectStorage();
  // 旧バージョン(単一保存)のデータをプロジェクトとして引き継ぐ
  try {
    let legacy = null;
    if (storageBackend === 'idb'){
      legacy = await idbTx('readonly', st => st.get('current'));
    } else if (storageBackend === 'ls'){
      const raw = localStorage.getItem('photoMovieSession');
      legacy = raw ? JSON.parse(raw) : null;
    }
    if (legacy && legacy.items && legacy.items.length){
      const legacyId = 'legacy_' + Date.now().toString(36);
      await storSet('proj_' + legacyId, legacy);
      const idx = await loadIndex();
      idx.push({id: legacyId, name: '以前の作業データ', savedAt: legacy.savedAt || Date.now(),
                count: legacy.items.length});
      idx.sort((a, b) => b.savedAt - a.savedAt);
      await storSet('index', idx);
      if (storageBackend === 'idb') await idbTx('readwrite', st => st.delete('current'));
      else if (storageBackend === 'ls') localStorage.removeItem('photoMovieSession');
    }
  } catch (e){}
  // 最新プロジェクトを開くか確認
  const idx = await loadIndex();
  if (idx.length){
    const latest = idx[0];
    if (confirm('前回のプロジェクト「' + latest.name + '」(' + (latest.count || 0) + '件)を開きますか?\nキャンセルすると新規プロジェクトで始めます(保存済みデータは残ります)')){
      const data = await storGet('proj_' + latest.id);
      if (data){
        currentProjId = latest.id;
        currentProjName = latest.name;
        document.getElementById('projName').textContent = currentProjName;
        await applyPayload(data);
        return;
      }
    }
  }
  newProjectState();
}

['secSelect', 'fxSelect', 'textStyleSelect', 'motionSelect'].forEach(id =>
  document.getElementById(id).addEventListener('change', () => { scheduleSave(); updateEstimate(); }));
startup();

// ---------- 保存 / 共有 ----------
document.getElementById('shareBtn').addEventListener('click', async () => {
  if (!resultBlob) return;
  const ext = resultBlob.type === 'video/mp4' ? 'mp4' : 'webm';
  const file = new File([resultBlob], 'album.' + ext, {type: resultBlob.type});
  if (navigator.canShare && navigator.canShare({files: [file]})){
    try { await navigator.share({files: [file]}); } catch(e){ /* キャンセル時は何もしない */ }
  } else {
    alert('この環境では共有できません。「ファイルとしてダウンロード」をご利用ください。');
  }
});
</script>
</body>
</html>
