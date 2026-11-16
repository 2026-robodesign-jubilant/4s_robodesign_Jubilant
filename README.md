# 4s_robodesign_Jubilant

2026 ロボット設計製作論 チーム Jubilant の**統合リポジトリ**です。
コードは機能ごとに別のリポジトリに分け、このリポジトリには「全部をそろえるための一覧（[`jubilant.repos`](jubilant.repos)）」と、チーム全体の説明を置きます。

CRANE-X7 に D435i を付け、**四隅に ArUco マーカーを印刷した紙**を見つけて、書かれた英語の文書を Claude で日本語にして VOICEVOX で読み上げ、紙の欄にはんこを押すことを目指しています。

## リポジトリ一覧

すべて Organization [2026-robodesign-jubilant](https://github.com/2026-robodesign-jubilant) にあります。

| リポジトリ | 担当 | 内容 |
|---|---|---|
| [ocr](https://github.com/2026-robodesign-jubilant/ocr) | OCR 担当 | ArUco で紙を真上から見た画像に補正 → Claude で日本語の要点 → VOICEVOX で読み上げ（`ocr_recognizer` ほか） |
| [stamp](https://github.com/2026-robodesign-jubilant/stamp) | はんこ担当 | CRANE-X7 で指定した座標にはんこを押す（2024 P班 `crane_x7_stamp` の Jazzy 版） |
| （作成予定） | 眠気担当 | 眠気とうなずき。いまはこのリポジトリの `feat/crane-x7-drowsiness` ブランチ |
| （作成予定） | 追従担当 | 紙の追従 |
| （作成予定） | ライト担当 | ライト |
| このリポジトリ | 全員 | `jubilant.repos`、チーム全体の説明。全体を起動する launch と、班どうしで受け渡す `.msg` もここに置く予定 |

CRANE-X7 本体は rt-net 公式の [crane_x7_ros](https://github.com/rt-net/crane_x7_ros) と [crane_x7_description](https://github.com/rt-net/crane_x7_description)（どちらも `jazzy` ブランチ）を `jubilant.repos` から取ってきます。自分のリポジトリにコピーして入れないでください（同じパッケージが 2 つあると colcon がビルドを止めます）。

## 開発環境

- Ubuntu 24.04（Pop!_OS 可）＋ **ROS 2 Jazzy**
- `git`、`python3-vcstool`（`vcs` コマンド。無ければ `sudo apt install python3-vcstool`）

## はじめに：全員分のコードを手元にそろえる（最初の 1 回だけ）

ワークスペース `~/ros2/jubilant_ws` を作り、このリポジトリを `src/jubilant` に clone してから、一覧にあるリポジトリをまとめて取ってきます。

```bash
mkdir -p ~/ros2/jubilant_ws/src && cd ~/ros2/jubilant_ws
git clone https://github.com/2026-robodesign-jubilant/4s_robodesign_Jubilant.git src/jubilant
vcs import src < src/jubilant/jubilant.repos   # 一覧のリポジトリを src/ に全部 clone する
source /opt/ros/jazzy/setup.bash
rosdep install -r -y -i --from-paths src --rosdistro jazzy   # 足りない ROS のパッケージを入れる（sudo のパスワードを聞かれる）
colcon build --symlink-install
source install/setup.bash
```

- **全部を最新にする**：`vcs pull src`
- **どこに未コミットの変更があるか見る**：`vcs status src`
- 1 つのリポジトリだけ使いたいときは、そのリポジトリだけを `src/` に clone すれば動きます（使い方は各リポジトリの README）。

初めて git を使う人は、最初に名前とメールを設定しておきます（コミットに記録されます）。

```bash
git config --global user.name  "あなたの名前"
git config --global user.email "あなたのメール"
```

## 自分のリポジトリを作ったら

1. Organization の中に作る（https://github.com/organizations/2026-robodesign-jubilant/repositories/new）
2. このリポジトリの `jubilant.repos` に 1 項目足し、上の「リポジトリ一覧」の行を書き換えて PR を出す

```yaml
  リポジトリの名前:
    type: git
    url: https://github.com/2026-robodesign-jubilant/リポジトリの名前.git
    version: main
```

## 毎回の開発の流れ（GitHub 初心者向け・ここが一番大事）

ルール: **`main` ブランチで直接作業しない。** 自分用の「ブランチ」を切って作業し、
できたら「プルリクエスト(PR)」で `main` に取り込みます。

```bash
# 1. 最新を取り込む（他の人の変更を反映）
git checkout main
git pull

# 2. 作業用ブランチを main から切る（名前は 種類/内容 が分かりやすい）
git checkout -b feat/my-change

# 3. ファイルを編集したら、変更を記録(commit)する
git add <変更したファイル>        # 例: git add ocr_recognizer/
git commit -m "何をしたかの説明"

# 4. GitHub に自分のブランチを上げる(push)
git push -u origin feat/my-change
```

5. ブラウザで GitHub のリポジトリを開くと「Compare & pull request」ボタンが出るので、
   それを押して **Pull Request** を作成。他のメンバーが確認してから `main` に Merge します。

用語メモ:
- **commit** … 変更の「セーブポイント」を作ること（まだ自分の PC の中だけ）
- **push** … セーブした変更を GitHub に送ること
- **Pull Request (PR)** … 「この変更を main に入れていい？」とチームに出す提案

## コミットしてはいけないもの（設定済み）

次のものは `.gitignore` で除外済みなので、`git add` しても上がりません（各リポジトリにも同じ設定を入れてください）（上げると壊れる・肥大するため）。

- `build/` `install/` `log/` … colcon が自動生成する
- `.venv/` `__pycache__/` … Python の仮想環境・中間ファイル
- `target/` … Rust（cargo）のビルド結果
- `*.wav` … VOICEVOX で作った音声ファイル

**API キーは絶対にコミットしない。** コードには書かず、環境変数（`ANTHROPIC_API_KEY` など）から読む。

## 困ったとき

- **push したら main に拒否された** → ルール通りです。ブランチを切ってから push し、PR で取り込みましょう。
- **他の人の変更とぶつかった(conflict)** → 慌てず `git pull` してから落ち着いて解消。分からなければチームで相談。
