# Domination プラグイン 導入・使い方ガイド

Minecraftサーバー向けの拠点占領型PvPミニゲームプラグインです。

## 前提条件

- **サーバーソフト**: Paperで動作確認済み。
- **動作確認済みバージョン**: 1.21.11
- **前提プラグイン**: CrackShotPlugin（URL=https://www.spigotmc.org/resources/crackshot-guns.48301/）←銃火器を多数追加するプラグイン。
- **リソースパック**: zipファイルに含まれているサーバーを用いる場合、サーバーリソースパックとして同じくzipファイルに含まれているリソースパック（m1kan's_for_1.21.11）を登録しています。サーバー参加時に「」
- **チーム**: 赤（red）／青（blue）の2チーム固定です。

## 導入方法

1. メインプラグインDomination-1.0.jarをこのGithubからダウンロードし、前提プラグインの**Crackshot.jar**をpluginsフォルダに配置します。
2. Github内にあるCrackShotフォルダをpluginsフォルダに配置します。これでゲームで使われるオリジナルの銃火器をサーバーに追加します。
3. サーバーを起動します。

### config.yml の設定例（実際の運用例）

```yaml
# ゲーム開始に必要な人数
required-players: 1
# カウントダウン秒数
countdown-seconds: 20

#ワールドのスポーン地点
spawn-location:
  world: "world"
  x: -9.0
  y: 38.0
  z: -120.0
  yaw: 0.0
  pitch: 0.0

# 各チームの座標 (world, x, y, z, yaw, pitch)
spawn-points:
  red:
    world: "world"
    x: 35.0
    y: 25.0
    z: 99.0
    yaw: 0.0
    pitch: 0.0
  blue:
    world: "world"
    x: 6.0
    y: 25.0
    z: -28.0
    yaw: 180.0
    pitch: 0.0

#　A~Eの各拠点の座標
points:
  A:
    world: world
    x: 13.0
    y: 21.0
    z: -1.0
    radius: 5.0
  B:
    world: world
    x: -13.0
    y: 8
    z: 10.0
    radius: 4.0
  C:
    world: world
    x: 1.0
    y: 20.0
    z: 45.0
    radius: 7.0
  D:
    world: world
    x: -17.0
    y: 8.0
    z: 69.0
    radius: 5.0
  E:
    world: world
    x: -31.0
    y: 31.0
    z: 105.0
    radius: 7.0
```

> 拠点は `A`〜`E` で1~5個で自由に設定できます（全て設定しなくても、記載した分だけ有効になります）。
### gui.yml の設定例（実際の運用例：兵科選択メニュー）

```yaml
# 兵科選択GUIの設定
class-menu:
  title: "&l兵科選択"
  rows: 3  # 行数 (1-6)
  items:
    Assault:
      slot: 11
      material: STONE_HOE
      name: "&c&l突撃兵"
      lore:
        - "&7近接攻撃特化のサブマシンガンをメイン武器とする兵科。"
        - "&eクリックで選択"
      commands:
        - "minecraft:clear %player% minecraft:enchanted_book"
        - "shot give %player% thompsonm1a1"
        - "shot give %player% coltm1911"
        - "give %player% minecraft:iron_ingot 208"
    Support:
      slot: 13 # 重複を避けるために13に変更
      material: GOLDEN_HOE
      name: "&b&l援護兵"
      lore:
        - "&7中距離攻撃特化のオートマチックライフルをメイン武器とする兵科。"
        - "&eクリックで選択"
      commands:
        - "minecraft:clear %player% minecraft:enchanted_book"
        - "shot give %player% m1garand"
        - "shot give %player% coltm1911"
        - "give %player% minecraft:gold_ingot 96"
        - "give %player% minecraft:iron_ingot 16"
    Sniper:
      slot: 15
      material: DIAMOND_HOE
      name: "&e&l狙撃兵"
      lore:
        - "&7遠距離攻撃特化のボルトアクションライフルをメイン武器とする兵科。"
        - "&eクリックで選択"
      commands:
        - "minecraft:clear %player% minecraft:enchanted_book"
        - "shot give %player% springfieldm1903"
        - "shot give %player% coltm1911"
        - "give %player% minecraft:gold_ingot 16"
        - "give %player% minecraft:iron_ingot 16"
```

