## Phase 1

I'm building a new 'Daily Practice' feature for my language app. The system currently allows users to chat with an AI partner, polish their messages, and save expressions. This new feature will use an SRS mechanism to pick a target word and fetch scenario-based exercises from an LLM. I'll need to handle JSON data storage for these scenarios and implement a 'Word Bank' bonus system. Here is the breakdown of the feature, user interactions, and UI logic.

### **Phase 1 Brief: The "Active Production" Feature**

#### **1. Purpose & Logic (The Backend)**

- **The SRS Trigger:** The system identifies a "due" word using an SRS (Spaced Repetition System) algorithm.
- **Dynamic Scenarios:** Each word in the database has a `scenarios` JSON field. <span style="color: red">(TODO: This field is not yet included in the current Model.)</span>
    - **Logic:** If `scenarios` is empty $\rightarrow$ call LLM to generate no more than 3 applicable scenarios $\rightarrow$ store in DB $\rightarrow$ render on page.
    - **Logic:** If `scenarios` exists $\rightarrow$ load directly to the "Scenario Context" card.
        
- **The Word Bank:** Randomly fetch 3 words from the entire user database to encourage "interleaving" (mixing different topics).
    

#### **2. User Capabilities (The Gameplay)**

- **Scenario Selection:** User reads the generated scenarios and chooses one to respond to.
    
- **Sentence Construction:** User writes an original sentence in the input area.
    
- **Bonus Logic:** If the user's sentence includes a word from the "Word Bank," they receive a "Bonus" tag or extra points upon submission.
    

#### **3. UI Interactions (The "Vibe")**

- **Visual Anchor:** The target SRS word is prominently displayed (e.g., "cookie-cutter").
    
- **Real-time Matching:** (Optional but cool) As the user types, words from the **Word Bank** could highlight or "check off" if detected in the input string.
    
- **Submission:** On clicking "Submit," the system validates the sentence and updates the SRS metadata (e.g., last reviewed date, ease factor).

## Phase 3
### 1. New Database Model

We need one new model to track the "Daily Goal" and "Focus Time."

- **`DailyPracticeLog`**:
    ```Python
    class DailyPracticeLog(models.Model):
	    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
	    date = models.DateField(default=timezone.now)
	    words_practiced = models.IntegerField(default=0)
	    
	    # Store the 3 WordNode IDs selected for today: [12, 45, 89]
	    word_ids = models.JSONField(default=list) 
	    
	    is_completed = models.BooleanField(default=False)
	    created_at = models.DateTimeField(auto_now_add=True)
	    updated_at = models.DateTimeField(auto_now=True)
	    
	    class Meta: constraints = [ models.UniqueConstraint(fields=['user', 'date'], name='unique_daily_log') ]
    ```

`words_practiced` increments after each successful verify response, regardless of pass/fail.
#### The Logic Flow
1. **User enters Daily Phrases:**
    - Backend checks if a `DailyPracticeLog` exists for today.
    - If **No**: Pick 3 random `WordNode` IDs, save them in `word_ids`, and return them.
    - If **Yes**: Return the existing `word_ids` and the `words_practiced` count.
2. **User finishes Word 1 & 2:**
    - Frontend calls an update endpoint. Backend increments `words_practiced`.
3. **User finishes Word 3:**
    - Backend sets `is_completed = True`. `updated_at` automatically snaps to the current time.

### 2. Backend Logic (Django)

We need to add a few paths to your `urls.py` and create corresponding views:
- **`GET /daily-phrases/init/`**:
    - Check `DailyPracticeLog` for today. If `is_completed`, return the "Done" state.
    - If `word_ids` is empty, query `WordNode.objects.filter(user=user, next_review_at__lte=now).order_by('?')[:3]` (with `box_level=1` as fallback) and save these IDs to `DailyPracticeLog`. 
    - **Batch Scenario Generation:** Identify all selected words missing an `explanation`. If any, trigger a **single** LLM call to generate scenarios for all missing words in one response (JSON format). Parse the response and bulk-update the corresponding `WordNode` records. Return the finalized data.
- **`POST /daily-phrases/verify/`**:
    - The endpoint that receives the user's sentence.
    - **Logic**: Call LLM -> Get Polished Text + Pass/Fail.
    - **Side Effect**:
        - **Fail:** `box_level = 1` and `next_review_at = timezone.now()` (immediately back to the review queue);
        - **Pass:** `box_level += 1` and `next_review_at = timezone.now() + timedelta(days=2**box_level)` (exponentially increase review interval);
        - Increment `words_practiced` in `DailyPracticeLog`. 
        - Update `updated_at`.
        - If words_practiced >= DAILY_PRACTICE_LIMIT, set is_completed = True.
        

