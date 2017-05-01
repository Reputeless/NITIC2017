Siv3D ハンズオンセミナー at NITIC 2017 公開資料

# ◆ 第 1 部 ◆ Siv3D をはじめよう
初めての Siv3D プログラムを動かそう。 Siv3D を使った開発をスムーズに始めるための手順を覚えます。

[Siv3D](http://siv3d.jp) は，音や画像を使ったプログラムや，マイク，Webカメラ，Kinectなど様々なデバイスを使ったアプリケーションを，シンプルなC++コードで開発できるフレームワークです．最新のC++規格を活用する，豊富なアプリケーション・プログラミング・インタフェース(API)により，お絵かきアプリは9行，Kinectを用いた人の姿勢のキャプチャは15行，ブロックくずしゲームは28行といったように，複雑なインタラクションを短いコードで記述できるのが特徴です．
現在Windows向けのソフトウェア開発キット（SDK）が無償で公開されているほか，対応プラットフォームをmacOSとLinuxにも広げた次世代版Siv3D，「OpenSiv3D」の開発も進められています．

### Siv3D のインストール方法 (Windows)
Siv3D Web ページ https://github.com/Siv3D/Reference-JP/wiki/ダウンロードとインストール

# ◆ 第 2 部 ◆ 絵や音で遊ぼう
Siv3D を使うとどのようなことができるかを、音や画像で遊べるプログラムを通して体験します。

今日は，Windows版のSiv3D SDKを用いて，音や画像で遊ぶ5つのインタラクティブなアプリケーションを実装し，その仕組みを解説します．C++プログラミングの文法や，Siv3Dの細かい機能についての説明は割愛していますが，コードは20～50行と短いので，プログラミングの経験がない方にも，音声処理や画像処理プログラミングの雰囲気や面白さが伝わると思います．

## 1. コンピュータ画伯

画像処理技術の進歩は目覚ましく，昨今は白黒画像をカラー画像に変換する技術や，線画に自動的に色を塗ってくれるツールが注目を集めている．私たちも何かすごい画像処理のプログラムを作ってみたいものだが，一朝一夕には難しい．そこでまずは，誰でも簡単にできるアルゴリズムから始めるとしよう．それは「適当に描く」ことだ．

#### 目標
コンピュータが，キャンバスのランダムな位置にランダムな色で丸を描き，絵を目標の画像に近づけていくプログラムを作ってみよう．丸を描く直前に絵を保存しておき，もし丸を描いて目標との類似度が離れてしまったら，保存していた絵に差し戻すことで，描けば描くほど目標に近づいていくようにしよう．

#### 実装
2つの画像がどれだけ類似しているかを，各ピクセルのRGB成分の差の絶対値を合計することで求めるDiff関数を定義する．この値が小さいほど，2つの画像は類似していることを意味する―①．Main関数ではまず，コンピュータに描かせる目標の画像をユーザに選択させて開く．画像は画面内に収まるよう縮小する―②．続いて，同じ大きさの白い画像を用意し―③，その時点での目標との差を計測する―④．メインループ内では，最初に現在の画像を保存しておき―⑤，ランダムな位置に―⑥ランダムな色－⑦と大きさ―⑧で円を描き足す―⑨．Diff関数で目標との類似度を調べ，差が縮まればそれを採用し―⑩，それ以外の場合は⑤で保存した画像に差し戻す―⑪．これをメインループごとに100回繰り返し，最後の時点での画像を画面に描画する―⑫．これで，真っ白な画像から始まり，だんだん目標の絵に近づいていくプログラムが完成する．

```cpp
# include <Siv3D.hpp>

// ① 2つの画像の差分を計算
double Diff(const Image& a, const Image& b) {
	double d = 0.0;
	for (auto p : step(a.size))
	{
		d += Abs(int(a[p].r) - int(b[p].r));
		d += Abs(int(a[p].g) - int(b[p].g));
		d += Abs(int(a[p].b) - int(b[p].b));
	}
	return d;
}

void Main() {
	// ② 目標とする画像を開く
	const Image target = Dialog::OpenImage()
		.fit(Window::Size());
	// ③ 同じ大きさの白い画像を用意
	Image image(target.size, Palette::White);
	Image old = image;
	DynamicTexture texture(old);
	double d1 = Diff(target, image); // ④

	while (System::Update()) {
		for (int i = 0; i < 100; ++i) {
			old = image; // ⑤

			// ⑥ 画像内のランダムな位置
			Point pos = RandomPoint(
				image.width, image.height);
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
			if (d2 < d1) {
				d1 = d2;
			}
			else {
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


## 2. あなたの声はどんな形




```cpp
# include <Siv3D.hpp>
void Main() {
	// ① マイク録音開始
	Recorder mic;
	mic.open(0, 5s, RecordingFormat::S44100, true);
	mic.start();
	// ② 加算ブレンドを有効に
	Graphics2D::SetBlendState(BlendState::Additive);
	while (System::Update()) {
		// ③ FFT で周波数成分を解析
		const auto fft = FFT::Analyze(mic);
		for (int i = 0; i < fft.length(); ++i) {
			// ④ 音階
			double s = Log2(
				(fft.resolution() * (i + 1)) / 27.5);
			// ⑤ パワー
			double p = fft.buffer[i];
			// ⑥ 円の中心位置
			const Vec2 pos =
				Window::Center() +
				Circular(Pow(p, 0.5) * 600, s * TwoPi);
			// ⑦ 円の色
			const Color c = HSV(s * 360)
				.toColorF(0.05 * s);
			// ⑧ 円を描く
			Circle(pos, 15 - s).draw(c);
		}
	}
}
```

## 3. 幸せになれる画像ビューア

```cpp
# include <Siv3D.hpp>
// ① コメントの文章と位置
struct Comment {
	String text;
	Vec2 pos;
};
void Main() {
	// ② 画像を選択する
	Texture photo(Dialog::OpenImage()
		.fit(Window::Size()));
	// ③ 事前に用意したコメントを読み込む
	TextReader reader(L"comments.txt");
	Array<Comment> comments;
	Comment c;
	while (reader.readLine(c.text)) {
		c.pos.x = Random(1000, 2000); // ④
		c.pos.y = Random(0, 420);
		comments.push_back(c);
	}
	// ⑤ コメント用のフォント
	Font font(30, Typeface::Bold, FontStyle::Outline);
	font.changeOutlineStyle(TextOutlineStyle(
		Palette::Gray, Palette::White, 1));
	while (System::Update()) {
		photo.draw(); // ⑥
		for (auto& c : comments) {
			font(c.text).draw(c.pos); // ⑦
									  // ⑧ コメントを左に流す
			c.pos.x -= 4 + c.text.length * 0.2;
			// ⑨ 画面外に出たらまた右へ
			if (c.pos.x < -500) {
				c.pos.x = Random(1000, 2000);
				c.pos.y = Random(0, 420);
			}
		}
	}
}
```

## 4. めちゃくちゃな音楽プレイヤ

```cpp
# include <Siv3D.hpp>
void Main() {
	// ① 表示用のフォントを用意
	Font font(20);
	// ② 音楽ファイルを開く
	Sound sound = Dialog::OpenSound();
	// ③ 音楽を再生
	sound.play();
	while (System::Update()) {
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

## 5. 数式の大地を探検する

```cpp
# include <Siv3D.hpp>
void Main() {
	// ① 背景を明るい色に
	Graphics::SetBackground(Color(120, 180, 160));
	// ② メッシュを用意
	MeshData meshData = MeshData::Grid(25, 160);
	DynamicMesh mesh(meshData);
	// ③ 数式を入力するテキストエリアを用意
	GUI gui(GUIStyle::Default);
	gui.addln(L"exp", GUITextArea::Create(2, 30));
	while (System::Update()) {
		if (!gui.textArea(L"exp").active) {
			Graphics3D::FreeCamera(); // ④
		}
		// ⑤ 数式が変更されたら
		if (gui.textArea(L"exp").hasChanged) {
			if (const ParsedExpression
				exp{ gui.textArea(L"exp").text }) {
				gui.textArea(L"exp").style.color
					= Palette::Black; // ⑥
				// ⑦ 数式に基づきメッシュの座標を計算
				for (auto& v : meshData.vertices) {
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
			else {
				gui.textArea(L"exp").style.color
					= Palette::Red; // ⑩
			}
		}
		// ⑪ メッシュを描画
		mesh.draw().drawShadow();
	}
}
```

# ◆ 第 3 部 ◆ Siv3D を学ぼう
Siv3D 公式 Web サイトのチュートリアルに沿って、Siv3D の基本機能を学びます。



