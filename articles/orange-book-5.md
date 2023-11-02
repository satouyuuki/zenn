---
title: "30日後にosを完成させる。(5日目)"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["os"]
published: true
---

### やったこと(p108)
c++の機能を使ったリファクタリング
昨日作ったWritePixelのピクセルデータ形式の判定は一度で良いはずが呼ばれるたびに計算される。
```cpp: main.cpp
/**
 * @retval 0 success
 * @retval !0 failed
 */
int WritePixel(const FrameBufferConfig& config,
		int x, int y, const PixelColor& c) {
	const int pixel_position = config.pixels_per_scan_line * y + x;
	if (config.pixel_format == kPixelRGBResv8BitPerColor) {
		uint8_t* p = &config.frame_buffer[4 * pixel_position];
		p[0] = c.r;
		p[1] = c.g;
		p[2] = c.b;
	} else if (config.pixel_format == kPixelBGRResv8BitPerColor) {
		uint8_t* p = &config.frame_buffer[4 * pixel_position];
                p[0] = c.b;
                p[1] = c.g;
                p[2] = c.r;
	} else {
		return -1;
	}
	return 0;
}
```

この関数内部でおこなってるピクセルのデータ形式判定を外だしする。
c++の言語機能に仮想関数という機能を使う
インターフェースと実装に分けるとこのようになる
```cpp: main.cpp
// インターフェース(抽象クラス)
class PixelWriter {
	public:
		PixelWriter(const FrameBufferConfig& config) : config_{config} {
		}
		virtual ~PixelWriter() = default;
		virtual void Write(int x, int y, const PixelColor& c) = 0;

	protected:
		uint8_t* PixelAt(int x, int y) {
			return config_.frame_buffer + 4 * (config_.pixels_per_scan_line * y + x);
		}

	private:
		const FrameBufferConfig& config_;
};

// 実装
class RGBResv8BitPerColorPixelWriter : public PixelWriter {
	public:
		using PixelWriter::PixelWriter;

		virtual void Write(int x, int y, const PixelColor& c) override {
			auto p = PixelAt(x, y);
			p[0] = c.r;
			p[1] = c.g;
			p[2] = c.b;
		}
};

class BGRResv8BitPerColorPixelWriter : public PixelWriter {
	public:
		using PixelWriter::PixelWriter;

		virtual void Write(int x, int y, const PixelColor& c) override {
			auto p = PixelAt(x, y);
			p[0] = c.b;
			p[1] = c.g;
			p[2] = c.r;
		}
};
```
このPixelWriterというのが仮想関数。ピクセルフォーマットに応じてクラスをnewすることができる。
ただosがない状態だと普通のnewができない。なぜならヒープ領域からメモリを確保するのはosの仕事だから。そのためosが無い状態では配置newという機能を使う。

```cpp: main.cpp
char pixel_writer_buf[sizeof(RGBResv8BitPerColorPixelWriter)];
PixelWriter* pixel_writer;

extern "C" void KernelMain(const FrameBufferConfig& frame_buffer_config) {
	switch (frame_buffer_config.pixel_format) {
		case kPixelRGBResv8BitPerColor:
			pixel_writer = new(pixel_writer_buf)
				RGBResv8BitPerColorPixelWriter{frame_buffer_config};
			break;
		case kPixelBGRResv8BitPerColor:
			pixel_writer = new(pixel_writer_buf)
				BGRResv8BitPerColorPixelWriter{frame_buffer_config};
			break;
	}
}
```
配置newは`new(インスタンスを配置するためのメモリ領域) Class名`という形で使う。


### EDK2のメモ
gBSの定義場所
https://github.com/tianocore/edk2/blob/master/MdePkg/Include/Library/UefiBootServicesTableLib.h

EFI_BOOT_SERVICESの定義場所(構造体)
https://github.com/tianocore/edk2/blob/fbbbd984998d83cf6b69e9291336aefbac23396c/MdePkg/Include/Uefi/UefiSpec.h#L1957

EFI_GET_MEMORY_MAPの定義場所
https://github.com/tianocore/edk2/blob/fbbbd984998d83cf6b69e9291336aefbac23396c/MdePkg/Include/Uefi/UefiSpec.h#L239C1-L247C5


引数が5個でEFI_STATUSを返す関数で(EFIAPI *EFI_GET_MEMORY_MAP)はEFIAPIへのポインタのメソッドということなのかな
```
typedef
EFI_STATUS
(EFIAPI *EFI_GET_MEMORY_MAP)(
  IN OUT UINTN                       *MemoryMapSize,
  OUT    EFI_MEMORY_DESCRIPTOR       *MemoryMap,
  OUT    UINTN                       *MapKey,
  OUT    UINTN                       *DescriptorSize,
  OUT    UINT32                      *DescriptorVersion
  );
```

ちなみにUINTNはunsigned intのnバイトという意味でosのアーキテクチャによってサイズが異なるみたい。X64だと64bit。
https://github.com/tianocore/edk2/blob/fbbbd984998d83cf6b69e9291336aefbac23396c/MdePkg/Include/X64/ProcessorBind.h#L211
