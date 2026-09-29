# Applied SIM for Advisory（Draft 0.5）

**ASG Conformance Target:** Applied SIM Standard Guidelines v0.1

## 1. 目的

企業（エンドユーザー）が実施した技術調査、またはAIによる技術調査の出力に対し、第三者としてのAI評価により不足観点を指摘し、Second Opinionとして提示する。ユーザーは提示された内容を目的に照らして採否判断する。

## 2. スコープ

- 対象：技術情報に関する調査結果・調査出力
- 対象外：調査そのものの実施（本プラクティスは評価・助言に限定する）
- 参照文書：hexbrick-tech/sim（Foundation）、hexbrick-tech/applied-sim-standard-guidelines（ASG v0.1）

## 3. 語彙マッピング

| SIM概念             | Advisoryでの定義                                                                                                           |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Observation       | 企業調査結果・AI出力・RAG返却の原文そのもの（要約・評価前）                                                                                       |
| Interpretation    | 原文が何を主張・含意しているかについての読み取り、およびObservation間の関係づけ・比較                                                                       |
| Evaluation        | 目的・基準に照らした判断（Gap判定、Opinion、採否）                                                                                         |
| Undefined         | 観測はされたが、まだEvaluation（Gap判定）を経ていない状態。永続的なArtifactとしては保持されない一時的条件。EvaluationによりGapとして記録されるか、不足なしとして処理が完了する               |
| Unknown           | 判断に必要な情報が入力に含まれていない状態                                                                                                  |
| Semantic Boundary | ある論点・立場が安定して扱える状態（Established）                                                                                         |
| Semantic Probe    | ユーザーへの追加確認（観測事実を引き出す形）                                                                                                 |
| Review Cycle      | Inquiry Frame配下の評価実行単位そのもの。Snapshotはこの実行単位を対象にした事後的な射影（Projection）であり、両者は責務として独立している。Review CycleはSnapshotが存在しなくても成立する |

## 4. プロセス
```
Step 1: 目的入力 + 調査結果/AI出力の共有（Inquiry Frame確定）
        ↓
   ┌──▶ Step 2: 不足観点の評価
   │        ↓
   │    Step 3: Opinion提示（必要に応じRAG補足）
   │        ↓
   │    Step 4: ユーザーが採否判断・ギャップ入力
   │        ↓ (任意)
   │    [SnapshotReport 出力]
   │        ↓
   └────── Step 2へフィードバック

```

Purpose/Scope/Criteriaを変更する場合は、新たなInquiry Frameを作成する。そのInquiry Frame配下でReview Cycleは新たに開始される（Cycle番号も1から再スタート）。Step 2〜4は、Inquiry Frameが変わらない限り反復されるサイクルである。
```
IF-001
 ├ RC-001 (Cycle 1)
 ├ RC-002 (Cycle 2)
 └ RC-003 (Cycle 3)

IF-002（Purpose等変更により新規作成）
 ├ RC-001 (Cycle 1、番号は独立して再スタート)
 └ RC-002

```

## 5. 成果物（Artifacts）

### 5.1 Inquiry Frame

| フィールド             | 内容                     |
| ----------------- | ---------------------- |
| ID                | IF-001                 |
| Purpose           | 調査目的                   |
| Scope             | 調査対象範囲                 |
| Judgment Criteria | 「十分」の判定基準（無ければUnknown） |
| Captured At       | 記録日時                   |

### 5.2 Review Cycle

| フィールド             | 内容                |
| ----------------- | ----------------- |
| ID                | RC-001            |
| Ref Inquiry Frame | IF-ID             |
| Started           | 開始日時              |
| Finished          | 終了日時（進行中はUnknown） |

### 5.3 Observation Ledger

| フィールド           | 内容                                           |
| --------------- | -------------------------------------------- |
| ID              | O-001                                        |
| Source Class    | Enterprise-Research / AI-Output / RAG-Return |
| Source          | 具体的出所                                        |
| Query           | RAG-Returnの場合のみ、実際のクエリ                       |
| Raw Observation | 原文（Markdown / JSON / コード等を含む）                |
| Captured At     | 記録日時                                         |
| Context         | 生成状況、または紐づくGap ID（RAG-Returnの場合）             |

### 5.4 Gap Record

| フィールド            | 内容                                                             |
| ---------------- | -------------------------------------------------------------- |
| ID               | G-001                                                          |
| Ref Review Cycle | RC-ID                                                          |
| Gap Description  | 不足内容                                                           |
| Basis            | Purpose/Scope/Criteriaのどれに照らしたか                                |
| Origin           | 原則 Model-derived。例外的にユーザー入力由来                                  |
| Status           | Open / Opinion Issued / Adopted / Rejected / Partially Adopted |
| Supersedes       | 引き継いだ旧Gap ID（再訪時のみ）                                            |

Inquiry Frameへの到達は `Ref Review Cycle → Review Cycle.Ref Inquiry Frame` の単一経路に統一する。Gap Record自体にInquiry Frameへの参照は持たせない。

**Status運用ルール：**

