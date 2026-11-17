# 格付けの判定方法 / Rating Methodology

対象: Base Mainnet 上の ERC-8004（agent0）登録エージェント
判定スクリプト: `score.py`（条件は冒頭の `CONFIG` に集約）

Scope: ERC-8004 (agent0) agents registered on Base Mainnet.
Script: `score.py` (all thresholds live in the `CONFIG` block at the top).

---

## 日本語

### 1. データの出所

| データ | 取得元 | スクリプト |
|---|---|---|
| エージェントの登録情報（owner・agentWallet・接続先・名前・説明） | The Graph の agent0 サブグラフ | `fetch_agents.py` |
| 接続先の稼働状況 | 各エンドポイントへの HTTP リクエスト（1 日 3 回、日本時間 8:07 / 14:07 / 20:07、GitHub Actions） | `probe.py` |
| x402 の支払い | The Graph Network 上の x402 Base サブグラフ（`Cb56epg3EvQ6JRpPfknbkM54QxpzTvLa7mwKNQQfUyoj`） | `fetch_payments.py` |
| フィードバック（評価） | agent0 サブグラフの Feedback | `fetch_feedback.py` |

x402 サブグラフは The Graph の公式ドキュメントではなく第三者（Graph Advocate）の資料に記載されていたもの。抽出した 25 件の支払いを Base の公開 RPC の取引記録と照合し、受取先・金額・送金元がすべて一致することを確認している（`verify_payments.py`）。

### 2. 稼働状況の判定（v3 基準、1 回の観測ごと）

| 種類 | 送るリクエスト | 生存（alive）とみなす応答 |
|---|---|---|
| MCP | `initialize` の JSON-RPC POST | 2xx かつ JSON-RPC の `result` を含む |
| A2A | `/.well-known/agent.json` の GET（URL が既に card を指す場合はそのまま） | 2xx かつ `name` を含む JSON |
| Web / OASF | GET | 2xx |

- **alive_paid**: 402 で x402 の支払い条件（`PAYMENT-REQUIRED` ヘッダー、または本文の `x402Version` / `accepts`）を返す → 生存（有料）
- **auth_required**: 401 / 403、**uncertain**: 400 / 405 / 条件のない 402 → 生存には数えない
- **dead**: 404・その他の 4xx・5xx・本文が条件を満たさない 2xx・タイムアウト・DNS エラー・接続失敗
- 対象外: github.com 宛て（判定しない）、内部向けアドレス（アクセスしない）、不正な URL
- エージェントは、接続先のどれか 1 つが生存ならその回は生存
- 本文は最大 64KB を JSON として解析するだけで、保存・実行はしない

### 3. 前提の定義

**観測時点** — 定点観測（v3 基準の全件実行）のうち、前の回の開始から **60 分以内**に始まった回は、同じ「観測時点」にまとめる（続けて 60 分以内に始まった回はすべて同じ時点になる）。まとめた時点では、**その中の全回で生存した場合だけ**生存とみなす（例: 2 回をまとめたなら両方で生存が必要）。間隔の近い観測が重複して数えられるのを防ぐため。

**安定稼働** — **観測時点が 6 以上**、かつ生存だった時点の割合が **7/9（約 77.8%）以上**。

- 本番の想定は 9 時点中 7 時点以上。GitHub Actions の定期実行は遅延・スキップがありうるため、時点数の下限と割合の両方で判定する。
- 生存には alive_paid を含み、auth_required と uncertain は含まない。
- 分母はそのエージェントが観測された時点の数。

**基準時刻** — 初回レポートの基準時刻は **2026-11-17 12:00 UTC**。「過去 90 日」はこの時刻から数え（2026-08-19 12:00 〜 2026-11-17 12:00 UTC）、これより後の支払いは含めない。基準時刻は支払いの集計期間にだけ適用する。定点観測は 2026-11-17 21:26（日本時間、12:26 UTC）開始の回までを使う。

**第三者からの支払い** — 過去 90 日の x402 支払いのうち、支払人が次のいずれでもないもの（A には、この支払人が**異なる 3 人以上**必要）:
- そのエージェントの owner 本人
- 同じ owner の他エージェントの agentWallet
- 他の登録エージェントの agentWallet（全約 118 万体の登録情報で照合）

上記に当たる支払いを「自作自演の疑いのある支払い」とし、受取額に占める割合を計算する。

**健全な評価** — 次を除いた評価が、**異なる送り手 3 人以上**から付いていること:
1. `tag1` または `tag2` が `miner-vouch` / `botcoin`
2. 値が 0〜100 の範囲外
3. いずれかの 1 時間に 10 件以上送ったことのある送り手の評価（すべて）
4. 相互評価のペア: A の関係者が B を評価し、B の関係者が A を評価している場合、その両方向の評価
5. 自己評価: 送り手が評価先の owner 本人、または同じ owner の agentWallet

相互評価と自己評価は **owner 単位**で判定する。送り手アドレスは「owner 本人として」または「agentWallet として」代表する owner に対応づける。同じ owner の別エージェントを経由した相互評価も検出できる。自己評価の除外は条件表に明記されていないが、「異なる第三者からの評価」という趣旨に合わせて加えた（`CONFIG["exclude_self"]` で切り替え可能）。

