# ワークショップ トラブルシューティング集

ワークショップ中に発生しやすいエラーと、AI エージェントへの依頼文を用途別にまとめています。
**環境側の修正はエージェントに任せる方針**のため、エラーが出たら以下の指示文をそのままチャットに貼り付けてください。

依頼文はエージェントに「原因を決めつけず、可能性を確認してから対処する」ように書かれています。

---

## UPF デプロイ: 新 UPF コンテナが起動しない（`Exited (126)` / `Permission denied`）

### 症状

- `docker ps` を見ても `upf-new` コンテナが表示されない（ずっと空のまま）
- または `docker ps -a` で `upf-new` が `Exited (126)` になっている
- 新 UPF EC2 のログに `Permission denied` 系のエラーが出ている

### エージェントへの依頼文

以下をそのままチャットに貼り付けてください:

```
upf-new コンテナが起動に失敗しているみたい。docker ps -a で状態を確認して、
Exited だった場合は docker logs で原因を調べて。

もし config/upf-iptables.sh の実行権限（x ビット）が無いことが原因なら、
git update-index --chmod=+x で実行権限を付けて push し直してほしい。
違う原因なら、まず原因を報告してから対処を提案して。

修正を push した場合は、新 UPF EC2 で git pull して
docker compose up -d --force-recreate で反映して。
```

### 完了確認

```
新 UPF EC2 の upf-new コンテナが Up 状態になっているか確認して。
```

`docker ps` の出力で `upf-new` が `Up` と表示されれば成功です。

---

## UPF デプロイ: 新 UPF との PFCP Association が確立しない

### 症状

- ヘルスチェック実行時に SMF のログに PFCP 関連のエラーが出る。例:
  ```
  Host lookup failed: lookup upf.free5gc.org on 127.0.0.11:53: no such host
  Failed to setup an association with UPF[...(0.0.0.0)]
  error: no destination IP address is specified
  ```
- SMF から新 UPF への PFCP 接続が「未確立」と判定される
- 既存スライスは問題ないが、新スライスだけが使えない

### エージェントへの依頼文

```
新 UPF との PFCP Association が確立していないみたい。
SMF 側の設定（config/smfcfg.yaml）で、新 UPF (UPF2) の nodeID / addr / endpoints が
0.0.0.0 などのプレースホルダーのまま残っていないかを確認してほしい。

もし 0.0.0.0 のままだったら、CloudFormation Outputs から新 UPF EC2 の
実際の Private IP を取得して、該当箇所を実 IP に書き換えて push して。
違う原因なら、まず原因を報告してから対処を提案して。

修正を push した場合は、free5gc EC2 で git pull して
docker compose up -d --force-recreate free5gc-smf で反映して。
```

### 完了確認

```
SMF のログで新 UPF との PFCP Association Setup が成功しているか確認して。
```

`PFCP Association Setup Response received` など成功を示すログが出れば OK です。

---
