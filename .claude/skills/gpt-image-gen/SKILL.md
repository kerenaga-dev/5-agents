# gpt-image-gen — יצירת תמונות עם OpenAI Images API

סקיל זה מעטיף קריאה ל-OpenAI Images API ויוצר תמונת PNG מ-prompt טקסטואלי.

---

## שימוש

קרא לסקיל זה עם:
- **`prompt`** — תיאור התמונה הרצויה
- **`output_path`** — נתיב מלא לשמירת ה-PNG (לדוגמה: `yuval/outputs/2026-06-10-hero.png`)

---

## ביצוע — Primary (curl + jq)

```bash
source .env 2>/dev/null || true

curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

**בדיקת הצלחה:**
```bash
[ -s <output-path>.png ] && echo "OK" || echo "ERROR: file missing or empty"
```

---

## ביצוע — Fallback (python3, לסביבות בלי jq)

```bash
source .env 2>/dev/null || true

RESPONSE=$(curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }')

python3 - <<'EOF'
import sys, json, base64
data = json.loads(''''"'"'$RESPONSE'"'"''')
b64 = data["data"][0]["b64_json"]
with open("<output-path>.png", "wb") as f:
    f.write(base64.b64decode(b64))
EOF
```

---

## פרמטרים קבועים

| פרמטר | ערך |
|--------|-----|
| model | `gpt-image-2` |
| size | `1024x1024` |
| quality | `medium` |
| output_format | `png` |

> ⚠️ אל תשנה את שם המודל. `gpt-image-2` הוא מודל קיים שיצא ב-21.4.2026.
> אם יש שגיאה — בדוק את `OPENAI_API_KEY` ואת הפרמטרים, לא את שם המודל.

---

## טיפול בשגיאות

- **401** — `OPENAI_API_KEY` חסר או לא תקין
- **400** — בעיה בפרמטרים (prompt ריק, size לא נתמך)
- **429** — rate limit — המתן ונסה שוב
- **קובץ ריק / גודל 0** — ה-decode נכשל; בדוק שה-response מכיל `data[0].b64_json`
