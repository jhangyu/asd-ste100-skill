# Before / After Examples

## Part 1 — Official STE Examples

These illustrate real ASD-STE100 rules, drawn from public secondary sources (see `references/writing-rules.md`). They are paraphrased illustrations of the rule, not quotes from the standard itself.

| Rule | Before | After | Why |
|---|---|---|---|
| One meaning per word | "Verify the system." / "Check the connections." / "Confirm receipt." | "Make sure the system is correct." (one approved term used consistently) | Three near-synonyms force the reader to guess whether they mean the same action. |
| One part of speech per word | "Oil the valve." | "Apply oil to the valve." | If "oil" is approved only as a noun, using it as a verb breaks the one-word-one-role guarantee. |
| Precise verb meaning | "Follow the safety instructions." | "Obey the safety instructions." | "Follow" can mean "come after" or "obey" — STE picks the unambiguous one. |
| Simple tense only | "We have received the technical reports from HQ." | "We received the technical reports from HQ." | Present perfect adds a second parse ("received, and still relevant now?") that simple past avoids. |
| Verb, not noun | "Perform an inspection of the filter." | "Inspect the filter." | The noun form hides the action and adds a filler verb that carries no meaning. |
| No phrasal verbs | "Take off the access panel." | "Remove the access panel." | "Take off" also means "depart" and "deduct" — the two words together do not predict the meaning. |

## Part 2 — Applied to Agent Output

These are original examples built for this skill's actual use case: rewriting AI agent output so another agent, a translation layer, or a non-native reader can parse it without ambiguity. They are illustrations, not quotes from any real system.

Word counts below are whitespace-separated tokens (`text.split()`), punctuation not counted separately. A different tokenizer will produce a different number.

### Example A — Tool description

**Before:**
> This tool will attempt to synchronize state across the various backends that have been configured, and if a conflict is detected it may resolve it automatically depending on the strategy that has been set, or otherwise it will surface the conflict for manual review.

**Violations flagged:**
- Two instructions in one sentence (sync + resolve/surface).
- Present perfect in the relative clauses ("have been configured", "has been set").
- 44 words, far over the 25-word descriptive cap.

Note what is *not* flagged: "will attempt to" and "may resolve". Those are hedges, not violations. The tool is not promised to succeed, and the rewrite must not promise it either.

**After:**
> The tool tries to synchronize state across the configured backends. If it finds a conflict, it reads the configured strategy. If the strategy allows automatic resolution, the tool may resolve the conflict without a user. If the tool does not resolve the conflict, it reports the conflict for manual review.

The last sentence branches on whether the conflict was resolved, not on what the strategy allows. That is what "or otherwise" meant in the original: the fallback covers a permitted resolution that still did not happen.

### Example B — Error message

**Before:**
> An error may have occurred while processing your request due to a possible mismatch in the expected data format, which could be caused by an outdated client version.

**Violations flagged:**
- One sentence carrying three separate claims (an error occurred, a format mismatch, a client version).
- 28 words, over the descriptive cap.

Not flagged: "may have occurred" and "could be caused by". The message is written by a system that does not know what went wrong. Both hedges are accurate reporting of that ignorance.

**After:**
> Your request may have failed. The cause may be a data format that does not match what the server expects. An outdated client can cause this mismatch. Check your client version.

**This example is the reason the modality rule exists.** An earlier version of this file rewrote the first sentence as "The request failed", and the third as "an outdated client **is the most common cause**". Both read better. Both are wrong: the first asserts a failure the system only suspects, and the second invents a frequency claim that appears nowhere in the input. A rewrite that supplies a cause, a frequency, or a mechanism has stopped being a rewrite.

Note also that "may have failed" keeps a compound verb form that the simple-tense rule would otherwise remove. **When the tense rule and the modality rule conflict, modality wins** — dropping the auxiliary here would delete the uncertainty along with the tense.

### Example C — Inter-agent instruction

**Before:**
> Once the upstream job has completed and assuming no errors were raised, the downstream agent should proceed to consume the output artifact, though it is worth noting that partial artifacts are sometimes produced under timeout conditions.

**Violations flagged:**
- Present perfect ("has completed") and subordinate-clause stacking ("assuming...", "though it is worth noting...").
- One sentence, three separate facts (completion condition, next action, edge-case warning).
- 36 words, over the 20-word instruction cap.

**After:**
> Wait for the upstream job to finish with no errors. Then read the output artifact. Warning: a timeout can produce a partial artifact. Check that the artifact is complete before you use it.

Two deliberate calls worth stating rather than hiding:
- "should proceed to consume" became the imperative "read". STE permits this for instructions, where a recommendation addressed to the executing agent is a command. Do not make the same move in descriptive text.
- The final sentence is **new**. The original warned about partial artifacts without saying what to do about it. Adding the check makes the warning actionable, but it is added content, so it is called out here rather than passed off as a rewrite. If the source's silence was deliberate, drop the sentence.