### 3. Frontend Logic (Vue 3)
1. **State Management**
	Use a `ref` or `reactive` object to track `currentWordIndex` (0, 1, 2) and `sessionTimer`. Store the fetched `word_ids` and their corresponding `WordNode` data in a local array `sessionWords`.

**B. **Primary Button Logic (Dynamic Toggle)**
1. **Stage: `COMPOSING`**
    - **Label:** "Submit My Masterpiece"
    - **Action:** Calls `POST /api/v1/daily-phrases/verify/`. On success, set `uiStage = 'REVIEWING'`.
2. **Stage: `REVIEWING`**
    - **If `currentWordIndex < 2`**:
        - **Label:** "Continue to Next Phrase"
        - **Action:** Increment `currentWordIndex`, reset input field, and set `uiStage = 'COMPOSING'`.
    - **If `currentWordIndex === 2`**:
        - Text:** "🏆 Congratulations! You’ve completed your daily goal!"

3. Word Card Imagery & Theme
	- **Main Card Visual:** The frontend must render one unified context-specific picture (or background) for the entire Word Card. Use Vue's dynamic binding to derive the image asset filename strictly from the **first scenario's tag** (`sessionWords[currentWordIndex].scenarios[0].tag`).
	- **Example:** If the first scenario's tag is `"Office"`, the card loads `/assets/scenarios/office.png` as its primary visual theme, setting the vibe for all three scenarios of that word.
	- **Fallback Strategy:** Implement a default "General" imagery fallback in case the returned tag is unknown, missing, or fails to load.

4. **The "Done" View:** Create a dedicated component for the **"Rocket"** completion screen. It should only pull summary data (e.g., `words_practiced`, `total_time`) from the `DailyPracticeLog`. This view should be shown automatically if `GET /daily-phrases/init/` returns `is_completed: true`.

**5. Constants & Configuration**
- **Daily Limit Variable:** Extract the daily practice limit into a configuration constant (e.g., `export const DAILY_PRACTICE_LIMIT = 3;`). Do not hardcode the number `3` in any state management, conditionals, or loops.
    

**6. Reusable Text Diff Component**
- **Component Extraction:** Create a reusable Vue component named `TextDiffViewer.vue` (referencing the core logic from our temporary `DiffTrial.vue` file) to display the Original vs. AI Polished text.
- **Dynamic Props:** The component must accept at least two props to handle different contexts:
    - `theme`: Accepts `'light'` or `'dark'`.
    - `enableActions`: Boolean to toggle the Accept/Reject buttons.
- **Implementation Contexts:**
    - **Daily Phrases View:** Implement as `<TextDiffViewer theme="light" :enableActions="false" />`.
    - **Chat Mode View:** Implement as `<TextDiffViewer theme="dark" :enableActions="true" />`.
#### System Prompt for Batch Scenario Generation

> **System Prompt Update:** Act as an expert linguist and language tutor. I will provide a JSON array of vocabulary words or phrases. For each item, generate exactly 3 distinct, practical, real-world contexts where this word is naturally used.

**Constraints:**
- Keep each scenario description brief (maximum 15 words).
- Do not include the definition of the word.
- **Tagging:** You MUST assign one context tag to each scenario from the following exact list: `["Office", "Meeting", "Interview", "Remote_Work", "Cafe", "Restaurant", "Party", "Street", "Home", "Shopping", "Gym", "Commute", "Airport", "Hospital", "Hotel", "Texting", "Phone_Call"]`. Do not invent new tags.
- **Output Format:** Respond ONLY with a valid JSON object.

**Example Output:** `{"pull request": [{"description": "A developer asking for a code review.", "tag": "Office"}, {"description": "Discussing code integration.", "tag": "Meeting"}, {"description": "Sending a message about branch merges.", "tag": "Texting"}]}`

### 🎯 核心场景 Tag 库 (建议 15-20 个足矣)

**👔 职场与商务 (Professional & Work)**
- `Office` (日常办公 / 同事交流)
- `Meeting` (会议室 / 正式讨论)
- `Interview` (求职面试)
- `Remote_Work` (视频会议 / Slack 聊天)

**☕️ 休闲与社交 (Social & Casual)**
- `Cafe` (咖啡馆 / 轻松交谈)
- `Restaurant` (餐厅 / 点餐 / 饭局)
- `Party` (派对 / 聚会)
- `Street` (街头偶遇 / 问路 / 户外)

