# app フォルダ — 全体ルール

## これは何か

自分や家族のための小さなアプリを、1アプリ1フォルダで置いておくリポジトリ。全部 PWA（HTML/CSS/JS のみ、サーバーなし、ログインなし）。公開は GitHub Pages で、URLは `https://tim-yuki-lub.github.io/app/<フォルダ名>/`。

実務（仕事）のコードは置かない。個人の内面の話は `mind`、仕事・事業は `business`（旧 funeral）。

## 構成

```
app/
├── index.html                 ← アプリ一覧（アプリを足したら1行足す）
├── README.md                  ← アプリの表（同じく1行足す）
├── .github/workflows/pages.yml ← main に入ると GitHub Pages へ自動で出る
└── <アプリ名>/                 ← 1アプリ1フォルダ。中身は自己完結させる
    ├── README.md              ← そのアプリのやりたいこと・方針メモ
    ├── index.html             ← 画面とロジック。基本はこの1ファイルに全部書く
    ├── manifest.webmanifest   ← アプリ名・アイコン・全画面表示の設定（パスは全部 ./ 相対）
    ├── sw.js                  ← オフライン用キャッシュ。中身を変えたら VERSION を上げる
    └── icons/                 ← icon-192 / icon-512 / icon-512-maskable / apple-touch-icon(180)
```

## アプリを作るとき・直すときの約束

- **データは使う人のスマホの中（localStorage）だけ。** サーバーやログインは作らない。漏れて困るものを扱わない前提。
- **iPhone の Safari で「ホーム画面に追加」して使う**のが基本。確認する項目：`apple-touch-icon`、`apple-mobile-web-app-capable`、`viewport-fit=cover` と safe-area、manifest の `display: standalone`、`start_url`/`scope` が `./`。
- **LINE の中のブラウザで開かれる前提も入れる。** URL に `?openExternalBrowser=1` を付けると LINE が Safari で開くので、LINE内と判定したらそのリンクを出す（`shitaku/index.html` 参照）。
- **変更したら `sw.js` の VERSION と `index.html` 内の VERSION を上げる。** 上げないと、ホーム画面から開いたときに古いまま出ることがある。
- 動作確認は Playwright（Chromium）で iPhone サイズのスクショを撮って見る。`node` と `python3`（Pillow）が使える。
- アイコンは Pillow で生成する（元のスクリプトはこのファイルの末尾）。

## 各アプリのメモ

### shitaku（明日の支度）— 姉用

- 計算：出発 ← 朝の準備（既定60分）＋朝タスク ← 起床 ← 睡眠（既定8時間）← 就寝 ← お風呂（就寝の2〜3時間前。既定 最短120分・長くて180分）← お風呂の前のタスク。
- タスクの置き場は3つ：「お風呂の前」「お風呂のあと、寝るまで」「朝、出発の前」。お風呂のあとのタスクが2時間を超えると、お風呂をその分早める。
- 「今夜」は昼12時を境に切り替える（午前中に開いたら前の晩の計画を見せる）。日ごとのオンオフは14日分だけ残す。
- チップはタップでオンオフ、長押しで編集・削除。
- 時間の枠（朝の準備・睡眠・お風呂から寝るまで）は候補を持ち（`state.frames`）、タップでその日の長さを選ぶ（`days[日付].pick`）。長押し／右クリックで編集・削除。設定画面は「いつも」の候補を書きかえる。
- **これから足したいこと・方針は `shitaku/README.md` にまとめている。** 思いついたことはそこに足す。着手前に必ず読む。

## GitとGitHubの扱い（ユーザーは何もしない）

ユーザーはコミット・プッシュ・PR・マージを自分ではしない。すべてClaudeがやり、確認も取らない。報告ではgitの用語を使わず「GitHubに反映しました」とだけ言う。

- **クラウド（スマホ）**：作業のまとまりが終わるたびに、指定されたブランチにコミット→push→mainへのPRを作成→そのままマージ（merge方式。squashにしない）までを1セットで行う。続けて作業する前に `origin/main` を取り込んでから始める。
- **Mac**：`~/Claude/app` の main で直接作業する。5分ごとの自動同期（business の `scripts/sync/`。`~/Claude/` 直下のフォルダは全部対象）があるので、Claudeもユーザーも何もしなくてよい。すぐ反映したいときは `bash ~/Claude/business/scripts/sync/sync.sh "$PWD"`。
- **ぶつかったとき**（Macに「同期を直して」の通知が出た／ユーザーにそう言われた）：Claudeが `git pull --rebase origin main` で解決し、両方の変更を残す。どちらかを捨てるしかない場合だけユーザーに聞く。解決後は `.git/sync-conflict` を消して push する。

## 付録：アイコン生成スクリプト（shitaku）

```python
from PIL import Image, ImageDraw
def make(size, maskable=False):
    S=size*4; img=Image.new('RGBA',(S,S)); d=ImageDraw.Draw(img)
    top=(28,31,58); bot=(63,63,168)
    for y in range(S):
        t=y/S; d.line([(0,y),(S,y)], fill=tuple(int(top[i]*(1-t)+bot[i]*t) for i in range(3))+(255,))
    k=0.72 if maskable else 0.86; cx=cy=S/2
    r=S*0.30*k; mx=cx-S*0.04*k; my=cy-S*0.06*k
    moon=Image.new('L',(S,S),0); md=ImageDraw.Draw(moon)
    md.ellipse([mx-r,my-r,mx+r,my+r],fill=255)
    md.ellipse([mx-r+S*0.17*k,my-r-S*0.10*k,mx+r+S*0.17*k,my+r-S*0.10*k],fill=0)
    img.paste((255,246,214,255),(0,0),moon)
    sx=cx+S*0.22*k; sy=cy+S*0.26*k; sr=S*0.10*k
    sun=Image.new('L',(S,S),0); sd=ImageDraw.Draw(sun); sd.pieslice([sx-sr,sy-sr,sx+sr,sy+sr],180,360,fill=255)
    img.paste((255,178,89,255),(0,0),sun)
    d.line([(sx-sr*1.5,sy),(sx+sr*1.5,sy)],fill=(255,178,89,255),width=int(S*0.018))
    return img.resize((size,size),Image.LANCZOS)
make(192).save('icons/icon-192.png'); make(512).save('icons/icon-512.png')
make(512,True).save('icons/icon-512-maskable.png'); make(180).save('icons/apple-touch-icon.png')
```