- `commands` には、コンソール権限で実行されるコマンドを自由に記述できます。`%player%` にはクリックしたプレイヤー名が代入されます。**兵科によって変わる配布物の中身をこのファイルだけで自由にカスタマイズできます。**

## コマンド・権限

| コマンド | 説明 | 必要権限 |
|---|---|---|
| `/domination start` | ゲームを即座に開始（カウントダウン省略） | `domination.admin` |
| `/domination stop` | ゲームを強制終了し、その時点のスコアで勝敗を判定 | `domination.admin` |
| `/domination reload` | `config.yml` と `gui.yml` を再読込 | `domination.admin` |
※基本的に`domination.admin` の権限はOP権限を持つプレイヤーしか持てません。

## ゲームの流れ

### 1. 開始
- サーバー内のオンライン人数が `required-players` 以上になると、**自動的にカウントダウンが始まります**
- カウントダウン中（`countdown-seconds` 秒）は、残り10秒以下でタイトル表示、それ以外はチャット通知（10秒刻み）。人数が条件を下回ると自動中止されます。
- 管理者（）は `/domination start` でカウントダウンを飛ばして即座に開始することも可能です。
- 開始と同時に、オンラインプレイヤーはシャッフルされて**赤/青チームに交互に振り分け**られ、以下が行われます。
  - 対応チームのスポーン地点へテレポート
  - チームカラーの革防具（ヘルメット/レギンス/ブーツ）＋鉄チェストプレートを支給
  - 「あなたは【赤/青チーム】です」とタイトル表示
  - 数秒後、兵科選択用のエンチャント本が配布される

### 2. プレイ中
- **兵科選択**: 配布されたエンチャント本（「【兵科選択】」表示）を**右クリック**するとメニューが開き、選んだ兵科に応じたアイテムが支給されます（内容は `gui.yml` 次第）。
- **拠点占領（A〜E）**:
  - 拠点の範囲内に**片方のチームだけ**が留まり続けると占領カウントが進み（ボスバーで表示）、**15秒**で占領成立、**+2点**がそのチームに入ります。
  - 両チームが同時にいる、または誰もいなくなると占領カウントはリセットされます。
  - 拠点はホログラム表示（所有チームで色分け）とパーティクルの縁取りで視覚的に確認できます。
- **戦闘・キル**: 敵を倒すと**+1点**がキラーのチームに入り、キラーの個人キル数も加算されます。
- **死亡〜リスポーン**:
  - 死亡すると、透明化＋飛行可能な「観戦者モード」になります（この間、他プレイヤーから攻撃されません）。
  - アクションバーに「復活まで残り○秒」が表示され、**15秒後**に元のチームのスポーン地点へリスポーンし、装備・兵科選択の本が再配布されます。
- **アイテムドロップ禁止**: プレイ中はアイテムを手動でドロップできません。
- **スコアボード**: 画面右にサイドバーで残り時間・各拠点の所有チーム・両チームの累計勝利数・現在のスコア・自分のキル数が常時表示されます。
- **切断時**: ゲーム中に切断すると、その時点の装備が保持され、再接続時に自動で復元されます。

### 3. 終了
- 制限時間（開始時に15分固定でタイマーが起動）が0になるか、サーバー管理者によって`/domination stop` が実行されると試合終了。
- 最終スコアを比較し、多い方が勝利（同点は引き分け）。
- 全員がロビーへ戻されます。
- 拠点はリセットされ、**10秒後**に人数条件を満たしていれば、次の試合の開始までのカウントダウン（デフォルトは20秒。configで変更可能）が始まります。

## 補足

- 赤/青の**累計勝利数はサーバー再起動まで保持**され続けます（試合ごとにはリセットされません）。
- 制限時間は**15分固定**です。
