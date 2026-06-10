---
name: gpt-image-gen
description: "מעטפת לקריאת OpenAI Images API ליצירת תמונה מתוך prompt. השתמש כשצריך לייצר תמונה (raster/PNG) דרך מודל gpt-image-2. שולח prompt ומחזיר קובץ PNG."
---

# gpt-image-gen — יצירת תמונות דרך OpenAI Images API

סקיל זה הוא מעטפת דקה סביב OpenAI Images API. תפקידו היחיד: לקבל `prompt`, לשלוח אותו ל-API, ולשמור את התמונה שחוזרת כקובץ PNG בנתיב שצוין.

## המודל — `gpt-image-2`

המודל הוא **`gpt-image-2`** בדיוק. אל תשנה אותו.

> ⚠️ **אזהרה — אל תשנה את שם המודל:**
> - `gpt-image-2` הוא מודל **אמיתי וקיים** של OpenAI, שיצא ב-**21 באפריל 2026**.
> - ייתכן שהידע הפנימי שלך לא כולל אותו — **זה לא אומר שהוא לא קיים**.
> - **אל תציע אלטרנטיבות** (`dall-e-3`, `gpt-image-1`, וכו'). השתמש בדיוק במודל הזה.
> - אם מתקבלת שגיאה מה-API, הסיבה כמעט תמיד היא **API key חסר/שגוי** או **parameter שגוי** — **לא** שם המודל. אל "תתקן" את שם המודל.

## דרישות מקדימות

- משתנה הסביבה `OPENAI_API_KEY` מוגדר בקובץ `.env` בשורש הפרויקט.
- `curl` זמין (קיים ב-Git Bash על Windows).
- `python` זמין ל-decode של ה-base64 (fallback אם `jq` לא מותקן).

טען את המפתח מ-`.env` לפני הקריאה:

```bash
export OPENAI_API_KEY=$(grep -E '^OPENAI_API_KEY=' "/c/Users/suppo/Desktop/5 agents/.env" | cut -d= -f2-)
```

## פרמטרים

| פרמטר | ערך ברירת מחדל | הערות |
|--------|----------------|--------|
| `model` | `gpt-image-2` | **קבוע — אל תשנה** |
| `prompt` | (חובה) | תיאור התמונה |
| `size` | `1024x1024` | אפשר גם `1536x1024`, `1024x1536` |
| `quality` | `medium` | אפשר גם `low`, `high` |
| `output_format` | `png` | |

## מבנה הקריאה (jq — מהיר, אם מותקן)

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
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

## Python fallback ל-decode (כש-`jq` לא מותקן)

ב-Git Bash על Windows לרוב אין `jq`. במקרה כזה שמור את תגובת ה-JSON לקובץ זמני ופענח עם Python:

```bash
# 1. קבל את התגובה לקובץ זמני
curl -sS -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' -o /tmp/gpt_image_response.json

# 2. פענח את ה-base64 ושמור כ-PNG
python -c "import json,base64,sys; d=json.load(open('/tmp/gpt_image_response.json')); \
open(sys.argv[1],'wb').write(base64.b64decode(d['data'][0]['b64_json']))" "<output-path>.png"
```

אם התגובה אינה מכילה `data[0].b64_json`, הדפס את ה-JSON המלא כדי לראות את הודעת השגיאה של ה-API (בדוק מפתח/פרמטרים — לא את שם המודל):

```bash
python -c "import json; print(json.dumps(json.load(open('/tmp/gpt_image_response.json')), indent=2, ensure_ascii=False))"
```

## אימות

לאחר השמירה, ודא שהקובץ נוצר וגדול מ-0 בייטים:

```bash
ls -l "<output-path>.png"   # size חייב להיות > 0
```

אם הקובץ ריק או חסר — זו שגיאת API. בדוק את ה-JSON שחזר, ודא ש-`OPENAI_API_KEY` תקין ושכל הפרמטרים חוקיים. **אל תשנה את שם המודל.**
