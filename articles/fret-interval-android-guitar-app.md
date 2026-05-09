---
title: "Kotlin未経験のAIエンジニアが、Claude Codeで3日でギター練習Androidアプリを作った"
emoji: "🎸"
type: "tech"
topics: ["claudecode", "android", "kotlin", "jetpackcompose", "個人開発"]
published: true
---

:::message
この記事は、**個人開発**で作っているAndroid向けギター練習アプリ FretInterval の開発記です。普段は AI エンジニア／データサイエンティストとして働いていて、Kotlin もAndroid開発も未経験の私が、Claude Code に書かせて3日でたたき台を作り、その後2ヶ月かけて磨き込んでいる過程を共有します。

Google Play非公開・USBデバッグでの自端末配布で、いまのところ「自分専用」として運用しています。同じように、専門外領域の課題を AI 支援で解決したい人の参考になればと思って書いています。
:::

## はじめに

Kotlinを一行も書いたことがありません。Android アプリ開発の経験もありません。

それでも、ギター練習用の Android アプリ FretInterval を Claude Code で作って、いまは毎日使っています。たたき台ができるまで実質3日、安定して使えるようになるまで約2ヶ月でした。コードは私はほぼ1行も書いていません。

![FretInterval デモ](/images/fret-interval-android-guitar-app/demo.gif)
*ルート音を C → D に変えると、指板上の度数表示がリアルタイムで切り替わる。*

この記事は、その開発記録です。

## なぜ作ったか

ギターを始めて約1年。スケールやコードの構成音が、指板の上のどこにあるのかが、いつまで経っても覚えられませんでした。

教則本もWebの図解もたくさんあります。けれど私が欲しかったのは、**ギターを手元に置けない時間**に頭の中で復習できる道具でした。電車の中、寝る前、昼休み。そういう「弾けない隙間時間」にスマホで指板を眺めて、ルート音を変えながら「いま4度はここ」「♭3度はここ」と確認できる動線が欲しかった。

教則本は紙の上で指板を切り替えられないし、Web の図解は情報量が多すぎて自分の今の練習に絞り込めない。**自分の練習動線にちょうど合う道具**は見つけられませんでした。

ところで、私の本職は AI エンジニア／データサイエンティストです。Kotlin はおろか、Android アプリ開発の経験はゼロ。普通なら「ないなら作ろう」と思っても、Kotlin と Android Studio と Jetpack Compose を一から学ぶコストを考えて諦めるところでした。

ところが、いまは Claude Code があります。

「LLM にコードを書かせれば、自分が書けない言語のアプリでも作れるんじゃないか？」と思って試したのが、この記事の出発点です。

## Claude Code で3日で「使える形」になった

実際の git log から時系列を抜き出すとこうです。

| 日 | コミット | 内容 |
|----|---------|------|
| 3/14 (Day 1) | 初コミット〜README | 仕様の骨格を Claude Code と詰める |
| 3/15 (Day 2) | v1.0 実装 → 三者設計レビュー → **v1.1 全フェーズ実装＋バグ修正完了** | 指板ビジュアライザー・メトロノーム・トレーニング・★評価・Room永続化が**この日のうちに全部入った** |
| 3/22 (Day 9) | 音質改善・タップ精度向上・横画面対応 | 実機検証で気づいた問題を反映 |
| 3/22 〜 5/9 | UX磨き込み・サウンドバンク差し替え 等 | 細部の磨き込み（約2ヶ月） |

Day 2 のコミット `919a37a (v1.1 全フェーズ実装完了＋バグ修正)` で、メトロノーム + トレーニングモード + ★評価 + Room 永続化までの **v1.1 骨格が全部入った**ことになります。Kotlin が読めない私から見ても、ビルドが通って実機で動いていました。

私が書いた Kotlin は、現時点で**ゼロ行**です。
全コードは Claude Code との対話で生成・修正したもので、私の役割は次の3つでした。