### 4. 格付け（上から順に判定し、最初に当てはまったもの）

| 格付け | 条件 |
|---|---|
| **A** | 安定稼働 ＋ 過去 90 日に**異なる 3 人以上**の第三者からの支払いあり ＋ 自作自演の疑いのある支払いが受取額の 50% 未満 |
| **B** | 安定稼働 ＋（観測期間中に alive_paid の接続先あり、または 健全な評価あり） |
| **C** | 安定稼働のみ |
| **対象外** | 上記以外（不安定・停止・接続先なし） |

「alive_paid の接続先あり」は、観測期間中に 1 回以上 alive_paid と判定された接続先があること。

**同じ財布を共有するエージェント**は、支払い実績を 1 つの事業体のものとして扱う。A の条件を満たしたエージェントが同じ agentWallet に複数ある場合、A はその中の 1 体だけに付け、残りは B にする。代表の 1 体は、稼働率 → 生存回数 → 有料接続先の回数 → 登録の古さ（agentId が小さい順）で選ぶ。

**注記（`ratings_AB.csv` の note 列）** — A のうち、条件の最低ラインにいるエージェントに注記を付ける。

- 生存した観測時点がちょうど最低ライン（例: 9 時点中 7 時点）。観測時点の時間幅を 90 分にすると安定稼働を満たさない場合は、その旨も記す。
- 第三者の支払人がちょうど 3 人。

### 5. 既知の限界

- **ウォレットの共有**: 支払いは受取ウォレット単位で記録される。同じ agentWallet を共有するエージェントは 1 つの事業体として扱い、A は 1 体だけに付ける（B 以下の判定は各エージェントの実績のまま）。
- **自作自演の検出範囲**: 登録済みの owner・agentWallet との一致だけを見る。未登録の別の財布を使った自作自演は検出できない。
- **x402 以外の支払い**: x402 サブグラフは仲介業者（facilitator）経由の決済だけを記録する。直接の USDC 送金は含まない。
- **観測環境**: GitHub Actions はデータセンターの IP から接続するため、一部のサイト（例: basescan.org、x.com）は 403 を返し、auth_required と判定される。
- **github.com 宛ての接続先**は判定対象外。
- **観測の時刻と間隔の不均一（初回レポート）**: 定点観測は 1 日 3 回（日本時間 8:07 / 14:07 / 20:07）の予定だが、初回レポートの観測期間中は GitHub Actions の定期実行に最大 3 時間半程度の遅延や実行の欠落があった。欠落の一部は手動実行で補ったため、観測の時刻と間隔は均一ではない。間隔の近い回は観測時点としてまとめたうえで、安定稼働を「観測時点 6 以上 かつ 生存率 7/9 以上」で判定する。実際の観測日時、回ごとの集計、どの回をまとめたかは `results/summary.md` の「観測の記録」で確認できる。
- **試運転（2026-11-14）** は観測 1〜2 回分しかなく、観測時点 6 以上の条件を満たすエージェントがいないため、`--min-runs` で下限を一時的に下げて実行した。本番の判定は定点観測の終了後に、下限 6 時点のまま行う。

### 6. 次回からの変更予定

- **A の条件の追加**: 「第三者からの支払いが、異なる 3 日以上（UTC の暦日）にわたること」を加える。短期間にまとめて行われた支払いだけで A になることを防ぐため。初回レポートの判定には適用していない。

---

## English

### 1. Data sources

| Data | Source | Script |
|---|---|---|
| Agent registry (owner, agentWallet, endpoints, name, description) | The Graph, agent0 subgraph | `fetch_agents.py` |
| Endpoint liveness | One HTTP request per endpoint, three times a day (08:07 / 14:07 / 20:07 JST) on GitHub Actions | `probe.py` |
| x402 payments | x402 Base subgraph on The Graph Network (`Cb56epg3EvQ6JRpPfknbkM54QxpzTvLa7mwKNQQfUyoj`) | `fetch_payments.py` |
| Feedback | agent0 subgraph, `Feedback` entity | `fetch_feedback.py` |

The x402 subgraph ID comes from third-party documentation (Graph Advocate), not The Graph's official docs. A sample of 25 payments was checked against Base public RPC transaction receipts, and recipient, amount and payer matched in all 25 (`verify_payments.py`).

### 2. Liveness per observation (v3 criteria)

| Kind | Request | Counted as alive when |
|---|---|---|
| MCP | JSON-RPC `initialize` POST | 2xx and the body contains a JSON-RPC `result` |
| A2A | GET `/.well-known/agent.json` (or the URL as-is if it already points to a card) | 2xx and a JSON body with `name` |
| Web / OASF | GET | 2xx |

