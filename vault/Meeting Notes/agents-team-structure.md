# Agents — מבנה הצוות

## Overview
תיקיית `.claude/agents/` מכילה הגדרות של חברי הצוות של ראובן: **יעל** (כתיבה), **יובל** (עיצוב), **חן** (מחקר). כל סוכן ב-Claude Code מוגדר כקובץ `.md` עם frontmatter (תפקיד, כלים זמינים, מודל) וגוף שמסביר את ההתמחות וההוראות. נכון ל-2026-05-13 **יעל ויובל קיימים ופועלים**: יעל עם 5 כלים מוגבלים (Read, Write, Edit, Glob, Grep) ושכתוב מ-`Content/` ל-`Output/`; יובל עם 4 כלים (Read, Write, Bash, Glob) ויצירת תמונות דרך OpenAI Images API (`gpt-image-2`) עם הסקיל `gpt-image-gen`. יעל מסמנת מקומות לתמונות עם placeholder `{{IMAGE_NEEDED: "..."}}` שראובן ממיר לקריאות יובל ומשלב חזרה לקבצי MD/HTML. חן עדיין לא הוגדרה.

## קבצים שמתועדים כאן

| קובץ | מה צריך להיות | למי משויך | סטטוס |
|---|---|---|---|
| `/.claude/agents/yael.md` | סוכנת כתיבת תוכן — ניסוח, סגנון, טקסט | יעל | **קיים (Opus, 5 כלים) · נבחנה end-to-end ב-2026-05-13** |
| `/.claude/agents/yuval.md` | סוכן עיצוב תמונות — חזותי, וויזואלי | יובל | **קיים (Opus, 4 כלים) · 2026-05-13** |
| `/.claude/agents/chen.md` | סוכנת מחקר — איסוף מידע, מקורות, עובדות | חן | **לא קיים** |

## תשתית נלווית של יעל

| נתיב | מטרה |
|---|---|
| `yael/style-guide.md` | מדריך סגנון הכתיבה — כרגע stub, ימולא ע"י המשתמש |
| `yael/reference/` | דוגמאות חיות לסגנון הבית — כרגע ריק (.gitkeep) |
| `Content/` | מאמרי גלם שממתינים לשכתוב |
| `Output/` | תוצרים משוכתבים (`.md` + `.html` לכל מאמר) |

## תשתית נלווית של יובל

| נתיב | מטרה |
|---|---|
| `yuval/reference/` | תמונות השראה לסגנון בית — כרגע ריק (.gitkeep), ממולא ע"י המשתמש |
| `yuval/outputs/` | תמונות מוגמרות (PNG + sidecar .txt עם ה-prompt) — gitignored פרט ל-.gitkeep |
| `.claude/skills/gpt-image-gen/SKILL.md` | מעטפת לקריאת OpenAI Images API עם המודל `gpt-image-2` (curl + jq + python fallback) |
| `OPENAI_API_KEY` ב-`.env` | נדרש להפעלת הסקיל; placeholder נוסף ל-`.env.example` |

## מבנה צפוי של קובץ סוכן

לפי הקונבנציה של Claude Code, כל קובץ ייראה בערך כך:

```markdown
---
name: yael
description: כותבת תוכן — אחראית על ניסוח, סגנון וטקסט
tools: Read, Edit, Write, WebSearch
model: sonnet
---

את יעל, כותבת התוכן בצוות של ראובן. כשמגיע אלייך משימה, את אחראית על...
```

## Open Questions
- מה הפרוטוקול להעברת מידע בין סוכנים — דרך ראובן בלבד, או ישיר? (כרגע: דרך ראובן בלבד — yael→placeholders→Reuven→yuval→MD/HTML.)
- האם להוסיף סוכן רביעי? (למשל "אדם" ל-QA, או "מאיה" לאסטרטגיה?)
- מתי יוגדר `chen.md`? אילו כלים יקבל?
- מי ממלא את `yael/style-guide.md` ואת `yael/reference/`? בלעדיהם יעל פועלת לפי שיקול דעת כללי. לאחר ה-run הראשון יש שלוש הנחיות מומלצות לעיגון — ראה [[crm-article-rewrite]].
- מי ימלא את `yuval/reference/`? עד שזה ריק, יובל מציין במפורש "reference ריק — סגנון חופשי" ופועל לפי שיקול דעת חזותי כללי.
- ה-pipeline end-to-end של "מאמר עם תמונות" עדיין לא נבחן על מקרה אמיתי — נדרש run אחד שיוודא שהשילוב של placeholders→yuval→MD/HTML עובד בפועל.

## Session Log

### 2026-05-13 — מיפוי מצב התיקייה [shipped]
- **What was done:** תועד שהתיקייה קיימת אבל ריקה. הוגדרה הציפייה לשלושה קבצי סוכנים על בסיס `CLAUDE.md`.
- **Decisions:** לא יצרתי stubs ריקים — חיכיתי שהמשתמש יחליט על המבנה והכלים של כל סוכן.
- **Notes / Caveats:** ברגע שיתווספו קבצי סוכנים — לעדכן את הטבלה כאן ולהוסיף Session Log entry.
- **Related:** [[claude-md-instructions]], [[project-overview]], [[custom-commands]]