**🏠 日常生活 (Daily Life)**
- `Home` (家里 / 亲情 / 独处)
- `Shopping` (超市 / 商场 / 购物)
- `Gym` (健身房 / 运动)
- `Commute` (地铁 / 公交 / 通勤路上)

**✈️ 旅行与特定场所 (Travel & Specific)**
- `Airport` (机场 / 海关 / 登机)
- `Hospital` (医院 / 看病 / 药店)
- `Hotel` (酒店 / 前台办理)

**📱 抽象通讯 (Digital)** _(这类场景可以放一个手机聊天的背景板)_
- `Texting` (发短信 / 微信 / WhatsApp)
- `Phone_Call` (打电话)

## API Contract: Daily Phrases

### 1. Initialize Daily Session

**Endpoint:** `GET /api/v1/daily-phrases/init/` 
**Purpose:** Fetches today's practice words or returns the completed status. Triggers batch scenario generation on the backend if scenarios are missing.

**Response Scenario A: Practice Pending (200 OK)**
_Frontend UI mapping: Initializes `sessionWords` array and sets `currentWordIndex`._
- `words_practiced`: 已完成 `/verify/` 提交的次数（0, 1, 2）。
- `is_completed`: 业务逻辑是否全部结束。

```JSON
{
  "is_completed": false,
  "words_practiced": 0,
  "session_words": [
    {
      "id": 102,
      "word": "pull request",
      "scenarios": [
        {
          "description": "A developer asking a senior engineer for a code review.",
          "tag": "Office"
        },
        {
          "description": "Merging a new feature branch into the main repository.",
          "tag": "Remote_Work"
        },
        {
          "description": "Resolving a merge conflict before a deployment sprint.",
          "tag": "Meeting"
        }
      ],
      "bonus_words": [ {"id": 201, "word": "merge"}, {"id": 205, "word": "commit"}, {"id": 210, "word": "review"} ]
    },
    {
      "id": 45,
      "word": "mitigate",
      "scenarios": [
        {
          "description": "A project manager discussing risk reduction strategies.",
          "tag": "Meeting"
        },
        {
          "description": "An architect designing a system to handle high traffic.",
          "tag": "Office"
        },
        {
          "description": "A doctor explaining ways to lessen medication side effects.",
          "tag": "Hospital"
        }
      ],
      "bonus_words": [ {"id": 201, "word": "merge"}, {"id": 205, "word": "commit"}, {"id": 210, "word": "review"} ]
    },
    {
      "id": 89,
      "word": "bandwidth",
      "scenarios": [
        {
          "description": "Telling a colleague you don't have time for a new project.",
          "tag": "Texting"
        },
        {
          "description": "Discussing internet speed limits with an ISP.",
          "tag": "Phone_Call"
        },
        {
          "description": "A manager assessing team capacity for the upcoming sprint.",
          "tag": "Meeting"
        }
      ],
      "bonus_words": [ {"id": 201, "word": "merge"}, {"id": 205, "word": "commit"}, {"id": 210, "word": "review"} ]
    }
  ]
}
```

**Response Scenario B: Already Completed Today (200 OK)** _Frontend UI mapping: Directly routes to the "Done / Rocket" view._

```JSON
{
	"is_completed": true,
	"words_practiced": 3,
	"completed_at": "2026-03-26T10:30:00Z",
	"summary": {
		"focus_minutes": 15,
		"daily_streak_percent": 100
	}
}
```
### **Refresh (`/refresh-bonus/`)
- **Endpoint:** `GET /api/v1/daily-phrases/refresh-bonus/?exclude_ids=102,45,89`
- **Logic:**
	- **SQL Query:** Randomly fetch 3 words where `user == request.user` and `box_level == 1`.
	- **Encapsulation (DRY):** This randomization and filtering logic must be centralized in a shared backend utility to be reused by both the `/init/` and `/refresh-bonus/` endpoints.
	```json
	{ "bonus_words": [
		{ "id": 301, "word": "refactor" },
		{ "id": 302, "word": "deploy" },
		{ "id": 303, "word": "debug" }
		]
	}
	```

**Frontend Implementation:**
- Upon a successful `200 OK` response, the frontend must immediately update the `bonus_words` array for the current active word to trigger a UI refresh.
---

#### 2. Verify Sentence & Update Progress

- **Endpoint:** `POST /api/v1/daily-phrases/verify/` 
- **Purpose:** Submits the user's sentence to the LLM, updates the SRS logic (`box_level`, `next_review_at`), and advances the daily progress.

