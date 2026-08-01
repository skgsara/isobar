# KG-FAX GUI layout reference

Distilled from the VCL form resources (binary DFM) of the
original program. Facts only: control names, classes,
positions, sizes, and captions. All coordinates are VCL
design-time pixels (PixelsPerInch = 96), relative to each
control's parent. Nested controls are shown with their
parent in parentheses.

Extracted with `tools/extract_dfm.py`. (The full verbose dump is kept
locally as private working material and is not published.)

## Form1 (TForm1, TFORM1) -- client area 888x551

Window caption: `KG-FAX`

The **English** column is the caption actually used in the port (UI
language is English — DEVIATIONS.md #5). The Japanese column remains
the reference to the original. Empty = no caption. All forms are
implemented; English captions are shown per-form below.

| Control | Class | Left | Top | Width | Height | Caption | English |
|---|---|---|---|---|---|---|---|
| BitBtn1 | TBitBtn | 776 | 520 | 105 | 25 | 終　　　　了 | Quit |
| Button1 | TButton | 392 | 520 | 90 | 25 | 画面消去 | Clear |
| ScrollBox1 | TScrollBox | 5 | 8 | 764 | 504 |  |  |
| Image1 (ScrollBox1) | TImage | 0 | 0 | 760 | 500 |  |  |
| Panel1 | TPanel | 776 | 456 | 105 | 57 |  |  |
| Label2 (Panel1) | TLabel | 16 | 6 | 48 | 12 | 同期検出 | Sync det |
| Label4 (Panel1) | TLabel | 16 | 22 | 48 | 12 | 同期補足 | Sync corr |
| Label15 (Panel1) | TLabel | 16 | 38 | 48 | 12 | 制御信号 | Control |
| Panel2 (Panel1) | TPanel | 72 | 8 | 17 | 9 |  |  |
| Panel3 (Panel1) | TPanel | 72 | 24 | 17 | 9 |  |  |
| Panel8 (Panel1) | TPanel | 72 | 40 | 17 | 9 |  |  |
| Panel7 | TPanel | 776 | 8 | 102 | 108 |  |  |
| Image3 (Panel7) | TImage | 1 | 1 | 100 | 106 |  |  |
| Button4 | TButton | 8 | 520 | 90 | 25 | データ読込 | Load data |
| Button5 | TButton | 104 | 520 | 90 | 25 | データ保存 | Save data |
| Button6 | TButton | 200 | 520 | 90 | 25 | 画像保存 | Save image |
| Button7 | TButton | 296 | 520 | 90 | 25 | 印刷 | Print |
| GroupBox1 | TGroupBox | 775 | 321 | 105 | 129 | パラメータ | Parameters |
| Label1 (GroupBox1) | TLabel | 11 | 53 | 60 | 12 | 同期信号長 | Sync len |
| Label3 (GroupBox1) | TLabel | 11 | 16 | 48 | 12 | 回転速度 | Speed |
| SpeedButton2 (GroupBox1) | TSpeedButton | 8 | 94 | 25 | 25 |  |  |
| ComboBox4 (GroupBox1) | TComboBox | 24 | 68 | 73 | 20 |  |  |
| ComboBox5 (GroupBox1) | TComboBox | 24 | 30 | 73 | 20 |  |  |
| Button9 (GroupBox1) | TButton | 39 | 94 | 60 | 25 | 詳細設定 | Details… |
| GroupBox2 | TGroupBox | 775 | 122 | 105 | 193 | 受信操作 | Reception |
| Button2 (GroupBox2) | TSpeedButton | 8 | 24 | 89 | 33 | 掃　　 引 | Scan |
| Button3 (GroupBox2) | TSpeedButton | 8 | 64 | 89 | 33 | 自動制御 | Auto ctl |
| SpeedButton1 (GroupBox2) | TSpeedButton | 8 | 144 | 89 | 33 | 同　　　期 | Sync |
| SpeedButton3 (GroupBox2) | TSpeedButton | 8 | 104 | 89 | 33 | 自動保存 | Auto save |
| Button10 | TButton | 680 | 520 | 90 | 25 | 色処理 | Color |
| Button8 | TButton | 488 | 520 | 90 | 25 | 縦表示 | Vertical |
| Button11 | TButton | 584 | 520 | 90 | 25 | XY反転 | XY flip |
| PrintDialog1 | TPrintDialog | 856 | 440 | — | — |  |  |

Live reception additions (drawn, not DFM controls; docs/01 sec. 4):
the preview shows incoming lines sideways — one 1-px column per 3
lines (3-line average), each column 500 px tall (3:1 in both axes,
aspect preserved), drawn flipped so the line start (sync edge) is at
the bottom — appended left to right, with cyan HUD text `掃引中` →
English `Scanning` and `Line# n`, plus an L-shaped cursor at the
reception frontier. The scope (Image3) shows spectrum AND waterfall
simultaneously: spectrum in the top 30 px, waterfall scrolling in the
bottom 76 rows; clicking the scope pauses/resumes it.

## Form10 (TForm10, TFORM10) -- client area 195x43

Window caption: `録音デバイス変更`

| Control | Class | Left | Top | Width | Height | Caption | English |
|---|---|---|---|---|---|---|---|
| Label1 | TLabel | 56 | 16 | 78 | 12 | 録音再設定中... | Reconfiguring recording... |

Ported as `gui/noticedialog.*`: a borderless popup shown while the audio
input device is being switched mid-reception (English caption —
DEVIATIONS #5). Dev flag `--test-notice`.

## Form2 (TForm2, TFORM2) -- client area 328x48

Window caption: `処理中...`

| Control | Class | Left | Top | Width | Height | Caption | English |
|---|---|---|---|---|---|---|---|
| Panel1 | TPanel | 0 | 0 | 329 | 49 |  |  |
| Label1 (Panel1) | TLabel | 8 | 8 | 313 | 17 | 処理中... | Processing... |
| ProgressBar1 (Panel1) | TProgressBar | 8 | 24 | 313 | 17 |  |  |

Ported as `gui/progressdialog.*` (RAII `ProgressScope`): shown around the
save-image, print, and auto-save BMP renders (English captions —
DEVIATIONS #5). On modern hardware these are near-instant, so the bar
only flashes briefly. Dev flag `--test-progress`.

## Form3 (TForm3, TFORM3) -- client area 409x224

Window caption: `Form3`

| Control | Class | Left | Top | Width | Height | Caption |
|---|---|---|---|---|---|---|
| Label1 | TLabel | 8 | 198 | 53 | 12 | ファイル名 |
| DriveComboBox1 | TDriveComboBox | 8 | 8 | 193 | 18 |  |
| DirectoryListBox1 | TDirectoryListBox | 8 | 32 | 193 | 129 |  |
| FileListBox1 | TFileListBox | 207 | 8 | 193 | 177 |  |
| FilterComboBox1 | TFilterComboBox | 8 | 168 | 193 | 20 |  |
| Button1 | TButton | 247 | 191 | 73 | 25 | OK |
| Button2 | TButton | 328 | 191 | 73 | 25 | キャンセル |
| Edit1 | TEdit | 64 | 195 | 177 | 20 |  |

## Form4 (TForm4, TFORM4) -- client area 320x156

Window caption: `詳細設定` -- English: `Details`

Port notes: the original applies on close and has no OK/Cancel; the
port's modal dialog adds them below the groups, so its window is
320x192. The Panel+UpDown pairs become spinners. The Info group text
is neutral port info, NOT the original's version string/branding
(legal policy). Control-to-setting mapping (by row, top to bottom):
ComboBox1 = SyncThre (value 20*index+10), then spinners LReSycn,
RReSycn, SyncWidth (n where width = 10*n ms), Sync2Thre, DetTime.

| Control | Class | Left | Top | Width | Height | Caption | English |
|---|---|---|---|---|---|---|---|
| GroupBox3 | TGroupBox | 8 | 8 | 193 | 145 | 同期処理 | Sync |
| Label5 (GroupBox3) | TLabel | 8 | 64 | 84 | 12 | 完全補足解除値 | Full release |
| Label6 (GroupBox3) | TLabel | 8 | 20 | 96 | 12 | 同期信号検出判定 | Sync detect |
| Label4 (GroupBox3) | TLabel | 8 | 83 | 72 | 12 | 同期補足範囲 | Corr range |
| Label1 (GroupBox3) | TLabel | 8 | 45 | 90 | 12 | 補足固定・解除値 | Lock/release |
| Label3 (GroupBox3) | TLabel | 8 | 102 | 72 | 12 | 同期信号閾値 | Sync thresh |
| Label7 (GroupBox3) | TLabel | 8 | 121 | 96 | 12 | 制御信号検出時間 | Ctl det time |
| Panel4 (GroupBox3) | TPanel | 120 | 59 | 41 | 17 | Panel4 | (spinner) |
| UpDown4 (GroupBox3) | TUpDown | 160 | 59 | 17 | 17 |  |  |
| ComboBox1 (GroupBox3) | TComboBox | 120 | 16 | 57 | 20 |  |  |
| Panel5 (GroupBox3) | TPanel | 120 | 78 | 41 | 17 | Panel5 | (spinner) |
| UpDown5 (GroupBox3) | TUpDown | 160 | 78 | 17 | 17 |  |  |
| Panel1 (GroupBox3) | TPanel | 120 | 40 | 41 | 17 | Panel1 | (spinner) |
| UpDown1 (GroupBox3) | TUpDown | 160 | 40 | 17 | 17 |  |  |
| Panel2 (GroupBox3) | TPanel | 120 | 97 | 41 | 17 | Panel2 | (spinner) |
| Panel3 (GroupBox3) | TPanel | 120 | 116 | 41 | 17 | Panel3 | (spinner) |
| UpDown2 (GroupBox3) | TUpDown | 160 | 97 | 17 | 17 |  |  |
| UpDown3 (GroupBox3) | TUpDown | 160 | 116 | 17 | 17 |  |  |
| GroupBox6 | TGroupBox | 208 | 8 | 105 | 145 | ソフト情報 | Info |
| Image1 (GroupBox6) | TImage | 36 | 23 | 33 | 33 |  | (portrait icon) |
| Label10 (GroupBox6) | TLabel | 4 | 69 | 98 | 13 | KG-FAX v1.1.3 | Isobar |
| Label11 (GroupBox6) | TLabel | 18 | 104 | 70 | 12 | Copyright K.G | version (ISOBAR_VERSION macro, = project version) |
| Label13 (GroupBox6) | TLabel | 3 | 121 | 99 | 12 | 2009/7/8 | © Sara Sakuragawa |
| Label2 (GroupBox6) | TLabel | 58 | 83 | 44 | 12 | Build000 | .syn compatible |
| (added) | TLabel |  |  |  |  |  | GPL v3+ |

## Form5 (TForm5, TFORM5) -- client area 280x111

Window caption: `色処理の設定`

| Control | Class | Left | Top | Width | Height | Caption | English |
|---|---|---|---|---|---|---|---|
| Label1 | TLabel | 22 | 8 | 45 | 12 | サンプル | Sample |
| Label2 | TLabel | 16 | 24 | 12 | 12 | 強 | S |
| Label3 | TLabel | 16 | 92 | 12 | 12 | 弱 | W |
| GroupBox1 | TGroupBox | 80 | 8 | 121 | 73 | 色処理の種類 | Type |
| RadioButton1 (GroupBox1) | TRadioButton | 16 | 16 | 73 | 17 | モノトーン | Monotone |
| RadioButton2 (GroupBox1) | TRadioButton | 16 | 32 | 57 | 17 | 色温度 | Color temp |
| RadioButton3 (GroupBox1) | TRadioButton | 16 | 48 | 81 | 17 | ブルーレイ | Blue ray |
| CheckBox1 | TCheckBox | 88 | 88 | 49 | 17 | 反転 | Invert |
| Panel1 | TPanel | 32 | 24 | 25 | 81 |  |  |
| Image1 (Panel1) | TImage | 1 | 1 | 23 | 79 |  | (gradient) |
| Button1 | TButton | 208 | 12 | 65 | 25 | OK | OK |
| Button2 | TButton | 208 | 44 | 65 | 25 | キャンセル | Cancel |

Radio mapping (docs/01 §4): モノトーン → palette mode 0, 色温度 → mode 3
(rainbow), ブルーレイ → mode 2 (blue ramp). Mode 1 has no radio. The
sample strip is a live preview: top = value 255 (strong), bottom = 0
(weak). Implemented as `gui/colordialog.*`.

## Form6 (TForm6, TFORM6) -- client area 290x103

Window caption: `ビットマップ種類選択`

| Control | Class | Left | Top | Width | Height | Caption | English |
|---|---|---|---|---|---|---|---|
| GroupBox1 | TGroupBox | 144 | 8 | 137 | 41 | 画像方向 | Orientation |
| RadioButton1 (GroupBox1) | TRadioButton | 16 | 16 | 41 | 17 | 横 | Land. |
| RadioButton2 (GroupBox1) | TRadioButton | 80 | 16 | 41 | 17 | 縦 | Port. |
| GroupBox2 | TGroupBox | 8 | 8 | 129 | 89 | 画像種類 | Image type |
| RadioButton3 (GroupBox2) | TRadioButton | 8 | 16 | 105 | 17 | 現在の表示画像 | Current view |
| RadioButton4 (GroupBox2) | TRadioButton | 8 | 32 | 105 | 17 | 760x500 Pixel | 760x500 Pixel |
| RadioButton5 (GroupBox2) | TRadioButton | 8 | 48 | 105 | 17 | 1140x750 Pixel | 1140x750 Pixel |
| RadioButton6 (GroupBox2) | TRadioButton | 8 | 64 | 105 | 17 | 2280x1500 Pixel | 2280x1500 Pixel |
| Button1 | TButton | 144 | 72 | 65 | 25 | OK | OK |
| Button2 | TButton | 216 | 72 | 65 | 25 | キャンセル | Cancel |
| CheckBox1 | TCheckBox | 152 | 52 | 129 | 17 | クリップボードへ出力 | (not implemented) |

Kind mapping (docs/01 §4): Current view = the on-screen preview bitmap;
760x500 = 3×3 box-average render (500 wide, ≤760 tall — the labels read
max-height × width); 1140x750 = 2×2; 2280x1500 = 1:1. Choosing Current
view disables Orientation. The clipboard checkbox is not implemented
(DEVIATIONS #12). Orientation mapping is deliberately swapped from the
original (横/縦): **Land. = 90° CCW rotated export, Port. = raster
as-is** (DEVIATIONS #14, user decision 2026-07-31). Implemented as
`gui/exportdialog.*`.

## Form7 (TForm7, TFORM7) -- client area 502x602

Window caption: `受信画像縦表示`

NOTE: Form7 is ORPHANED in the original — no button ever opens it
(verified 2026-07-30, docs/01 sec. 4). Not ported. The 縦表示 (Vertical)
button is instead an in-place 90° rotate toggle of the main buffer.

| Control | Class | Left | Top | Width | Height | Caption |
|---|---|---|---|---|---|---|
| Panel1 | TPanel | 0 | 0 | 502 | 602 |  |
| Image1 (Panel1) | TImage | 1 | 1 | 500 | 600 |  |

## Form8 (TForm8, TFORM8) -- client area 558x167

Window caption: `自動保存の設定` (English: "Auto-save settings")

| Control | Class | Left | Top | Width | Height | Caption | English |
|---|---|---|---|---|---|---|---|
| Label2 | TLabel | 8 | 8 | 136 | 12 | 自動データ保存先フォルダ | Auto-save folder |
| DriveComboBox1 | TDriveComboBox | 8 | 24 | 193 | 18 |  | (not implemented) |
| DirectoryListBox1 | TDirectoryListBox | 8 | 48 | 193 | 113 |  | (not implemented) |
| Button2 | TButton | 479 | 138 | 73 | 23 | キャンセル | Cancel |
| Button3 | TButton | 399 | 138 | 73 | 23 | OK | OK |
| FileListBox1 | TFileListBox | 208 | 24 | 185 | 113 |  | (not implemented) |
| FilterComboBox1 | TFilterComboBox | 208 | 141 | 185 | 20 |  | (format radio: .syn / .bmp) |
| GroupBox1 | TGroupBox | 399 | 72 | 153 | 64 | 画像保存時のサイズ | Saved image size |
| RadioButton1 (GroupBox1) | TRadioButton | 16 | 12 | 113 | 17 | 760x500 Pixel | 760x500 Pixel |
| RadioButton2 (GroupBox1) | TRadioButton | 16 | 27 | 97 | 17 | 1140x750 Pixel | 1140x750 Pixel |
| RadioButton3 (GroupBox1) | TRadioButton | 16 | 42 | 105 | 17 | 2280x1500 Pixel | 2280x1500 Pixel |
| GroupBox2 | TGroupBox | 399 | 25 | 153 | 41 | 掃引最大幅到達時の動作 | At max scan width |
| CheckBox1 (GroupBox2) | TCheckBox | 16 | 16 | 121 | 17 | 掃引を再スタート | Restart scan |

The drive/dir/file listboxes are replaced by a read-only path field +
"Browse…" (native directory chooser, DEVIATIONS #13). **FilterComboBox1
is the auto-save output FORMAT** (docs/01 §4: filter 0 = `*.syn`, 1 =
`*.bmp`, 2 = `*.jpg`); FLTK has no filter combo, so the port replaces it
with a two-button radio (.syn / .bmp — JPEG dropped, DEVIATIONS #15).
`FilterComboBox1Change` in the original enables the size radios + drive
controls only for bmp/jpg; the port disables the size radios when .syn
is chosen, same effect. Implemented as `gui/autosavedialog.*`.

## Form9 (TForm9, TFORM9) -- client area 384x135

Window caption: `入力デバイスの選択` (English: *Select input device*)

The port implements this as `gui/devicedialog.*`: a device list (ListView1)
with a chooser button. The original's Volume button (Button2) has no macOS
equivalent and is deactivated (DEVIATIONS #2); on the port it is an `@menu`
icon button that opens the device list.

| Control | Class | Left | Top | Width | Height | Caption | English |
|---|---|---|---|---|---|---|---|
| ListView1 | TListView | 8 | 8 | 273 | 121 |  | (device list) |
| Button1 | TButton | 288 | 8 | 89 | 25 | デバイス選択 | (device chooser) |
| Button2 | TButton | 288 | 72 | 89 | 25 | ﾎﾞﾘｭｰﾑｺﾝﾄﾛｰﾙ | (Volume — deactivated; see DEVIATIONS #2) |
| Button3 | TButton | 288 | 104 | 89 | 25 | キャンセル | Cancel |