1. **何を作りたいかを言語化する**（仕様書、UIモックアップ、機能優先順位）
2. **設計判断する**（どの API を使うか、どこを妥協するか、どこを尖らせるか）
3. **実機で動かしてフィードバックする**（onTap 遅延、不可視ノートのタップ問題、ルート音のオクターブ崩壊などは、実機テストでしか出ない）

「コードを書く」と「コードを設計してレビューする」は別のスキルだ、ということを Claude Code が教えてくれました。

### 効いたこと

- **仕様書を先に整えた**: 機能の細部を Markdown でまとめて Claude Code に渡すと、生成コードの精度が一気に上がる。FretInterval では `FretInterval_仕様書_v2.md` を 1,100行規模で書いた
- **「三者設計討議」をさせた**: ギタリスト視点・ユーザー視点・実装者視点をそれぞれ別の対話で出させ、矛盾を炙り出してから本実装に進む。コミット `1594629 (Update spec and v1.1 plan based on three-party review)` がその痕跡
- **`tasks/lessons.md` にハマりを残した**: 同じ罠を Claude Code に踏ませない。`fallbackToDestructiveMigration()` 禁止、`@file:OptIn` を使う、`onTap` ではなく `onPress` を使う、などをルール化して以後の生成に効かせた
- **コミット粒度を保たせた**: 1機能=1コミットを Claude Code に守らせると、後で git diff で挙動を読み解きやすい

### 効かなかったこと

- **実機の体感は伝わらない**: メトロノームのタップ遅延（onTap が数百ms 遅れて発火する体感ラグ）は、私が実機で気づいて Claude Code にフィードバックする必要があった
- **音楽理論の細部**: 「Perfect Fourth = 5 semitones」のような名前と数値の不一致は、私側で確認しないと検出できなかった

## 移動中にスマホで Claude Code 開発する仕組みを作った

開発を続けるうちに気づいたのが、**「移動時間に学習する」と決めて作ったアプリなのだから、移動時間に開発もできれば最高じゃないか**ということでした。

開発を始めて2週間ほど経った 3/27（Day 14）、GitHub Actions で APK の自動ビルドを設定しました（コミット `8430f4b`）。

```yaml
# .github/workflows/build.yml
name: Build APK

on:
  push:
    branches: [main, 'claude/**']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: "17"
          distribution: temurin
      - uses: gradle/actions/setup-gradle@v4
      - run: chmod +x ./gradlew && ./gradlew assembleDebug
      - uses: actions/upload-artifact@v4
        with:
          name: FretInterval-debug
          path: app/build/outputs/apk/debug/app-debug.apk
```

ポイントは `branches: [main, 'claude/**']` で、**Claude Code が作る `claude/...` ブランチでも自動ビルドが走る**ようにしてあることです。

これで、開発フローがこうなりました。

1. 電車の中で、スマホから Claude Code に「ここを直して」と依頼する
2. Claude Code が `claude/some-branch` にコミット & プッシュする
3. GitHub Actions が APK をビルドして artifact に上げる
4. 駅に着いた頃、スマホで artifact をダウンロードしてインストール
5. 練習の合間に新しい挙動を確認、また依頼する

つまり、**移動時間に指板で度数を覚えるアプリを、移動時間に Claude Code で作っていた**、という入れ子構造になりました。PCを開かない日でも、スマホ1台で開発が回ります。

最初は「ローカルで `./gradlew assembleDebug` できるんだから CI なんていらないでしょ」と思っていました。けれど AI 支援開発と CI を組み合わせると、**PCがなくても開発できる**という別の意味が出てきます。Claude Code がコードを書き、GitHub Actions が APK を作り、私はスマホで結果を確かめる。書き手も、ビルド環境も、検証端末も、それぞれ別の場所にあって、私はそれを「依頼」と「確認」で繋ぐだけ。コードを「書く」必要も、PCを開く必要もなくなりました。

これは Claude Code 単体ではなく、**Claude Code × Git × CI/CD** の組み合わせで初めて成立する体験で、個人開発のあり方を一段階変えたように感じています。

## FretInterval で何ができるか

機能は3つに整理できます。

### 1. 指板ビジュアライザー

