Siv3D ハンズオンセミナー at NITIC 2017 公開資料

# ◆ 第 1 部 ◆ Siv3D をはじめよう
初めての Siv3D プログラムを動かそう。 Siv3D を使った開発をスムーズに始めるための手順を覚えます。

[Siv3D](http://siv3d.jp) は，音や画像を使ったプログラムや，マイク，Web カメラ，Kinect など様々なデバイスを使ったアプリケーションを，シンプルな C++ コードで開発できるフレームワークです．最新の C++ 規格を活用する，豊富なアプリケーション・プログラミング・インタフェース (API) により，お絵かきアプリは 9 行，Kinect を用いた人の姿勢のキャプチャは 15 行，ブロックくずしゲームは 28 行といったように，複雑なインタラクションを短いコードで記述できるのが特徴です．
現在 Windows 向けのソフトウェア開発キット（SDK）が無償で公開されているほか，対応プラットフォームを macOS と Linux にも広げた次世代版 Siv3D，「OpenSiv3D」の開発も進められています．

### 1. Siv3D のインストール方法 (Windows)
Siv3D Web ページ https://github.com/Siv3D/Reference-JP/wiki/ダウンロードとインストール

### 2. プロジェクトの作り方

### 3. ビルドと実行

-----

# ◆ 第 2 部 ◆ 絵や音で遊ぼう
Siv3D を使うとどのようなことができるかを、音や画像で遊べるプログラムを通して体験します。

今日は，Windows 版の Siv3D SDK を用いて，音や画像で遊ぶ 5 つのインタラクティブなアプリケーションを実装し，その仕組みを解説します．C++ プログラミングの文法や，Siv3D の細かい機能についての説明は割愛していますが，コードは 20～50 行と短いので，プログラミングの経験がない方にも，音声処理や画像処理プログラミングの雰囲気や面白さが伝わると思います．

## 1. コンピュータ画伯

<img src="https://github.com/Reputeless/NITIC2017/blob/master/2.png" width="320" height="240"> <img src="https://github.com/Reputeless/NITIC2017/blob/master/3.png" width="320" height="240">  

画像処理技術の進歩は目覚ましく，昨今は白黒画像をカラー画像に変換する技術や，線画に自動的に色を塗ってくれるツールが注目を集めている．私たちも何かすごい画像処理のプログラムを作ってみたいものだが，一朝一夕には難しい．そこでまずは，誰でも簡単にできるアルゴリズムから始めるとしよう．それは「適当に描く」ことだ．

#### 目標
コンピュータが，キャンバスのランダムな位置にランダムな色で丸を描き，絵を目標の画像に近づけていくプログラムを作ってみよう．丸を描く直前に絵を保存しておき，もし丸を描いて目標との類似度が離れてしまったら，保存していた絵に差し戻すことで，描けば描くほど目標に近づいていくようにしよう．

#### 実装
2つの画像がどれだけ類似しているかを，各ピクセルのRGB成分の差の絶対値を合計することで求めるDiff関数を定義する．この値が小さいほど，2つの画像は類似していることを意味する―①．Main関数ではまず，コンピュータに描かせる目標の画像をユーザに選択させて開く．画像は画面内に収まるよう縮小する―②．続いて，同じ大きさの白い画像を用意し―③，その時点での目標との差を計測する―④．メインループ内では，最初に現在の画像を保存しておき―⑤，ランダムな位置に―⑥ランダムな色－⑦と大きさ―⑧で円を描き足す―⑨．Diff関数で目標との類似度を調べ，差が縮まればそれを採用し―⑩，それ以外の場合は⑤で保存した画像に差し戻す―⑪．これをメインループごとに100回繰り返し，最後の時点での画像を画面に描画する―⑫．これで，真っ白な画像から始まり，だんだん目標の絵に近づいていくプログラムが完成する．

```cpp
# include <Siv3D.hpp>

// ① 2つの画像の差分を計算
double Diff(const Image& a, const Image& b)
{
	double d = 0.0;

	for (auto p : step(a.size))
	{
		d += Abs(int(a[p].r) - int(b[p].r));
		d += Abs(int(a[p].g) - int(b[p].g));
		d += Abs(int(a[p].b) - int(b[p].b));
	}

	return d;
}

void Main()
{
	// ② 目標とする画像を開く
	const Image target = Dialog::OpenImage().fit(Window::Size());

	// ③ 同じ大きさの白い画像を用意
	Image image(target.size, Palette::White);
	Image old = image;
	DynamicTexture texture(old);

	double d1 = Diff(target, image); // ④

	while (System::Update())
	{
		for (int i = 0; i < 100; ++i)
		{
			old = image; // ⑤

			// ⑥ 画像内のランダムな位置
			Point pos = RandomPoint(image.width, image.height);

			// ⑦ ランダムな色
			ColorF color;
			color.r = Random();
			color.g = Random();
			color.b = Random();
			color.a = Random();

			// ⑧ ランダムな大きさ
			int size = Random(1, 10);

			// ⑨ 円を描いてみる
			Circle(pos, size).write(image, color);

			// ⑩ 目標に近づけば採用
			double d2 = Diff(target, image);
		
			if (d2 < d1)
			{
				d1 = d2;
			}
			else
			{
				image = old; // ⑪
			}
		}

		// ⑫ テクスチャを更新して描画
		texture.fill(image);
		texture.draw();
	}
}
```

#### さらに発展
丸の代わりに直線や曲線を描き込んで絵のタッチを変えてみたり，描き込みの位置を前回描き込んだ位置と近づけることで，人間らしく描かせたりするような工夫をするのも面白いだろう．

-----

## 2. あなたの声はどんな形
<img src="https://github.com/Reputeless/NITIC2017/blob/master/4.png" width="320" height="240">

共感覚と呼ばれる特殊な知覚を持つ人のなかには，音に色がついて見える人がいると言われている．ちょっと素敵な能力のような気がしてうらやましいが，プログラミングができれば，私たちも似たような感覚を体験できる．
私たちが耳にする声とは，声帯や口の開き方によって特徴づけられる，空気中の圧力変動の波である．この波形信号をマイクで記録し，高速フーリエ変換（FFT）により，音声にどのような周波数成分が含まれているかを計算で求め，その結果を色や形としてグラフィカルに表現すれば，あなたにも声が見えるようになる．

#### 目標
マイクで自分の声を録音し，そこに含まれる周波数成分をリアルタイムで可視化するアプリケーションを作ってみよう．周波数をドレミの音階に対応付けて円周上に配置し，周波数成分の大きさに応じて円を描くことで，画面上に声の形を作り出そう．

#### 実装
まず，マイクをセットアップして声の録音を開始する．録音用のバッファは毎秒44100サンプルで5秒分，バッファがいっぱいになるとバッファの先頭から上書きする―①．カラフルな円を重ねて描くときに，光が飽和するような美しい表現を得るために，色のブレンドモードを加算ブレンド（Additive）にしておこう―②．メインループでは，マイクで録音された直近の音声に，どのような周波数成分が分布しているのかをFFTで解析する―③．結果の配列から，それぞれの音階―④におけるパワー―⑤を求める．ここでの音階sは，ピアノの一番低いラの音を0として，1オクターブ上がるごとに1大きくなる値である．円の位置は，その音階のパワーが大きいほど中心から離れ，その方向は音階によって異なる．ラの音であれば12時の方向に，半音高いラのシャープであれば1時の方向に伸び，1オクターブで1周する―⑥．円の色も音階を色相環に対応させる．ラの音であれば赤色，ラのシャープであればオレンジ色と，こちらも1オクターブで1周させる―⑦．計算した位置と色に基づいてそれぞれの円を描くと声の形が現れる―⑧．

```cpp
# include <Siv3D.hpp>

void Main()
{
	// ① マイク録音開始
	Recorder mic;
	mic.open(0, 5s, RecordingFormat::S44100, true);
	mic.start();

	// ② 加算ブレンドを有効に
	Graphics2D::SetBlendState(BlendState::Additive);

	while (System::Update())
	{
		// ③ FFT で周波数成分を解析
		const auto fft = FFT::Analyze(mic);

		for (int i = 0; i < fft.length(); ++i)
		{
			// ④ 音階
			double s = Log2((fft.resolution() * (i + 1)) / 27.5);

			// ⑤ パワー
			double p = fft.buffer[i];

			// ⑥ 円の中心位置
			const Vec2 pos = Window::Center() + Circular(Pow(p, 0.5) * 600, s * TwoPi);

			// ⑦ 円の色
			const Color c = HSV(s * 360).toColorF(0.05 * s);

			// ⑧ 円を描く
			Circle(pos, 15 - s).draw(c);
		}
	}
}
```

#### さらに発展
声以外にも，口笛やいろいろな楽器を鳴らして，どのような形が現れるか調べてみよう．音の成分の表現方法を，棒グラフやもっと複雑なエフェクトにしてみるなど，ビジュアルを工夫するのも良いだろう．

-----

## 3. 幸せになれる画像ビューア
<img src="https://github.com/Reputeless/NITIC2017/blob/master/5.png" width="443" height="493">
旅行先の素敵な景色，おいしそうなランチ，可愛い我が子の成長記録．私たちがSNSに写真や動画をアップロードするのは，他人からの「いいね」によって，自らの幸せを再確認したいからなのかもしれない．一般的に，ある個人のアカウントを注目している「フォロワー」の人数が多いほど，投稿にはたくさんの反響が寄せられる．数万人単位のフォロワーを抱える有名人の投稿には，ステージに浴びせられる歓声のように，賞賛や羨望のコメントが降り注ぐ．
さて，私たちのような平凡な人間でも，プログラミングができると，そんな人気者の気分を少しだけ味わえるというのはご存じだろうか．用意するのは，あなたが今日撮った写真と，降り注がせたいコメントを列挙したテキストファイルだけである．

#### 目標
自分の撮った写真を開くと，たくさんのコメントが画面を埋め尽くす画像ビューアを作ってみよう．コメントはテキストファイルに改行で区切って記述する．20～30パターンのコメントを用意すれば，にぎやかな画面になるだろう．

#### 実装
まず，画面に流れるコメントの文章（String型）と位置（Vec2型）を保持するComment型を定義しておこう―①．Main関数の最初で，ユーザにダイアログから写真を選択させる．画面内に収まるようにサイズを調整しておく―②．次に，TextReaderクラスを使って，用意しておいたテキストファイルから1行ずつコメントを読み込む―③．コメントを表示する初期位置はランダムにする―④．コメント用のフォントもあらかじめ用意しておく．文字にアウトラインを付けるとよく映える―⑤．メインループ内では，読み込んだ写真を背景に描画し―⑥，その上にコメントを表示する―⑦．コメントが画面の右から左へ流れるように座標を制御すると―⑧，どこかで見たことのあるインタフェースになる．画面外に流れたコメントは，座標をリセットして再利用すれば，いつまでもコメントを流せる―⑨．

```cpp
# include <Siv3D.hpp>

// ① コメントの文章と位置
struct Comment
{
	String text;
	Vec2 pos;
};

void Main()
{
	// ② 画像を選択する
	Texture photo(Dialog::OpenImage().fit(Window::Size()));

	// ③ 事前に用意したコメントを読み込む
	TextReader reader(L"comments.txt");
	Array<Comment> comments;
	Comment c;
	while (reader.readLine(c.text))
	{
		c.pos.x = Random(1000, 2000); // ④
		c.pos.y = Random(0, 420);
		comments.push_back(c);
	}

	// ⑤ コメント用のフォント
	Font font(30, Typeface::Bold, FontStyle::Outline);
	font.changeOutlineStyle(TextOutlineStyle(Palette::Gray, Palette::White, 1));

	while (System::Update())
	{
		photo.draw(); // ⑥

		for (auto& c : comments)
		{
			font(c.text).draw(c.pos); // ⑦
			
			// ⑧ コメントを左に流す
			c.pos.x -= 4 + c.text.length * 0.2;

			// ⑨ 画面外に出たらまた右へ
			if (c.pos.x < -500)
			{
				c.pos.x = Random(1000, 2000);
				c.pos.y = Random(0, 420);
			}
		}
	}
}
```

-----

## 4. めちゃくちゃな音楽プレイヤー
<img src="https://github.com/Reputeless/NITIC2017/blob/master/6.png" width="320" height="240">
一青窈の「もらい泣き」の音の高さを下げて再生すると，平井堅が歌っているように聞こえるそうだ．私たちは撮った写真の見栄えを良くしたり，加工をしたりすることはよくあるが，音楽に関してはなかなか遊ぶ機会が少ないのではないだろうか．パソコンに保存されているお気に入りの音楽を再生する，ちょっと不思議な音楽プレイヤーを作ってみよう．

#### 目標
再生している音楽のテンポ（1分間に何拍刻むかの速さ）とピッチ（音の高さ）をリアルタイムに変更できる音楽プレイヤーを作ろう (脚注1)．画面に基準の線を表示し，マウスカーソルを右に移動させると速いテンポで，左に移動させると遅いテンポで，上に移動させると高いピッチで，下に移動させると低いピッチで再生されるようにしよう．

(脚注1) いわゆる「早送り再生」では，テンポとピッチが同時に速く，高くなるが，今回のプログラムでは，それぞれの要素を独立して操作する．例えば，ゆっくり再生しても音の高さは変わらないといったことが可能である．

#### 実装
テンポとピッチを表示するためのフォントを用意する―①．続いてユーザにダイアログから音楽ファイルを選択させ―②，それを再生する―③．メインループ内では，ユーザがテンポやピッチを制御する基準となる線―④と円―⑤を描く．マウスカーソルが画面の中心よりも右にあればあるほどテンポを速くし―⑥，マウスカーソルが画面の中心よりも上にあればあるほどピッチが高くなるように値を計算する―⑦．計算した結果は，再生中の音楽に設定することで反映される―⑧．最後にこれらの値を画面に表示して完成である－⑨．

```cpp
# include <Siv3D.hpp>

void Main()
{
	// ① 表示用のフォントを用意
	Font font(20);

	// ② 音楽ファイルを開く
	Sound sound = Dialog::OpenSound();

	// ③ 音楽を再生
	sound.play();

	while (System::Update())
	{
		// ④ 線を描く
		Line(0, 240, 640, 240).draw(4);
		Line(320, 480, 320, 0).draw(4);

		// ⑤ カーソルの位置に円を描く
		Point pos = Mouse::Pos();
		Circle(pos, 20).draw(Palette::Orange);

		// ⑥ テンポを計算
		double tempo = Exp2((pos.x - 320) / 240.0);

		// ⑦ ピッチを計算
		double pitch = -(pos.y - 240) / 60.0;

		// ⑧ 音楽にテンポとピッチを適用
		sound.changeTempo(tempo);
		sound.changePitchSemitones(pitch);

		// ⑨ 現在のテンポとピッチを表示
		font(L"tempo: ", tempo).draw(20, 20);
		font(L"pitch: ", pitch).draw(20, 60);
	}
}
```

-----

## 5. 数式の大地を探検する
<img src="https://github.com/Reputeless/NITIC2017/blob/master/7.png" width="320" height="240">
私たちは地図アプリやゲームで3D空間を探検することはよくあるが，実際にこうした3Dのデジタルデータを作る機会は少ない．近頃は3Dプリンタが手の届く価格で利用できるようになったのだから，何か3Dのものを作ってみようではないか．複雑なモデリングソフトは使わず，数学の知識だけで3Dマップを生成できるアプリケーションを作ってみよう．

#### 目標
テキストエリアにx, yの関数の数式を入力すると，そのグラフを3D空間上に描いてくれるプログラムを作ろう．キーボード入力で，3Dグラフの中を探検できるようにもしてみよう．たった数十文字の数式から生み出されるバリエーション豊かな地形やダンジョンが，あなたの冒険を待っている．

#### 実装
グラフが見やすいように画面の背景を明るい色に設定する―①．次に，グリッド状の頂点配列で構成される3Dメッシュデータを用意する―②．ユーザが数式を入力するための，2行×30文字分のテキストエリアを，グラフィカルユーザインタフェース（GUI）クラスを使って用意する―③．メインループ内では，ユーザが数式を入力中でなければ，アローキーやW/A/S/Dなどのキーボード入力で3D空間上の視点を操作できるようにする関数を呼ぶ―④．もし，テキストエリア内の数式が変更されたら―⑤，数式をパースし，有効な式であれば数式の文字を黒くする―⑥．そして，数式を用いてメッシュ状の頂点の配列の座標を再計算する．Siv3Dの3D空間ではy軸が上方向なので，各頂点のy座標を，x 座標とz座標から求める―⑦．配列の座標を更新したら，陰影の計算のために必要な法線情報を再構築し―⑧，メッシュデータを新しい配列で更新する―⑨．もし数式のパース時にエラーがあれば，数式の文字を赤くして，ユーザに伝える―⑩．メッシュとその影を描画すれば―⑪，画面に3Dグラフの地形が現れ，その中を探検できる．

> :warning: Release ビルドで、「デバッグなしで実行」をしないとちょっと重いです。

```cpp
# include <Siv3D.hpp>

void Main()
{
	// ① 背景を明るい色に
	Graphics::SetBackground(Color(120, 180, 160));

	// ② メッシュを用意
	MeshData meshData = MeshData::Grid(25, 160);
	DynamicMesh mesh(meshData);

	// ③ 数式を入力するテキストエリアを用意
	GUI gui(GUIStyle::Default);
	gui.addln(L"exp", GUITextArea::Create(2, 30));

	while (System::Update())
	{
		if (!gui.textArea(L"exp").active)
		{
			Graphics3D::FreeCamera(); // ④
		}

		// ⑤ 数式が変更されたら
		if (gui.textArea(L"exp").hasChanged)
		{
			if (const ParsedExpression exp{ gui.textArea(L"exp").text })
			{
				gui.textArea(L"exp").style.color = Palette::Black; // ⑥
				
				// ⑦ 数式に基づきメッシュの座標を計算
				for (auto& v : meshData.vertices)
				{
					v.position.y = exp.evaluateOpt({
						{ L"x", v.position.x },
						{ L"y", v.position.z } })
						.value_or(0);
				}

				// ⑧ 法線を更新
				meshData.computeNormals();

				// ⑨ メッシュを更新
				mesh.fillVertices(meshData.vertices);
			}
			else
			{
				gui.textArea(L"exp").style.color = Palette::Red; // ⑩
			}
		}

		// ⑪ メッシュを描画
		mesh.draw().drawShadow();
	}
}
```

#### さらに発展
`draw()`の引数に色や画像を渡せば，地形に色や模様をつけられる．気に入った形のメッシュを3Dプリンタで扱えるファイル形式で出力し，実体化したものを棚に並べて鑑賞するのも楽しいだろう．

## その他のおすすめサンプル

- 対称定規 https://github.com/Siv3D/Reference-JP/wiki/%E5%AF%BE%E7%A7%B0%E5%AE%9A%E8%A6%8F
- MIDIビジュアライザー https://github.com/Siv3D/Reference-JP/wiki/MIDI%E3%83%93%E3%82%B8%E3%83%A5%E3%82%A2%E3%83%A9%E3%82%A4%E3%82%B6%E3%83%BC
- 顔検出 https://github.com/Siv3D/Reference-JP/wiki/%E9%A1%94%E6%A4%9C%E5%87%BA
- Image to PhysicsBody https://github.com/Siv3D/Reference-JP/wiki/Image-to-PhysicsBody

# ◆ 第 3 部 ◆ Siv3D を学ぼう
Siv3D 公式 Web サイトのチュートリアルに沿って、Siv3D の基本機能を学びます。  
- [Siv3D チュートリアル 1](https://github.com/Siv3D/Reference-JP/wiki/Siv3D%E3%81%AE%E5%9F%BA%E6%9C%AC)

