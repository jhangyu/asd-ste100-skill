# Chinese Controlled-Language Rules — Rationale and Examples

This file explains the Chinese rules in SKILL.md. ASD-STE100 has no Chinese edition, and no Chinese controlled-language standard covers agent text. These rules are this skill's own design. They carry over the STE principles that do not depend on English grammar: one meaning per word, one instruction per sentence, an explicit actor, length caps, and hedge preservation.

Examples use Traditional Chinese (zh-TW). The same rules apply to Simplified Chinese.

## What Carries Over from STE, and What Does Not

**Carries over:**
- One instruction per sentence.
- A length cap per sentence.
- No semicolons.
- One term, one meaning.
- Lists for sequences.
- Short noun stacks.
- Active voice when the actor matters.
- Keep every hedge, number, and scope qualifier.
- No marketing adjectives.

**Does not carry over:**
- Phrasal verbs, present perfect, and English nominalization. Chinese has no direct equivalent, so the linter skips these rules in Chinese sentences.
- The ~900-word dictionary. No Chinese dictionary exists for this purpose. The skill applies the principle: pick the plainest common word and use it the same way every time.

## Length Unit and Caps

**Unit.** One Han character counts as 1. One run of Latin letters or digits (a word, an identifier, a number) counts as 1. Punctuation and spaces count as 0. 「刪除 cache.db 檔案」 is 2 + 1 + 2 = 5 units, because `cache.db` is one run.

**Caps.**
- Instructions and procedures: ≤30 characters.
- Descriptions: ≤40 characters. The linter enforces this cap.

**Why these numbers.** English STE caps instructions at 20 words and descriptions at 25 words. In technical text, one English word usually becomes about 1.5 to 1.7 Han characters. 25 words × 1.6 = 40 characters. 20 words × 1.5 = 30 characters. A 40-character sentence also fits on one line in most terminals and chat windows. The linter can only check the 40-character cap, because it cannot tell an instruction from a description.

## Structural Rules

### 1. 一句一指令 — One instruction per sentence

**Rationale.** Chinese joins actions with 「並」「並且」「然後」「再」 or with a plain 「，」. The reader must then decide whether the actions are a sequence, a condition, or alternatives. One sentence per action removes that decision.

- 正例：「停止服務。刪除暫存檔。重新啟動服務。」
- 反例：「停止服務並刪除暫存檔，然後再重新啟動。」

Linter: not checked.

### 2. Sentence length

**Rationale.** A long Chinese sentence often hides several clauses behind commas. Commas do not end a sentence in Chinese, so a 「，」 chain can run for 80 characters with no 「。」.

- 正例：「上游工作完成後，讀取輸出檔。」（13 characters）
- 反例：「在上游工作已經完成並且沒有產生任何錯誤的情況下，下游代理程式應該繼續讀取並處理輸出檔，但要注意逾時可能會產生不完整的檔案。」（over 40 characters）

Linter: hard finding above 40 characters (`long-sentence`).

### 3. 主詞明確 — Explicit actor

**Rationale.** Chinese often drops the subject. A human uses context to restore it. An agent that reads one message in isolation has no context. Drop the subject only when exactly one actor can do the action, as in a direct command to the reader.

- 正例：「主代理程式收到結果後，由測試代理程式刪除暫存檔。」
- 反例：「收到結果後刪除暫存檔。」— who receives the result, and who deletes the file?

Allowed: 「刪除暫存檔。」 as a command to the reader. The reader is the only possible actor.

Linter: not checked.

### 4. No semicolons

**Rationale.** STE Rule 8.1 bans the semicolon. In Chinese, 「；」 joins clauses whose relation is unclear: a list, a contrast, or a sequence. A full stop makes each clause stand alone.

- 正例：「主機 A 已離線。主機 B 正常。」
- 反例：「主機 A 已離線；主機 B 正常。」
- 反例：「主機 A 已離線;主機 B 正常。」（half-width ; in Chinese text）

Linter: hard finding for 「；」 and 「;」 (`semicolon`).

### 5. Lists for sequences and long enumerations