### 2026-05-13 — יצירת הסוכן יעל [shipped]
- **What was done:** נוצר `.claude/agents/yael.md` (flat, frontmatter + גוף בעברית) עם 5 כלים (Read, Write, Edit, Glob, Grep) ומודל Opus. נוצרה תשתית נלווית: `yael/style-guide.md` (stub), `yael/reference/.gitkeep`, `Content/.gitkeep`, `Output/.gitkeep`. `CLAUDE.md` עודכן בשורה מורחבת ליעל ובסעיף חדש "## ניתוב" עם trigger keywords עברית+אנגלית (שכתב/ערוך/נסח/תרגם/סכם/מאמר/תוכן/פוסט · rewrite/edit/rephrase/translate/summarize/article/content/post).
- **Decisions:** Opus נבחר ע"י המשתמש על פני Sonnet כדי לוודא איכות שכתוב מקסימלית. כלים מוגבלים בכוונה — אין Bash/WebSearch/API/Agent — כדי לסמן בבירור שיעל היא כלי כתיבה טהור; מחקר ועיצוב נשארים לחן וליובל. תוצר כפול (`.md` + `.html`) דורש מיעל לייצר HTML inline עם RTL/LTR מותאם וטיפוגרפיה רזה (Heebo / system-ui).
- **Notes / Caveats:** style-guide ו-reference עדיין ריקים — יעל תציין זאת בפתיח של תשובות עד שהמשתמש ימלא אותם. כלי `Glob` נכלל כדי שיעל תוכל לסרוק את `yael/reference/**/*` אוטומטית. הסוכן עוד לא נבחן end-to-end על מאמר אמיתי.
- **Related:** [[claude-md-instructions]], [[project-overview]], [[custom-commands]]

### 2026-05-13 — יעל נבחנה end-to-end על מאמר CRM [shipped]
- **What was done:** ה-run הראשון של יעל על מאמר אמיתי — `Content/מאמר לדוגמא.txt` שוכתב לסגנון בית, וייוצרו `Output/מאמר לדוגמא.md` ו-`Output/מאמר לדוגמא.html`. ה-flow עבד מקצה לקצה: routing דרך CLAUDE.md → plan mode → AskUserQuestion להחלטות עריכה → Agent(yael) → verification → vault update.
- **Decisions:** הוספה לטבלת ה-status שיעל **נבחנה end-to-end** (לא רק "קיימת"). שלוש הנחיות עריכה שיעל המליצה לעגן ב-style-guide נרשמו ב-[[crm-article-rewrite]] לתיעוד עתידי.
- **Notes / Caveats:** ההפעלה דרך subagent עבדה כצפוי, כולל ההתנהגות של "ציון בפתיח שה-style-guide ריק". יעל לא עידכנה את ה-vault בעצמה — ראובן עידכן בסיום הסשן.
- **Related:** [[crm-article-rewrite]], [[claude-md-instructions]], [[project-overview]]

### 2026-05-13 — יצירת הסוכן יובל + סקיל gpt-image-gen [shipped]
- **What was done:** נוצר `.claude/agents/yuval.md` (flat, frontmatter+body, model Opus, 4 כלים: Read/Write/Bash/Glob) עם workflow בן 7 צעדים — סריקת `yuval/reference/`, חילוץ סגנון, ניסוח prompt משולב באנגלית, קריאה ל-OpenAI Images API (`gpt-image-2`), שמירה ל-`yuval/outputs/<YYYY-MM-DD>-<slug>.png` + sidecar `.txt` עם ה-prompt, verification, ודיווח. נוצר סקיל `.claude/skills/gpt-image-gen/SKILL.md` עם תבנית curl ל-`/v1/images/generations`, חילוץ b64_json דרך jq, ו-python fallback ל-base64 decode. נוצרו `yuval/reference/.gitkeep` ו-`yuval/outputs/.gitkeep`. `.env` + `.env.example` קיבלו `OPENAI_API_KEY`. `.gitignore` חוסם את `yuval/outputs/*` פרט ל-`.gitkeep`. `yael.md` עודכנה עם פרוטוקול placeholders `{{IMAGE_NEEDED: "..."}}` שהיא משאירה במקומות לתמונה, ושורת סיכום שמדווחת אותם לראובן. `CLAUDE.md` עודכן: יובל ברשימת הצוות, trigger keywords עברית+אנגלית, ותהליך "מאמר עם תמונות" end-to-end (yael→placeholders→yuval לכל אחד→שילוב יחסי `../yuval/outputs/...` ב-MD ו-HTML).
- **Decisions:** מודל `gpt-image-2` (יצא 2026-04-21) — הסקיל ויובל מתעדים פעמיים שאסור להחליפו ל-dall-e למרות שייתכן שהמודל לא מופיע בידע הפנימי. נתיב יחסי `../yuval/outputs/<file>.png` נבחר על פני העתקה ל-`Output/images/` כדי לשמור על פשטות. slugs תמיד ASCII כדי לא לשבור paths כש-yael שומרת קבצים בעברית. yuval/outputs/ gitignored (תמונות מיוצרות, לא ל-git). `quality: medium` ו-`size: 1024x1024` כברירת מחדל; שינוי דורש החלטה מודעת.
- **Notes / Caveats:** ה-pipeline end-to-end עוד לא נבחן על מקרה אמיתי — `yuval/reference/` ריק עד שהמשתמש ימלא, ו-`OPENAI_API_KEY` ב-`.env` עדיין placeholder ריק (המשתמש מילא ידנית). הסקיל כולל python fallback למקרה ש-jq לא מותקן (Git Bash על Windows וכו׳).
- **Related:** [[claude-md-instructions]], [[environment-and-secrets]], [[skills-catalog]], [[project-overview]], [[crm-article-rewrite]]