- Openは未処理ではなく、「EvaluationによりGapとして成立し、未終結である状態」を指す。それ自体が報告価値を持ち、滞留を問題視しない。
- Opinion Issuedのままの改訂は無制限に許容する（Opinion側のSupersedesで表現）。
- 終端状態（Adopted / Rejected / Partially Adopted）からOpenへの巻き戻しは行わない。同一論点の再訪は新規Gap＋Supersedesで表現する。
```
Observation
  ↓
Undefined（Evaluation前の一時条件）
  ↓
Evaluation
  ↓
Gap: Open（成立・未終結）

```

### 5.5 Opinion

| フィールド                          | 内容                                                                                         |
| ------------------------------ | ------------------------------------------------------------------------------------------ |
| ID                             | OP-001                                                                                     |
| Ref Review Cycle               | RC-ID                                                                                      |
| Refers To Gap                  | G-ID                                                                                       |
| Opinion Text                   | 具体的な意見・提案                                                                                  |
| Used Observations              | 根拠としたObservation ID群                                                                       |
| Includes Model-Derived Content | Yes / No。Used Observationsの言い換え・解釈（関係づけ・比較を含む）を超えて、観測範囲に含まれない一般知識・業界慣行・外部情報を根拠に用いている場合はYes |
| Supersedes                     | 同一Gap内で改訂した旧Opinion ID（該当時のみ）                                                              |

**Supersedesスコープ：** 同一Gap内のみで発生する。Gapをまたぐ系譜はGap.Supersedes経由で辿り、Opinion同士がGapをまたいで直接参照し合うことはしない。

Observation/RAGの別を知りたい場合は、Used ObservationsのIDをSource Class別に参照すれば導出できるため、Opinion側には個別フィールドを持たせない。

### 5.6 Reassessment Log

| フィールド      | 内容                     |
| ---------- | ---------------------- |
| ID         | R-001                  |
| New Gap    | 新規Gap ID               |
| Supersedes | 引き継いだ旧Gap ID           |
| Trigger    | 契機となった新規Observation ID |

本ログは同一Inquiry Frame内でのGap Supersedesのみを扱う。Inquiry Frame自体の変遷（Purpose/Scope/Criteria変更）はInquiry Frameの系譜（複数のIFが並立し得る）で表現し、本ログの対象外とする。

### 5.7 SnapshotReport（任意、Step 4後に出力）

| フィールド                      | 内容                        |
| -------------------------- | ------------------------- |
| ID                         | SR-001                    |
| Ref Review Cycle           | RC-ID                     |
| Generated At               | 出力日時                      |
| Inquiry Frame Ref          | IF-ID（参照のみ）               |
| Gap Status Summary         | Status別件数                 |
| Terminal Gaps (this cycle) | このサイクルで終端に至ったGap ID一覧     |
| Open Gaps (累積)             | 現在Openの全Gap ID＋滞留期間（事実のみ） |
| Supersedes Chains          | このサイクルで発生したSupersedes関係   |
| Basis Note                 | 下記4項目を明記                  |

**Basis Note：**

- 本レポートはProjection Artifactである
- Reporting目的でのみ用いる
- Restore（復元）を目的としない
- Snapshotから逆算してObservationを再構築してはならない
- 後続Cycleの入力として参照できるが、判断根拠にはならない

過去のSnapshotReportは上書き・無効化せず、そのまま履歴として残す。

## 6. ドメイン固有ルール

- **AI出力の扱い（ASG-N3）：** AI出力が流暢・もっともらしいことは、内容が正しい・観測された事実であることの根拠にしない。他の入力と同様、まず原文をObservationとして固定する。
- **RAGの扱い（ASG-B1, O3）：** RAGはStep 3以降にのみ導入する。Step 2の不足判定はInquiry FrameとStep 1入力のみに基づき、RAGを混入させない。RAG返却は個別に出所・クエリ付きでObservation Ledgerへ記録し、統合・要約しない。RAGは権威ではなく追加観測として扱う。
- **不足の導出：** 原則として都度AIが導出する（Origin: Model-derived）。ユーザー側に事前の不足認識がある場合は、他AIによる補完が既になされているとみなす。
- **サイクル：** Step 4完了後、新たなObservationや既存Gapの状態を踏まえ、次のStep 2が開始される。強制的な再評価は行わない。

## 7. ASG要件マッピング（抜粋）

| ASG要件  | 対応箇所                                                                              |
| ------ | --------------------------------------------------------------------------------- |
| O1, O2 | Observation Ledger：原文を加工せず記録。Opinion/Snapshotで根拠の二重管理をしない                         |
| O3, U2 | Undefinedを一時条件として定義し、観測存在＝境界確立とは扱わない                                              |
| I1     | Interpretation（関係づけ・比較）とModel-Derived Contentを区別                                  |
| N2, N3 | AI出力・RAGを「観測された証拠」と混同しない運用ルール。Model-Derived ContentをObservation/Interpretationと区別 |
| E1, E2 | Gap Record / OpinionにおけるBasisの必須化。SnapshotReportは新たな評価を持ち込まない                     |
| U1, U3 | Gap = Evaluation結果、Unknown = 判断材料の欠如、を明確に区別                                       |
| B1     | Open状態のGapを未確立の権威として扱わない、RAGを権威化しない                                               |
| F1, F3 | Gap Statusの不可逆設計＋新規Gap＋Supersedesによる再評価の表現。Step4→Step2のサイクルはF1の具体化                |