- **alive_paid**: HTTP 402 with x402 payment terms (`PAYMENT-REQUIRED` header, or `x402Version` / `accepts` in the body). Counted as alive (paid).
- **auth_required** (401/403) and **uncertain** (400, 405, or 402 without terms) are not counted as alive.
- **dead**: 404, other 4xx, 5xx, 2xx without a valid body, timeout, DNS error, or connection failure.
- Not judged: github.com targets (skipped), internal addresses (never contacted), invalid URLs.
- An agent is alive in a run if any of its endpoints is alive.
- Bodies are parsed as JSON (max 64KB) only. They are never stored or executed.

### 3. Definitions

**Observation point** — a full v3 run that starts **within 60 minutes** of the previous run's start is merged into the same observation point, so a chain of runs each within 60 minutes of the last becomes one point. At a merged point, an agent counts as alive **only if it was alive in every run** of that point (for two merged runs, both must be alive). This keeps closely spaced runs from being counted twice.

**Stable** — the agent was observed at **at least 6 observation points** and was alive at least **7/9 (≈77.8%)** of them.

- The plan is 7 of 9 points. Scheduled GitHub Actions runs can be delayed or skipped, so both a minimum number of points and a ratio are required.
- alive_paid counts as alive; auth_required and uncertain do not.
- The denominator is the number of points at which the agent was observed.

**Reference time** — the first report's reference time is **2026-11-17 12:00 UTC**. The 90-day payment window is counted back from it (2026-08-19 12:00 to 2026-11-17 12:00 UTC), and later payments are excluded. The reference time applies only to the payment window; observations include the run that started at 21:26 JST (12:26 UTC) on 2026-11-17.

**Third-party payment** — an x402 payment in the last 90 days whose payer is none of the following (grade A requires **at least 3 distinct** such payers):
- the agent's owner
- the agentWallet of another agent with the same owner
- the agentWallet of any other registered agent (checked against all ~1.18M registrations)

Payments that do match one of these are "suspicious", and their share of the total amount received is computed.

**Healthy feedback** — feedback from **at least 3 distinct senders** remains after removing:
1. records whose `tag1` or `tag2` is `miner-vouch` or `botcoin`
2. values outside 0–100
3. all feedback from any sender that ever sent 10 or more within one hour
4. mutual-rating pairs: if A's side rates B and B's side rates A, both directions are removed
5. self-rating: the sender is the target's owner or an agentWallet of the same owner

Mutual and self-rating are evaluated **per owner**. A sender address maps to the owners it represents, either as the owner itself or as an agentWallet. This also catches mutual rating routed through different agents of the same owner. Excluding self-rating is not in the original table; it was added to match the intent of "distinct third parties" and can be switched off with `CONFIG["exclude_self"]`.

### 4. Grades (evaluated top to bottom; the first match wins)

| Grade | Condition |
|---|---|
| **A** | Stable + third-party payments from **at least 3 distinct payers** in the last 90 days + suspicious payments are less than 50% of the amount received |
| **B** | Stable + (an endpoint judged alive_paid during the observation period, or healthy feedback) |
| **C** | Stable only |
| **Not rated** | Everything else (unstable, dead, or no endpoint) |

"An endpoint judged alive_paid" means at least one observation run judged one of the agent's endpoints alive_paid.

**Agents sharing a wallet** are treated as one business for payments. If several agents with the same agentWallet qualify for A, only one gets A and the rest get B. The representative is chosen by uptime, then number of alive runs, then number of alive_paid runs, then registration age (smallest agentId).

**Notes (`note` column in `ratings_AB.csv`)** — A-rated agents that sit exactly at a threshold get a note.

- Alive at exactly the minimum number of observation points (e.g. 7 of 9). If the agent would not be stable with a 90-minute merge window, the note says so.
- Exactly 3 third-party payers.

### 5. Known limitations

- **Shared wallets**: payments are recorded per receiving wallet. Agents sharing an agentWallet are treated as one business and only one of them can get A (grades B and below still use each agent's own record).
- **Self-dealing coverage**: only matches against registered owners and agentWallets. Self-dealing through unregistered wallets is not detected.
- **Non-x402 payments**: the x402 subgraph only records facilitator-settled payments. Plain USDC transfers are not included.
- **Observation network**: GitHub Actions connects from data-center IPs, and some sites (e.g. basescan.org, x.com) answer 403, which is recorded as auth_required.
- **github.com endpoints** are not judged.
- **Uneven observation times (first report)**: observations are scheduled three times a day (08:07 / 14:07 / 20:07 JST). During the first report's observation period, however, GitHub Actions scheduled runs were delayed by up to about 3.5 hours or skipped entirely. Some missing runs were filled with manual runs, so observation times and intervals are uneven. Closely spaced runs are merged into observation points, and stability is judged as "at least 6 observation points and alive at 7/9 or more of them". The actual observation times, per-run figures, and which runs were merged are in the "Observation record" section of `results/summary.md`.
- **The trial runs (2026-11-14)** had only 1–2 observation runs, so no agent met the minimum of 6 observation points. They were run with `--min-runs` to lower the minimum temporarily. The production rating runs after the observation period with the minimum kept at 6 points.

### 6. Planned changes for the next report

- **Additional condition for grade A**: third-party payments must span **at least 3 distinct days** (UTC calendar days). This keeps payments bunched into a short burst from being enough for an A. It was not applied to the first report.
