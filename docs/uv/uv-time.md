# uvの処理速度
## パッケージインストール
pip：
```sh
.venv ❯ time pip install numpy
Collecting numpy
  Downloading numpy-2.3.4-cp311-cp311-macosx_14_0_arm64.whl.metadata (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.1/62.1 kB 2.0 MB/s eta 0:00:00
Downloading numpy-2.3.4-cp311-cp311-macosx_14_0_arm64.whl (5.4 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 5.4/5.4 MB 32.0 MB/s eta 0:00:00
Installing collected packages: numpy
Successfully installed numpy-2.3.4

[notice] A new release of pip is available: 24.0 -> 25.2
[notice] To update, run: pip install --upgrade pip
pip install numpy  2.00s user 0.89s system 35% cpu 8.111 total
```

uv：
```sh
.venv ❯ time uv add numpy
Resolved 2 packages in 633ms
Prepared 1 package in 475ms
Installed 1 package in 120ms
 + numpy==2.3.4
uv add numpy  0.16s user 0.27s system 29% cpu 1.458 total
```