### Example D — README prose (STE-flavored mode)

**Before:**
> Our caching layer is designed to slot seamlessly into your existing stack with minimal friction and no vendor lock-in; it leverages semantic similarity to dramatically reduce the cache misses that traditionally plague LLM workloads.

**Violations flagged:**
- Marketing adjectives and claims without measurement ("seamlessly", "minimal friction", "dramatically").
- Semicolon joining two separate ideas.
- Nominalization and soft phrasing ("is designed to slot into", "leverages").
- 34 words, over the 25-word descriptive cap.

**After:**
> A normal cache matches requests by exact text, so a small change in wording causes a cache miss. This cache compares the meaning of a new prompt against the prompts it already holds. It runs alongside your current stack and stores no data outside it.

Flavored mode kept the explanatory rhythm and did not force one fixed term per concept. It still cut the marketing adjectives, the semicolon, and the length.

## Part 3 — Chinese (中文) Examples

These examples apply the Chinese rules from `references/chinese-rules.md`. The rewrite stays in Chinese. English identifiers stay in English. Length counts use the Chinese length unit: one Han character or one Latin run counts as 1, punctuation counts as 0.

### Example E — Error message (錯誤訊息)

**Before:**
> 由於系統偵測到您的設定檔格式可能與目前版本不相容，因此本次的同步作業可能沒有被完整執行，建議您檢查設定檔並重新嘗試同步。

**Violations flagged:**
- Three claims in one sentence: a format mismatch, an incomplete sync, and a recommended action.
- 57 characters, over the 40-character cap.
- 被-passive hides who did not complete the sync.

Not flagged: both 「可能」. The system does not know the cause. The rewrite keeps both hedges.

**After:**
> 設定檔格式可能與目前版本不相容。同步作業可能沒有完成。請檢查設定檔，再執行一次同步。

### Example F — Tool description (工具說明)

**Before:**
> 本工具提供無縫的一站式資料同步體驗，可以自動偵測、比對、合併、備份、還原各個後端的資料；若發生衝突則依照設定進行處理。

**Violations flagged:**
- Marketing adjectives (無縫、一站式).
- Five actions chained with 「、」.
- Semicolon 「；」.
- No actor in the conflict clause: who handles the conflict?

**After:**
> 本工具同步各個後端的資料。工具執行以下動作：
> 1. 偵測資料變更。
> 2. 比對各個後端的資料。
> 3. 合併資料。
> 4. 備份資料。
> 5. 在需要時還原資料。
>
> 如果發生衝突，工具依照衝突策略設定處理衝突。

### Example G — Inter-agent instruction (代理間指令)

**Before:**
> 等上游跑完沒錯的話就接著處理產出檔，不過逾時的時候有時候會產出不完整的檔案要注意。

**Violations flagged:**
- No actor: who waits, and who processes the output file?
- Three facts in one sentence: a condition, an action, and a warning.
- 39 characters, over the 30-character instruction cap. The linter does not flag it, because it checks only the 40-character description cap.

Not flagged: 「有時候」. The rewrite keeps the frequency hedge.

**After:**
> 下游代理程式等待上游工作完成，且確認沒有錯誤。然後，下游代理程式讀取產出檔。警告：逾時有時候會產生不完整的產出檔。

### Example H — Status report (狀態回報)

**Before:**
> 部署已大致完成，使用者端看起來正常，但部分用戶回報登入有問題，客戶服務那邊也收到幾件類似的反映。

**Violations flagged:**
- Synonym rotation: 「使用者」「用戶」 name the same referent. 「客戶服務」 is a different team and stays.
- Four facts in one sentence, 44 characters.

Not flagged: 「大致」「看起來」「部分」「幾件」. These carry the reporter's confidence and the scope. The rewrite keeps all of them.

**After:**
> 部署大致完成。使用者端看起來正常。部分使用者回報無法登入。客戶服務也收到幾件類似的回報。

### Example I — Hedged statement (必須保留推測語氣)

**Before:**
> 這個錯誤應該是快取過期造成的，也許清除快取後就會恢復正常，不過也可能是網路問題。

**Violations flagged:**
- Three claims in one sentence.

Not flagged: 「應該」「也許」「可能」. All three are hedges. The writer does not know the cause.

**After:**
> 這個錯誤應該是快取過期造成的。清除快取後，系統也許會恢復正常。原因也可能是網路問題。

**Wrong rewrite (do not do this):**
> 這個錯誤是快取過期造成的。清除快取即可恢復正常。

This version reads better and is shorter. It is wrong: it turns a guess into a fact, and it deletes the second possible cause. The same rule applies as in Example B: a rewrite that removes a hedge has stopped being a rewrite.

## How to Read These Examples

Part 1 shows the actual rules this skill is built on. Part 2 shows the transfer: the same discipline — one meaning per word, active voice, simple tense, one instruction per sentence, explicit conditions instead of buried subordinate clauses — makes machine-to-machine and cross-language text safer to parse, not just aircraft manuals.
