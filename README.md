# ◆ 第 1 部 ◆ Siv3D をはじめよう
初めての Siv3D プログラムを動かそう。 Siv3D を使った開発をスムーズに始めるための手順を覚えます。

### Siv3D のインストール方法 (Windows)
Siv3D Web ページ https://github.com/Siv3D/Reference-JP/wiki/ダウンロードとインストール

# ◆ 第 2 部 ◆ 絵や音で遊ぼう
Siv3D を使うとどのようなことができるかを、音や画像で遊べるプログラムを通して体験します。

## 1. コンピュータ画伯
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



