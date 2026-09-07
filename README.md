# Anvil TW — 從 RD 到量產：十一關，一條線

公司流程手冊。靜態網頁，無建置流程、無後端；內容在 data.json，網頁上可直接編輯後匯出。

## 發布到 GitHub Pages

```bash
git init
git add .
git commit -m "流程手冊 v1"
git branch -M main
git remote add origin https://github.com/<你的帳號>/<repo>.git
git push -u origin main
```

推上去之後到 repo 的 **Settings → Pages**，Source 選 `Deploy from a branch`，
Branch 選 `main` / `/ (root)`，儲存。約一分鐘後網址會是：

```
https://<你的帳號>.github.io/<repo>/
```

## 怎麼改內容

### 方法一：在網站上改（推薦）

打開網站，右上角按 **編輯** → 左邊選要改的區塊 → 改欄位，後面的頁面會即時更新 →
按 **下載 data.json** → 到 GitHub 的 repo 頁面，點 `data.json`，按鉛筆旁的
「Upload files」或直接把檔案拖進去取代舊的 → commit。約一分鐘後網站就是新的。

編輯器只在你的瀏覽器裡跑，沒有後端，所以別人打開網址也改不到你的內容——
真正生效的唯一途徑是把 `data.json` 推回 repo。

### 方法二：直接改 data.json

所有內容都在 `data.json`，四個區段：`D`、`NOTE`、`GATE`、`EXTRA`。
圖表、表格、名冊、清單都是從這份資料算出來的，不用逐處修改。

| 要改什麼 | 改 data.json 的哪裡 |
| --- | --- |
| 人員、代號、職稱 | `D.roles` |
| 職掌與待辦清單 | `D.roles[].duties` |
| 關卡名稱、過關條件 | `D.stages` |
| 每一關誰做什麼 | `D.tasks` |
| 表格清單 | `D.docs`（`om` 欄位代表「省略、併入某張表」） |
| 系統 | `D.sys`（`tbd` 代表暫定，`now` 代表現況） |
| 四條規則 | `D.rules` |
| 檢驗分級 | `D.levels` |
| 採購四軌 | `D.tracks` |
| 月結時程與三張表 | `D.tl`、`D.three` |
| 待決策、風險 | `D.opens`、`D.risks` |
| 每一關的「最容易掉球」 | `NOTE` |
| 來料四條路、料號狀態、委外 | `EXTRA` |

每個欄位都有中英兩份：`zh` / `en`、`bz` / `be` 這樣的成對命名。改中文記得也改英文，
右上角的語言切換才不會露出中文。

## 檔案

| 檔案 | 用途 |
| --- | --- |
| `index.html` | 網頁本身，含編輯器。不用改 |
| `data.json` | 全部內容。要改就改這個 |
| `README.md` | 這份說明 |

## 檢查有沒有漏翻

```bash
node -e "
const fs=require('fs');const s=fs.readFileSync('index.html','utf8');
const a=s.indexOf('<script>')+8,b=s.lastIndexOf('</script>');
const j=JSON.parse(fs.readFileSync('data.json','utf8'));
const st={};
global.document={getElementById:i=>({set innerHTML(v){st[i]=v},setAttribute(){},set onclick(f){}}),
 querySelectorAll:()=>[],documentElement:{}};
global.IntersectionObserver=function(){return{observe(){}}};
global.fetch=()=>Promise.resolve({ok:true,json:()=>Promise.resolve(j)});
new Function(s.slice(a,b)+'\nglobalThis.sw=l=>{L=l;render();};')();
setTimeout(()=>{globalThis.sw('en');
Object.keys(st).forEach(k=>{
  const hit=[...new Set([...st[k].matchAll(/>([^<>]*[\u3400-\u9FBF][^<>]*)</g)].map(x=>x[1].trim()))];
  if(hit.length)console.log(k+':',hit.join(' | '));
});},300);"
```

沒有輸出就代表英文版沒有殘留中文。

## 目前的暫定與待確認

- 進銷存與採購系統尚未選定，文件中標為 **Odoo（暫定）**；選型只影響那一欄。
- 採購總表（PO Log）目前在 Excel，由採購維護。
- 代號尚未對應到真人。
- 其餘未定事項見網頁最後一節「待決策與風險」。
