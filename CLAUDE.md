# aon-co-jp fork メモ

- 方針(2026-11-19、ユーザー指示): Node.js・REST API全廃、JSONはRS-JSONへ統一(二重使用しない)。
- 本forkでは Node.js アドオン `examples/addon.node` と、その専用CI `examples.yml`、`CMAKE_JS_VERSION`分岐を削除した。
- 未了: `examples/server`のREST(httplib)・`nlohmann/json`のRS-JSON(Rust)置換は別作業。上流`ggml-org/whisper.cpp`との差分が増えるため、取り込み時は競合に注意。

## 2026-11-19 方針: Node.js・REST API を使わない / JSONはRS-JSONだけ(日本語)
- **やること**: Node.js と REST API をやめます。JSONは `RS-JSON` だけで読み書きし、ほかのJSON部品(`serde_json`・`nlohmann` など)と**二重に使いません**。
- **済んだこと**: `whisper.cpp` の fork(`aon-co-jp/whisper.cpp`)から Node.js アドオン `examples/addon.node` と専用CIを消しました。洗い出しの結果は `runo/NODEJS-REMAINING-20261119.md`。
- **まだのこと**: (1)置き換えられるもの(`open-upstream-watch` の試験、`Reaxt/clients/js`、`2Poem` の例)、(2)JSONの二重使用の洗い出し、(3)Node.jsが必須のもの(VS Code拡張、Tauriの画面ビルド)は、やめてよいか本人の判断待ち、(4)`whisper.cpp` の `examples/server`(REST+nlohmann)。
- **VPS**: `whisper.cpp` のVPSのクローンは2コミット遅れのまま(未更新)。


### English (simple words, easy to read)
- **What we do**: We stop using Node.js and REST APIs. We read and write JSON with `RS-JSON` only. We do **not** use two JSON tools at the same time (for example `serde_json` or `nlohmann` together with RS-JSON).
- **What is done**: In our copy of `whisper.cpp` (`aon-co-jp/whisper.cpp`), we deleted the Node.js add-on `examples/addon.node` and the tests that only ran it. The full list is in `runo/NODEJS-REMAINING-20261119.md`.
- **What is not done yet**: (1) things we can replace now (the `open-upstream-watch` test, `Reaxt/clients/js`, the `2Poem` example); (2) finding places that use two JSON tools; (3) things that really need Node.js (VS Code extensions, Tauri screen builds) - we wait for the owner to decide; (4) `examples/server` in `whisper.cpp` (REST + nlohmann).
- **VPS**: the VPS copy of `whisper.cpp` is still 2 commits behind (not updated).

## Versionless API の方針 / Versionless API policy (2026-11-25)
**日本語**: このリポジトリのAPIは **Versionless**(URL・パッケージ名に `v1` 等を付けず、互換性を壊さず足すだけで進化させる)。欄は足すだけ・足す欄は省略可能・知らない欄は無視・名前変更は新名を足して旧を非推奨に(期限は既定で6か月以上、使用0を確認してから削除)・enumは未知の値を許容・安全上の緊急修正は例外。既存の `.v1` は呼び出し側を壊さないよう当面残し、版なしの別名を追加して段階的に畳む。詳細・出典は [`runo/API-POLICY.md`](https://github.com/aon-co-jp/runo/blob/main/API-POLICY.md) 末尾。適用はこれから順に行う。
**English**: This repository's APIs are **Versionless**: no `v1` in URLs or package names; evolve by adding only, without breaking compatibility. Only add fields, new fields are optional, ignore unknown fields, rename by adding the new name and deprecating the old (default deadline 6 months or more, remove only after confirming zero use), tolerate unknown enum values, emergency security fixes are the exception. Existing `.v1` names stay for now so callers do not break; a versionless alias is added and `.v1` is retired in stages. Details and sources: the end of [`runo/API-POLICY.md`](https://github.com/aon-co-jp/runo/blob/main/API-POLICY.md). Rollout is step by step.

## API 方針: REST 全廃・gRPC / Connect へ
**日本語**: このリポジトリでは **REST API を新規に作らない**。ブラウザ・`curl` から呼ぶ API は **Connect の単純呼び出し**(`POST /<パッケージ>.<Service>/<Method>`、`content-type: application/json`)、サービス間は **gRPC / gRPC-web**。GraphQL は読み取り専用。既存の REST は触るときに移し、移したら旧経路は消す。JSON は RS-JSON(`from_str_secure` / `parse_secure`)で読む。例外は相手の規格が形を決めるもの(ACME、OAuth/OIDC のコールバック、第三者の Webhook、ヘルスチェック、静的ファイル、OTLP)のみ。迷ったら Connect。正本は [`runo/API-POLICY.md`](https://github.com/aon-co-jp/runo/blob/main/API-POLICY.md)。
**English**: This repository **does not add new REST APIs**. APIs called from browsers or `curl` use **Connect unary calls** (`POST /<package>.<Service>/<Method>`, `content-type: application/json`); service-to-service calls use **gRPC / gRPC-web**. GraphQL is read-only. Existing REST is migrated when touched and the old route is removed. Read JSON with RS-JSON (`from_str_secure` / `parse_secure`). Exceptions are only shapes dictated by someone else's standard (ACME, OAuth/OIDC callbacks, third-party webhooks, health checks, static files, OTLP). When in doubt, use Connect. Source of truth: the link above.

## 完成まで作り切る方針 / Finish everything policy (2026-11-27)
**日本語**: 全リポジトリで、**未確認は自動で確認し、未実施は実施し、未了は完成させる**(確認を求めず進める)。使用頻度が低いと判断される機能や、**1から作り直す大規模開発になる内容でも、やめずに最後まで開発・TEST・DEBUGする**(別の方式で済ませない、後回しにしない)。進め方: 調査(英語・日本語の検索とGitHub) → 設計 → 実装 → 実機/実データで確認 → 失敗は原因を直して再試験 → README・CLAUDE・PORTING・POLICY を日英で更新 → push。「できた」と書くのは実測した範囲だけ、確認できなかった所は「未確認」と正直に書く。例外(確認を取る): 元に戻せない削除、秘密の公開、所有者の認証・決済・実機操作が要るもの、利用者のシステム設定の変更。
**English**: In every repository, **verify what is unverified, do what is not yet done, and finish what is unfinished** automatically, without asking. Even features that look rarely used, or work that means **a large rewrite from scratch, are developed, tested and debugged to the end** (no substituting another approach, no postponing). Method: research (English and Japanese search plus GitHub) -> design -> implement -> verify on real devices or data -> fix causes and retest on failure -> update README, CLAUDE, PORTING and POLICY in Japanese and English -> push. Write "done" only for what was measured, and say "unverified" honestly for the rest. Exceptions that still need confirmation: irreversible deletions, publishing secrets, anything needing the owner's authentication, payment or real-device actions, and changes to the user's system settings.