ルート音と「7モード（Ionian〜Locrian） / 4コード（Maj7, m7, Dom7, m7b5） / 2ペンタトニック」のいずれかを選ぶと、指板上に該当音が色分けで表示されます。度数表示と音名表示は切り替え可能です。

![指板画面（縦）](/images/fret-interval-android-guitar-app/fretboard-portrait.png)
*縦画面は5フレット拡大表示。スクロールで全フレット移動。*

![指板画面（横）](/images/fret-interval-android-guitar-app/fretboard-landscape.png)
*横画面は12〜22フレット全体表示。*

カラースキームはこうなっています。

| 役割 | 色 |
| --- | --- |
| Root | 赤 |
| 3rd | 青 |
| 5th | 緑 |
| 7th | 黄 |
| Character Note | 紫 |
| Avoid Note | オレンジ |

### 2. メトロノーム

BPM 30〜250、4拍子・3拍子・6/8拍子に対応し、4分・8分・3連・16分音符を切り替えられます。タップテンポで現在の弾いているテンポを取り込んだり、スピードトレーナーで指定小節ごとに +1〜+5 BPM 自動加速したりできます。

![メトロノーム画面](/images/fret-interval-android-guitar-app/metronome.png)

### 3. トレーニング

メトロノームに合わせて、画面上の指板をタップしていくドリルです。

- **Sandboxモード**: 自由タップ → 度数をリアルタイム表示
- **Level 0**: 0〜5フレット → 全フレットの音名タップ（ヒント有/無）
- **Level 1**: R/5、コードトーン、ペンタ、スケール7音 のインターバルタップ

正答率 70% / 85% / 95% で★1/2/3が付き、Room で永続化されます。

![Level選択画面](/images/fret-interval-android-guitar-app/level-select.png)

ここから先は技術の話です。

## 「自分用」と割り切った理由

このアプリは Google Play に出していません。USBデバッグで自分の端末にだけ入れています。Kotlin を覚えるためでもなく、誰かに使ってもらうためでもない。**ギター練習を続けるための個人的な道具**として作っています。

Play配信を前提にすると、考えなければいけないことが一気に増えます。クローズドテスト、課金、サポート、プライバシーポリシー、対応端末の幅。これらは正しい工程ですが、いま私が欲しいのは「明日朝の練習で使える道具」です。

「for me」と割り切ったことで、機能の優先順位が驚くほどクリアになりました。

- ルート変更は片手で完結できるか？
- 練習中に画面を見て1秒以内に「いま3度を押してる」と分かるか？
- メトロノームは、寝る前にイヤホンなしでも気にならない音か？

この3つに答えるためだけに作ればいい、と決めると迷いがなくなります。誰かに使ってもらうために抽象化したり、設定項目を増やしたりしないで済みます。

もし将来 Play に出すとしても、それは「自分が使い込んできた道具を、外向けに整える」という工程になります。先に整える必要はありません。

## 技術ハイライト3つ

ここから記事の山場です。コード自体は Claude Code が書いたものですが、設計判断は私と Claude Code の対話で固めたものです。特に面白かった3点を紹介します。

### 1. 12音を整数で扱う音楽理論エンジン

ギターは複雑な楽器に見えます。6本の弦、22フレット、5オクターブ、コードもスケールもモードもある。

けれど内部で動いている音楽理論は、12個の整数（0〜11）でほぼ全部まわります。

```kotlin
// 0=C, 1=C#, 2=D, ... 11=B
// ルートに対するインターバルを計算: (target - root + 12) % 12
fun calculateInterval(root: Int, target: Int): Int =
    (target - root + 12) % 12
```

たった1行のこの関数が、アプリ全体を支える計算の中心です。

スケールやコードは、インターバル（半音数）のリストとして表現できます。

```kotlin
// Ionian（メジャースケール）= R, 2, 3, 4, 5, 6, 7
val ionian = listOf(0, 2, 4, 5, 7, 9, 11)

// Maj7コード = R, 3, 5, 7
val maj7 = Chord("Maj7", listOf(0, 4, 7, 11))

// マイナーペンタトニック = R, ♭3, 4, 5, ♭7
val minorPentatonic = listOf(0, 3, 5, 7, 10)
```