**Rationale.** A 「、」 chain hides how many items it holds. It also hides whether the items are steps, conditions, or alternatives. A numbered list shows the count and the order.

- 正例：
  1. 檢查設定檔。
  2. 檢查日誌。
  3. 檢查快取。
- 反例：「檢查設定檔、日誌、快取、索引、權限、憑證。」

Use a numbered list for 3 or more steps. Use a list for 5 or more items of any kind.

Linter: advisory finding for 5 or more items chained with 「、」 (`enumeration-run-on`).

### 6. 一詞一義 — One term, one meaning

**Rationale.** Chinese has many near-synonyms for the same referent: 「使用者／用戶／客戶」「刪除／移除／清除」「設定／配置／組態」. A reader cannot tell whether two names mean one thing or two. Pick one name and use it every time.

- 正例：「使用者送出請求。系統回覆使用者。」
- 反例：「使用者送出請求。系統回覆用戶。」

Linter: not checked. The English synonym-rotation check does not run on Chinese sentences.

### 7. No stacked noun piles (長串名詞堆疊)

**Rationale.** Chinese allows long noun stacks with no marker between the nouns. The reader cannot see which noun modifies which. Keep a stack to 3 nouns or fewer. Unpack the rest with 「的」 or a clause.

- 正例：「處理器會設定任務佇列的優先順序。」
- 反例：「代理任務佇列優先順序處理器設定檔」

Linter: not checked. This needs word segmentation, which the stdlib cannot do.

### 8. Keep hedges

**Rationale.** 「可能」「也許」「應該」「大概」 carry the author's confidence. Confidence is content. A shorter sentence that drops 「可能」 makes a different claim. This is the same rule as the English modality rule. When the length rule and the hedge rule conflict, the hedge rule wins.

- 正例：「請求可能已失敗。」→「請求可能已失敗。」（unchanged）
- 反例：「請求可能已失敗。」→「請求已失敗。」
- 反例：「也許是逾時造成的。」→「原因是逾時。」

Note: 「應該」 has two meanings. It can be a hedge (「服務應該已重新啟動」= probably restarted) or an obligation (「你應該重新啟動服務」= you should restart it). Keep the original reading. If the reading is unclear, keep 「應該」 and add a `Kept as-is:` line.

Linter: never flagged. `--selftest` asserts that 「可能已失敗」 passes clean.

### 9. Avoid 被-passive when the actor matters

**Rationale.** 「被」 lets the writer leave out the actor. For an agent, the actor is often the most important fact.

- 正例：「排程器刪除了這個工作。」
- 反例：「這個工作被刪除了。」

Allowed when the actor is unknown or irrelevant: 「檔案被鎖定，無法寫入。」

Linter: not checked.

### 10. Keep measure words and scope qualifiers

**Rationale.** 「只」「全部」「每個」「至少」, numbers, and measure words set the scope of an action. Dropping them to save characters changes the action.

- 正例：「只刪除 3 個過期的快取檔。」
- 反例：「刪除快取。」

If keeping the qualifier breaks the length cap, split the sentence. Do not drop the qualifier.

Linter: not checked.

### 11. No marketing adjectives

**Rationale.** These words claim quality instead of showing it. They carry no fact an agent can act on.

- 正例：「這個工具在 2 秒內完成同步。」
- 反例：「這個工具提供無縫、極速的一站式同步。」

Words: 無縫、強大、極致、全方位、一站式、頂級、革命性、顛覆性、業界領先、極速 (and their Simplified forms).

Linter: hard finding (`marketing-adjective`).

## Mixed Chinese and English

- Classify each sentence on its own. A sentence that contains a Han character is a Chinese sentence.
- Keep English identifiers, commands, and terms in English inside a Chinese sentence. 「執行 `npm test`。」 stays as it is.
- Do not translate a segment unless the user asks for a translation.
- The linter follows the same split. English rules run on English sentences. Chinese rules run on Chinese sentences, on the same line.

## Sources

This ruleset has no official source. It adapts the ASD-STE100 rule categories summarized in `references/writing-rules.md`. The character-per-word ratio is a rule of thumb for technical text, not a measured constant.
