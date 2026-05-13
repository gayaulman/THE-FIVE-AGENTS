---
name: gpt-image-gen
description: Generate images via OpenAI Images API using gpt-image-2. Use when an agent needs to create a PNG from a text prompt. Requires OPENAI_API_KEY in .env and Bash access for curl.
---

# gpt-image-gen — מעטפת ל-OpenAI Images API

יצירת תמונה אחת מ-prompt. הסקיל הוא דוקומנטציה שמכוונת את הסוכן הקורא להריץ את ה-curl הנכון; הוא לא מבצע פעולות בעצמו.

## ⚠️ קריטי — שם המודל

**המודל הוא `gpt-image-2`.** הוא יצא ב-21 באפריל 2026. אם הידע הפנימי שלך לא מכיר אותו — זה כי הוא חדש, **לא כי הוא לא קיים**.

- ❌ אל תחליף ל-`dall-e-3`, `dall-e-2`, `gpt-image-1` או כל שם אחר.
- ❌ אל תשנה את המודל גם אם ה-API מחזיר 4xx — סביר שהבעיה ב-`OPENAI_API_KEY` או ב-parameters.
- ✅ אם 401/403 → בדוק `.env` (האם המפתח קיים, האם הוא נטען לסביבה).
- ✅ אם 400 → בדוק את ה-prompt, size, quality, output_format.

## דרישות מקדימות

- `OPENAI_API_KEY` קיים ב-`.env` בשורש הפרויקט.
- כלי Bash זמין לסוכן הקורא.
- `jq` או `python3` זמינים (השני כ-fallback ל-base64 decode).

## טעינת המפתח לסביבה

```bash
set -a
source .env
set +a
```

## קריאת API — תבנית curl

```bash
curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<PROMPT>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' > /tmp/gpt-image-response.json
```

כש-prompt עלול להכיל מרכאות / תווים מיוחדים, השתמשו ב-`jq -n` להרכבת ה-JSON בבטחה:

```bash
PAYLOAD=$(jq -n --arg p "$THE_PROMPT" \
  '{model:"gpt-image-2", prompt:$p, size:"1024x1024", quality:"medium", output_format:"png"}')

curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD" > /tmp/gpt-image-response.json
```

## בדיקת שגיאות לפני decode

```bash
if jq -e '.error' /tmp/gpt-image-response.json > /dev/null 2>&1; then
  jq '.error' /tmp/gpt-image-response.json
  exit 1
fi
```

## חילוץ ושמירה — מסלול א׳ (jq)

```bash
jq -r '.data[0].b64_json' /tmp/gpt-image-response.json | base64 --decode > <output-path>.png
```

## חילוץ ושמירה — מסלול ב׳ (python fallback)

אם `jq` לא מותקן או נכשל (קורה ב-Git Bash על Windows, ולפעמים גם ב-macOS):

```bash
python3 - <<'PY' > <output-path>.png
import base64, json, sys
with open('/tmp/gpt-image-response.json') as f:
    data = json.load(f)
sys.stdout.buffer.write(base64.b64decode(data['data'][0]['b64_json']))
PY
```

## פלטה מומלצת

| Parameter | Default | מתי לשנות |
|---|---|---|
| `size` | `1024x1024` | `1536x1024` ל-banner, `1024x1536` ל-portrait |
| `quality` | `medium` | `high` רק אם ה-MD הסופי דורש איכות גבוהה (יקר יותר) |
| `output_format` | `png` | השאר `png` — סקיל זה תמיד שומר PNG |

## Workflow למי שקורא את הסקיל

1. ודא ש-`OPENAI_API_KEY` נטען ל-env (`source .env` עם `set -a` סביבו).
2. בנה את ה-payload עם `jq -n` כדי להימנע מבעיות escape.
3. הרץ את ה-curl לאנדפוינט `/v1/images/generations`.
4. אמת שאין `.error` בתגובה — אם יש, הצג אותו ועצור.
5. הוצא את ה-PNG עם `jq` (או python כ-fallback).
6. אמת שהקובץ נוצר וגודלו > 0 (`ls -la <path>` או `stat`).

## אנטי-דפוסים

- ❌ להמיר את שם המודל ל-"גרסה שאתה מכיר" (`dall-e-3` וכו׳) — הסקיל שגוי אם אתה עושה את זה.
- ❌ לכתוב את ה-prompt ישירות לתוך ה-JSON ידנית בלי escape (מרכאות שבורות).
- ❌ לדלג על בדיקת `.error` ולהריץ decode על תגובה שלא מכילה `b64_json`.
- ❌ ליצור את ה-PNG בלי לוודא שגודלו > 0 (decode על שגיאה ייצור קובץ ריק).