指板上のあるノートを表示すべきかは、こう判定します。

```kotlin
fun shouldShow(noteValue: Int, root: Int, intervals: List<Int>): Boolean =
    calculateInterval(root, noteValue) in intervals
```

ここで一度ハマりました。仕様書に「avoid note は Ionian の 4度」と書いていたので semitone=4 だと思ったら、Perfect Fourth = 5 semitones が正解でした。「度数の名前」と「semitone値」は別物で、4度音程は完全4度（5半音）です。Claude Code もこのズレを最初は踏みかけていて、私側で気づいて修正を依頼した箇所です。

この設計のおかげで、スケールやコードの追加は「`List<Int>` を1つ書けば終わり」になりました。後述の指板描画もこのリストを引数に受けるだけです。

### 2. Compose Canvas で指板を描く

指板は、画像を貼り付けて済む UI ではありません。ルートが変わるたび、スケールが変わるたび、表示する音が動的に変わります。

選んだのが Jetpack Compose の Canvas API です。私自身は Compose を触ったこともなかったので、「動的な指板を描きたい」という要件だけ Claude Code に伝えて、Canvas を使う方針も含めて提案させました。

縦画面と横画面で座標系が違うのが面倒なポイントです。縦は5フレットを大きく拡大、横は12〜22フレット全体を見せたい。これを座標変換でなく、レイアウトロジックの分岐で書いています。

```kotlin
@Composable
fun FretboardCanvas(
    state: FretboardState,
    isLandscape: Boolean,
    onNoteClick: (FretNote) -> Unit,
    /**
     * true のとき、onPress（指が触れた瞬間）でタップ検出し、
     * グリッドベースのヒット判定を使う
     */
    trainingMode: Boolean = false,
    // 他の引数は省略
) {
    val density = LocalDensity.current
    val notePxRadius = with(density) { 14.dp.toPx() }

    Canvas(modifier = Modifier
        .fillMaxSize()
        .pointerInput(trainingMode) {
            // タップ判定（後述）
        }
    ) {
        drawStrings()
        drawFretWires()
        drawFretMarkers()
        drawNotes(state.fretNotes, notePxRadius)
    }
}
```

描画順は「弦線 → フレットワイヤー → ポジションマーカー → ノート」の固定順です。ノートは円＋テキストの重ね描きで、Root=赤、3rd=青、5th=緑、7th=黄、Character=紫、Avoid=オレンジ。

設計面で意識したのは、寸法計算を Composable 側に寄せることでした。`Canvas { ... }` ブロック自体は `DrawScope`（`Density` を継承）なので `dp.toPx()` は使えますが、毎フレーム描画されることを考えると、密度依存の値はコンポーザブル側で `LocalDensity` から1回だけ取得して渡す方が綺麗にまとまります。本アプリでは `FretboardMeasure` というレイアウト計算用クラスを `remember(...)` で保持して、画面サイズや density が変わったときだけ再計算するようにしています。

ここの設計は、最初の Claude Code 出力では Canvas 内に density 依存のコードが散らかっていて、私から「再描画コストを下げたい」と要求して整理してもらいました。AI 任せにしっぱなしにすると、動くけれど読みづらいコードが残ります。

### 3. AudioTrackで±0.01%のBPM精度

メトロノームに、なぜ精度が必要か。

スピードトレーナーで「8小節ごとに +2 BPM」のような自動加速をかけるとき、精度が低いと BPM が揺れて練習にならないからです。長く回すほど誤差が積み上がるので、最初の数拍は気にならなくても、3分後にはずれているということが起こります。

最初は SoundPool で実装してみたのですが、再生スケジューラに依存するため精度が出ませんでした。AudioTrack の MODE_STREAM に切り替え、自分でフレームを書き込む方針にしました。「Bresenham 累積誤差管理」というアイデアは Claude Code が提案してきたもので、私はその時はじめて Bresenham を音声タイミング制御に使う発想を知りました。

