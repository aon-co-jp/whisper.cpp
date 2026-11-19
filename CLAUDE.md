# aon-co-jp fork メモ

- 方針(2026-11-19、ユーザー指示): Node.js・REST API全廃、JSONはRS-JSONへ統一(二重使用しない)。
- 本forkでは Node.js アドオン `examples/addon.node` と、その専用CI `examples.yml`、`CMAKE_JS_VERSION`分岐を削除した。
- 未了: `examples/server`のREST(httplib)・`nlohmann/json`のRS-JSON(Rust)置換は別作業。上流`ggml-org/whisper.cpp`との差分が増えるため、取り込み時は競合に注意。
