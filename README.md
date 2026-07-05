# M5Stack向けPlatformIO共通設定

M5Stack向けのPlatformIO設定をまとめたリポジトリです。

このリポジトリは、各PlatformIOプロジェクトからGit submoduleとして追加して使用します

## 共通INIファイル：`platformio-m5stack.ini`

### 使い方

プロジェクトにsubmoduleとして追加します。

```sh
git submodule add https://github.com/3110/m5stack-platformio-config.git config
```

プロジェクト側の `platformio.ini` で、次のように指定します。

```ini
[platformio]
extra_configs =
  config/platformio-m5stack.ini
```

### タグを指定して使用する場合

安定した設定を使いたい場合は、submoduleを特定のタグに固定できます。

```sh
cd config
git checkout v1.0.0

cd ../..
git add .gitmodules config
git commit -m "Add m5stack PlatformIO config"
```

`extra_configs` ではタグを直接指定できません。`extra_configs` はローカルにあるINIファイルを読み込むだけなので、使用するバージョンはGit submodule側で固定します。

### 更新方法

最新版に更新する場合は、次のようにします。

```sh
git submodule update --remote config
```

別のタグに更新する場合は、submodule内でタグをcheckoutしてから、親リポジトリ側でsubmoduleの参照先をコミットします。

```sh
cd config
git fetch --tags
git checkout v1.0.0

cd ..
git add config
git commit -m "Update m5stack PlatformIO config to v1.0.0"
```

## ファームウェア生成スクリプト`generate_user_custom.py`

このリポジトリには、M5Stack向けの共通PlatformIO設定に加えて、Uファームウェアを生成するための `generate_user_custom.py` が含まれています。

このスクリプトはPlatformIOの `extra_scripts` として使用します。通常のビルド結果とファイルシステムイメージを結合し、ファームウェアファイルを生成します。

### 使い方

プロジェクト側の `platformio.ini` で、`extra_scripts` に `generate_user_custom.py` を指定します。

```ini
[env]
extra_scripts =
  config/generate_user_custom.py
```

あわせて、次のカスタムオプションを指定します。

```ini
[env]
custom_firmware_dir = firmware
custom_firmware_version_file = version.h
custom_firmware_target = m5stack-basic
custom_firmware_name = example
custom_firmware_suffix = bin
```

### カスタムオプション

| オプション | 説明 |
|---|---|
| `custom_firmware_dir` | 生成したファームウェアを出力するディレクトリです。相対パスの場合はプロジェクトルートからの相対パスとして扱われます。 |
| `custom_firmware_version_file` | ファームウェアバージョンを記述したファイル名です。`custom_firmware_dir` 内のファイルとして読み込まれます。 |
| `custom_firmware_target` | 出力ファイル名に含めるターゲット名です。通常はPlatformIOの環境名や対象機種名を指定します。 |
| `custom_firmware_name` | 出力ファイル名に含めるファームウェア名です。 |
| `custom_firmware_suffix` | 出力ファイルの拡張子です。通常は `bin` を指定します。 |

### バージョンファイル

`custom_firmware_version_file` には、`"v1.0.0"` のような形式のバージョン文字列を含めてください。

例:

```cpp
const char* FIRMWARE_VERSION = "v1.0.0";
```

スクリプトはこのファイルから `vX.Y.Z` 形式の文字列を探し、出力ファイル名には先頭の `v` を除いたバージョン番号を使用します。

### 出力ファイル名

生成されるファームウェアのファイル名は、次の形式になります。

```text
{custom_firmware_target}_{custom_firmware_name}_firmware_{version}.{custom_firmware_suffix}
```

例えば、次の設定の場合:

```ini
custom_firmware_target = m5stack-basic
custom_firmware_name = example
custom_firmware_suffix = bin
```

バージョンファイルに `"v1.0.0"` が含まれていれば、次のファイルが生成されます。

```text
m5stack-basic_example_firmware_1.0.0.bin
```

### 実行方法

PlatformIOのカスタムターゲットとして `firmware` が追加されます。

```sh
pio run -t firmware
```

PlatformIOのメニューの「Custom」から「Generate User Custom」を選択しても実行できます。

このターゲットは `buildfs` に依存しているため、ファームウェア生成時にファイルシステムイメージもあわせてビルドされます。