```kotlin
// MetronomeEngine.runLoop() の中、ループのたびに計算する
private fun runLoop(track: PcmTrack) {
    var totalSubdivFrames = 0.0  // 書き込むべき累積フレーム数（浮動小数）
    var writtenFrames = 0L       // 実際に書き込んだ累積フレーム数

    while (running) {
        val framesPerBeat = SAMPLE_RATE * 60.0 / bpm
        val framesPerSubdiv = framesPerBeat / pattern.subdivisions

        // この細分割で書き込むべきフレーム数を Bresenham 式で計算
        totalSubdivFrames += framesPerSubdiv
        val targetFrame = totalSubdivFrames.toLong()
        val thisSubdivFrames = (targetFrame - writtenFrames).toInt()
        writtenFrames = targetFrame

        // クリック音 + サイレンスを thisSubdivFrames 分だけ書き込む
        // ...
    }
}
```

ポイントは `Double` で累積していく `totalSubdivFrames` と、整数で進んでいく `writtenFrames` の差分を毎回取ることです。ある瞬間で `framesPerSubdiv` が小数（例: 8268.75）になっても、その端数は次の細分割に持ち越されるため、長時間動かしても誤差が積み上がりません。

「±0.01%」という数字は、サンプル数ベースの計算上の理論値です（Bresenham 累積誤差管理により、長時間動作でも誤差が積み上がらない設計のため）。実機で長時間走らせても聴感上のズレは出ていません。

ここは1つの記事にできるくらいネタが深いので、別記事で改めて書く予定です（記事末に予告します）。

もう一つ、ギター音の再生にも工夫があります。最初の実装では `noteValue`（0〜11）で音を再生していたので、指板のどこを押しても同じオクターブの E が鳴っていました。`Tuning.openMidiNotes` に MIDI 絶対値（E2=40, A2=45, ..., E4=64）を持たせて、`SoundManager` で押した位置の MIDI 番号を計算するように直しました。サンプル自体は MIDI 40〜83 の範囲を約3半音間隔で15個用意し、最も近いサンプルを選んで `SoundPool` の `rate` パラメータでピッチシフト（最大 ±1.5半音）するという構成です。

```kotlin
// SoundManager.playNote() の核心
fun playNote(midiNote: Int) {
    val sampleIdx = findNearestSample(midiNote)
    val semitoneDiff = midiNote - sampleMidiNotes[sampleIdx]
    // 12平均律: 1半音 = 周波数比 2^(1/12)
    val rate = 2.0.pow(semitoneDiff / 12.0).toFloat()
    soundPool.play(soundIds[sampleIdx], 1f, 1f, 1, 0, rate)
}
```

## ハマった話 5選

開発を通じて踏んだ罠を残しておきます。同じ場所で詰まる人の参考になれば。

### 1. onTap は数百ms 遅れる

`detectTapGestures(onTap = ...)` は、ロングプレス確認のため指を離してから数百ms遅れて発火します。普通の UI なら気にならないですが、トレーニングモードでメトロノームに合わせてタップする UI だと体感ラグが出ます。

トレーニングモードでは `onPress`（指が触れた瞬間に発火）に切り替えました。普通の指板画面は `onTap` のまま（長押しでルート変更機能を維持するため）。

### 2. 不可視ノートはタップが当たらない

Level 0-3（ヒントなしの音名タップ）では `fretNotes = emptyList()` で指板を「真っ白」に表示します。このとき従来の円形ヒット判定（`normalRadius * 1.5f`）では当たり判定が通りません。

`hitTestGrid()` を追加して、セル全体を当たり判定にしました。可視ノートが無くても、`FretboardCalculator` で位置からピッチクラスを合成して `onNoteClick` を呼びます。

### 3. noteValue(0-11) と MIDI絶対値は別管理

前述のとおり、音楽理論計算（インターバル判定）は `noteValue`（0-11）のまま、音再生用に `Tuning.openMidiNotes`（MIDI絶対値）を別経路で持つ設計にしました。混ぜると指板上のすべてのCが同じオクターブで鳴ります。

