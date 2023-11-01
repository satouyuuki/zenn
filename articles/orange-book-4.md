---
title: "30日後にosを完成させる。(4日目)"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os"]
published: true
---

## やったこと(p100)
ピクセルを描くこと
![](/images/pixel-write.png)

### 好きな位置に好きな色を描画する
ピクセル描画に必要な情報
```hpp: frame_buffer_config.hpp
struct FrameBufferConfig {
	uint8_t* frame_buffer; // フレームバッファ領域へのポインタ
	uint32_t pixels_per_scan_line; // フレームバッファの余白を含めた横要項のピクセル数
	uint32_t horizontal_resolution; // 水平の解像度
	uint32_t vertical_resolution; // 垂直の解像度
	enum PixelFormat pixel_format; // ピクセルのデータ形式
};
```
ブートローダー側はUEFIのGOPから取得した情報を上の構造体にコピーし、構造体へのポインタをKernelMain()の第一引数に渡す。

KernelMainでは画面全体を白に塗った後に(100, 100)の位置から横に200, 下に100だけ緑を描画する
```cpp: main.cpp
extern "C" void KernelMain(const FrameBufferConfig& frame_buffer_config) {
	for (int x = 0; x < frame_buffer_config.horizontal_resolution; ++x) {
		for (int y = 0; y < frame_buffer_config.vertical_resolution; ++y) {
			WritePixel(frame_buffer_config, x, y, {255, 255, 255});
		}
	}
	for (int x = 0; x < 200; ++x) {
		for (int y = 0; y < 100; ++y) {
			WritePixel(frame_buffer_config, 100 + x, 100 + y, {0, 255, 0});
		}
	}
	while (1) __asm__("hlt");
}
```

### 感想
ターミナルを分割したくterminatorというツールをインストールした。
これで開発がだいぶやりやすくなった。
https://cryborg.hatenablog.com/entry/2016/09/03/164940
ctrl+shift+o: ターミナルの分割
option+上下キー: ターミナルの移動

明日はvimでc++のコード補完をしたい。
あとvimのパッケージ管理とかを調べる。
