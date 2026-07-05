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

## ファームウェア生成スクリプト`generate_merged_firmware.py`

このリポジトリには、E配布用の結合済みファームウェアイメージを生成する `generate_merged_firmware.py` を含めています。

このスクリプトはPlatformIOのカスタムターゲットとして `firmware` を追加します。通常のアプリケーションバイナリに、ブートローダーやパーティションテーブルなどの追加イメージを結合し、1つのファームウェアファイルとして出力します。

## 使い方

プロジェクト側の `platformio.ini` で、`extra_scripts` に `generate_merged_firmware.py` を指定します。

```ini
[env]
extra_scripts =
  config/generate_merged_firmware.py
```

ファームウェアを生成するには、次のコマンドを実行します。

```sh
pio run -t firmware
```

## 基本設定

必要に応じて、プロジェクト側の `platformio.ini` にカスタムオプションを指定します。

```ini
[env:m5stack-basic]
custom_firmware_dir = firmware
custom_firmware_name = example
custom_firmware_target = m5stack-basic
# custom_firmware_env = ${this.__env__}
custom_firmware_suffix = bin
```

## カスタムオプション

| オプション                     | 説明                                                                                                                 | 省略時                     |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| `custom_firmware_dir`          | 生成したファームウェアを出力するディレクトリです。相対パスの場合はプロジェクトルートからの相対パスとして扱われます。 | `firmware`                 |
| `custom_firmware_name`         | 出力ファイル名に含めるファームウェア名です。                                                                         | プロジェクトディレクトリ名 |
| `custom_firmware_target`       | 出力ファイル名に含めるターゲット名です。                                                                             | `custom_firmware_env`の値  |
| `custom_firmware_env`          | 出力ファイル名に含める環境名です。                                                                                   | PlatformIOの環境名         |
| `custom_firmware_suffix`       | 出力ファイルの拡張子です。                                                                                           | `bin`                      |
| `custom_firmware_dependencies` | `firmware` ターゲット実行前に追加で依存させるターゲットやファイルを指定します。                                      | なし                       |

## バージョン指定

出力ファイル名に含めるバージョンは、次の順で決定されます。

1. `custom_firmware_version`
2. `custom_firmware_version_file`
3. GitHub Actionsの `GITHUB_REF_NAME`
4. `dev`

明示的に指定する場合は、次のようにします。

```ini
[env]
custom_firmware_version = v1.0.0
```

`custom_firmware_version_file` を指定すると、ファイル内からバージョン文字列を読み取ります。

```ini
[env]
custom_firmware_version_file = version.h
```

バージョンファイルの例:

```cpp
const char* FIRMWARE_VERSION = "v1.0.0";
```

標準では、次のような形式のバージョン文字列を読み取ります。

```text
v1.0.0
1.0.0
v1.0.0-beta.1
v1.0.0+build.1
```

読み取り用の正規表現を変更したい場合は、`custom_firmware_version_regex` を指定します。

```ini
[env]
custom_firmware_version_regex = VERSION "v?(\d+\.\d+\.\d+)"
```

出力ファイル名では、先頭の `v` は取り除かれます。

## チップ指定

通常はPlatformIOのボード設定からチップ種別を自動判定します。

必要に応じて、明示的に指定できます。

```ini
[env]
custom_firmware_chip = esp32s3
```

チップ種別は、次の順で決定されます。

1. `custom_firmware_chip`
2. `board_build.mcu`
3. ボード定義の `build.mcu`

## Flash設定

必要に応じて、`merge_bin` に渡すFlash設定を指定できます。

```ini
[env]
custom_firmware_flash_mode = dio
custom_firmware_flash_freq = 80m
custom_firmware_flash_size = 16MB
```

## 出力ファイル名

生成されるファームウェアのファイル名は、次の形式になります。

```text
{name}_{target}_firmware_{version}.{suffix}
```

例えば、次の設定の場合:

```ini
custom_firmware_name = example
custom_firmware_target = m5stack-basic
custom_firmware_version = v1.0.0
custom_firmware_suffix = bin
```

次のファイルが生成されます。

```text
example_m5stack-basic_firmware_1.0.0.bin
```

出力先は `custom_firmware_dir` で指定したディレクトリです。

## 注意事項

このスクリプトはESP32系のPlatformIO環境での利用を想定しています。

内部ではPlatformIOが提供する `FLASH_EXTRA_IMAGES`、`ESP32_APP_OFFSET`、`UPLOADER` などの値を使用して、`merge_bin` によりファームウェアを結合します。

`FLASH_EXTRA_IMAGES` が空の場合や、アプリケーションバイナリが見つからない場合はエラーになります。