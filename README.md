# M5Stack向けPlatformIO共通設定

M5Stack向けのPlatformIO設定をまとめた共通INIファイルです。

このリポジトリは、各PlatformIOプロジェクトからGit submoduleとして追加し、`platformio.ini` の `extra_configs` で `platformio-m5stack.ini` を読み込んで使用することを想定しています。

## 使い方

プロジェクトにsubmoduleとして追加します。

```sh
git submodule add https://github.com/3110/m5stack-platformio-ini.git config/m5stack-platformio-ini
```

プロジェクト側の `platformio.ini` で、次のように指定します。

```ini
[platformio]
extra_configs =
  config/m5stack-platformio-ini/platformio-m5stack.ini
```

## タグを指定して使用する場合

安定した設定を使いたい場合は、submoduleを特定のタグに固定できます。

```sh
cd config/m5stack-platformio-ini
git checkout v1.0.0

cd ../..
git add .gitmodules config/m5stack-platformio-ini
git commit -m "Add m5stack PlatformIO config"
```

`extra_configs` ではタグを直接指定できません。`extra_configs` はローカルにあるINIファイルを読み込むだけなので、使用するバージョンはGit submodule側で固定します。

## 更新方法

最新版に更新する場合は、次のようにします。

```sh
git submodule update --remote config/m5stack-platformio-ini
```

別のタグに更新する場合は、submodule内でタグをcheckoutしてから、親リポジトリ側でsubmoduleの参照先をコミットします。

```sh
cd config/m5stack-platformio-ini
git fetch --tags
git checkout v1.0.0

cd ../..
git add config/m5stack-platformio-ini
git commit -m "Update m5stack PlatformIO config to v1.0.0"
```