**Request Payload:**
```JSON
{
  "word_id": 102,
  "user_sentence": "I opened a pull request to merge my latest commit.",
  "active_bonus_words": [
    {"id": 201, "word": "merge"},
    {"id": 205, "word": "commit"},
    {"id": 210, "word": "review"}
  ]
}
```

**Response Payload (200 OK):** 
_Frontend UI mapping: Feeds data into the `<TextDiffViewer>` and updates the "Continue / Complete" button state._
```json
{
  "verification": {
    "is_pass": true,
    "polished_text": "...",
    "feedback": "...",
    "mastered_word_ids": [201, 205] 
  },
  "session_progress": {
    "words_practiced": 1,
    "is_completed": false 
  }
}
```

**The LLM Prompt**

> _"Check the user_sentence for the target_word AND any provided bonus_words. Determine if each was used correctly and naturally. Return a list of mastered_word_ids.""_

**Backend Update Logic:** 
- **Target Word:** If `is_pass` is true, `target_word.box_level += 1`. 
- **Bonus Words:** For each ID in the LLM's `mastered_word_ids`: 
	- Security Check (must belong to user). 
	- SRS Boost: `WordNode.objects.filter(id=bonus_id).update(box_level=F('box_level') + 1, next_review_at=...)`. 
	- **Response Payload (200 OK):**

```JSON
{
  "verification": {
    "is_pass": false,
    "polished_text": "I submitted a pull request to your repository.",
    "feedback": "In this context, 'pull request' is a noun phrase. You need a verb like 'submitted' or 'opened' before it.",
    "mastered_word_ids": [201, 210]
  },
  "session_progress": {
    "words_practiced": 1,
    "is_completed": false
  }
}
```

_(Note: `session_progress.is_completed` returns `true` on the 3rd word.

### 3. Frontend Navigation & State Management
To handle the transition between **submitting a sentence** and **moving to the next word**, the frontend must maintain a local state machine.

#### A. State Definitions
- **`currentWordIndex`**: (0, 1, or 2) Tracks progress through the `session_words` array.
- **`uiStage`**: `['COMPOSING', 'SUBMITTING', 'REVIEWING']`
    - **`COMPOSING`**: Default state. Input is active; Word Bank is refreshable.
    - `SUBMITTING`: 全局加载动画/Skeleton，输入框锁定
    - **`REVIEWING`**: Triggered after `/verify/` returns. Input is read-only; Text Diff and Feedback are visible.
#### B. Primary Button Logic (Dynamic Toggle)
The main action button handles two distinct operations based on the `uiStage`:
1. **When `uiStage == 'COMPOSING'`:**
    - **Label:** "Submit My Masterpiece"
    - **Action:** Triggers `POST /api/v1/daily-phrases/verify/`.
    - **Success Callback:** Store the `verification` data and switch `uiStage` to `'REVIEWING'`.
2. **When `uiStage == 'REVIEWING'`:**
    - **Label:** "Continue" (if `currentWordIndex == 2`, 不再显示按钮，显示纯文本：🏆 Congratulations! You’ve completed your daily goal!).
    - **Action:** * If `currentWordIndex < 2`: Increment `currentWordIndex`, reset the input field, and set `uiStage` back to `'COMPOSING'`.

#### C. Interaction Constraints
- **Word Bank Refresh (🔄):** Enabled only when `uiStage == 'COMPOSING'`.
- **Input Field:** Editable when `uiStage == 'COMPOSING'`; Locked/Read-only when `uiStage == 'REVIEWING'`.

---
### 4. Technical Shared Logic (DRY Requirement)
The backend **must encapsulate** the logic for selecting **Random `box_level=1` Bonus Words** into a single reusable utility. This ensures that:
1. The **`/init/`** endpoint (during session startup).
2. The **`/refresh-bonus/`** endpoint (during manual UI refresh).
...both consistently pull 3 random words from the user's own `box_level=1` pool while excluding all `session_word_ids` provided in the request to ensure zero overlap between targets and bonus words.





### Direct Phrase Refactoring (Refactor the Input)

**Target Phrase:** {{Input_Phrase}}

1. **Refactor the Input:** Provide 0-3 natural, high-leverage alternatives specifically for the phrase above. 
2. **Dynamic "Vibe" Labeling:** Instead of fixed slots, label each alternative with its specific context (e.g., "Sharp & Critical," "Conversational," "Business/Tech," or "Philosophical").
3. **Requirement for Each Alternative:**
   * **Refined Expression:** The alternative phrase/idiom.
   * **Real-world Example:** One clear sentence showing it in action.
**Graceful Silence:** If the provided phrase is already the most natural option, or if alternatives feel forced, do not generate redundant data.