### 4. Roomマイグレーションを書き忘れるとアプリが起動時にクラッシュする

`UserSettings` や `TrainingProgress` に後からフィールドを追加する場合、必ず `Migration(from, to)` を書いて `addMigrations()` に登録します。`fallbackToDestructiveMigration()` はユーザーデータが消えるので避ける。自分用アプリでも、自分の練習履歴を消したくないので同じです。Claude Code は最初これを軽く扱ったので、`tasks/lessons.md` でルール化して以後の生成に効かせました。

### 5. ExperimentalFoundationApi の伝播

`@OptIn(ExperimentalFoundationApi::class)` を関数に付けても、内部のラムダや別関数に伝播しないケースがあります。ファイル先頭に `@file:OptIn(androidx.compose.foundation.ExperimentalFoundationApi::class)` を付けるのが確実です。

## 約2ヶ月使ってみての変化

開発を始めて約2ヶ月、ほぼ毎日 FretInterval を触っています。

ギターは、まだ初心者の域を出ません。けれど、移動時間にスマホで指板を眺めて「ルートからの距離」を考える癖がつきました。コードトーンを意識して弾けるようになり、適当に押さえても「いま4度の音を押した」と分かるくらいにはなっています。教則本だけ読んでいた頃より、確実に景色が変わっています。

副次的な発見もありました。Kotlin を1行も書いていないのに、Claude Code とのレビュー対話を通じて Compose の感覚や AudioTrack の挙動が頭に入ってきます。「コードを書く」より「コードを設計し、AI に書かせ、レビューする」という体験に近い。AI エンジニアとしての普段の仕事と地続きの感覚があります。

「自分用」と割り切ったことで、機能優先順位がぶれませんでした。誰かに見せるための飾りつけはなく、明日の練習で必要なものだけを足していく。Claude Code に投げる要望もシンプルに保てます。

## 今後

直近は v1.2 として、Level 2（スケール&アルペジオの8段階）とスコア画面の強化を予定しています。

将来的には、マイク入力でピッチ判定する「ギターモード」（TarsosDSP想定）も入れたいと思っています。画面タップではなく、実際にギターを弾いて答え合わせするモードです。

Google Play 配信の判断は v1.2 完成後に持ち越しています。先に整える必要はないと思っています。

### 関連記事の予告

この記事で触れたトピックのうち、特に深く書きたい4本を続編として準備中です。

- 「AudioTrack で ±0.01% の BPM 精度を出す ─ Bresenham 累積フレームカウント詳解」
- 「Jetpack Compose Canvas で動的な指板UIを描く ─ 座標系・ヒット判定・パフォーマンス」
- 「Kotlin 未経験で Android アプリを Claude Code に書かせるためのプロンプト設計」
- 「スマホ1台で完結する Claude Code × GitHub Actions の Android 個人開発フロー」

公開したらこの記事末にもリンクを追加します。

## まとめ

「Kotlin を一行も書けない AI エンジニアが、Claude Code で3日で Android アプリを作った」という越境ストーリーでした。

専門外領域で詰まったとき、自分のためだけのツールを **AI 支援で** 作るというのは、地味ですが効くアプローチだと思います。

- AI 支援を使うと、専門外言語でも「書く」コストが激減する
- 動線（毎日のどこで使うか）が明確だから、機能優先順位がぶれない
- 配信を前提としないと、無駄な抽象化を避けられる
- CI/CD と組み合わせると、PCがなくてもスマホ1台で開発が回る
- 開発と実践が両輪になり、フィードバックループが短い

「コードを書く」と「コードを設計してレビューする」は別のスキルだ、というのが2ヶ月使ってみての一番の発見でした。後者は AI エンジニアとしての普段の仕事と地続きで、専門外言語のアプリ開発がぐっと近い距離に来ています。

そして、**移動時間に学ぶアプリを、移動時間に作る**。AI とクラウドビルドの組み合わせでしか辿り着けないこの体験が、思いがけず一番面白い発見でした。

同じことを考えている方の参考になれば嬉しいです。
