# 🧩 פלייבוק — סיסטם לבניית דף מוצר ואתר בשופיפיי

> **המטרה:** שלד קבוע לבניית דף נחיתה/מוצר בסגנון "צומחים מחדש", כדי שכל מוצר חדש ייבנה מהר, מסודר, לפי תוכנית ברורה.
> נבנה ונבדק על החנות `tzomchim-mechadash`. RTL עברית, Heebo, פלטת אדום/לבן פרימיום.

---

## 0. עקרונות-על (מה עשה את האתר טוב)

1. **דף אחד, סקשן אחד (`landing-page.liquid`)** — כל הדף הוא Section אחד עם CSS מוטמע, מתוחם תחת `.scm-page` כדי לא לדלוף לתבנית.
2. **מבנה רגשי → הוכחה → הצעה → אמון → סגירה.** לא סתם רשימת פיצ'רים.
3. **אדום אחד מעודן כאקסנט בלבד** על קנבס לבן. איפוק = יוקרה נתפסת.
4. **RTL מלא + פונט אחד (Heebo)** בכל האתר.
5. **הכל דינמי מהגדרות הסקשן** — מוצרים, תמונה, סרטון נבחרים ב-Theme Editor, לא קשיח בקוד.
6. **Workflow דו-ענפי** — פיתוח ב-`claude/regrow-handoff`, מיזוג ל-`shopify-theme` (הענף החי שממנו שופיפיי מושכת).

---

## 1. ארכיטקטורת קבצים

```
sections/
  landing-page.liquid          ← הדף כולו (CSS+HTML+JS+Schema settings). הקובץ המרכזי.
  regrow-terms.liquid          ← עמוד תקנון (רק עוטף את הסניפט)
  main-page.liquid             ← תוקן: מציג תקנון אוטומטית אם handle=terms
snippets/
  regrow-terms-content.liquid  ← תוכן התקנון (משותף: עמוד + חלון קופץ)
templates/
  index.json                   ← דף הבית: order=["landing-page"] + חיבורי מוצרים
  page.terms.json              ← תבנית עמוד תקנון (type: regrow-terms)
layout/
  theme.liquid                 ← dir=rtl, Heebo גלובלי, meta description, הסתרת header/footer בדף הבית
assets/
  tz-*.webp / tz-video.mp4 / ba-*.jpeg   ← נכסים
```

**כלל זהב:** דף הבית מסתיר את ה-header/footer של התבנית (יש לדף שלנו ניווט+פוטר משלו):
```liquid
{%- unless request.page_type == 'index' -%}{% section 'header' %}{%- endunless -%}
```

---

## 2. מערכת העיצוב (Design Tokens)

הדבק את בלוק ה-`:root` הזה בראש ה-CSS ושנה רק את צבע ה-`--red` למותג הבא:

```css
.scm-page{
  --red:#D11F2D; --red-dark:#B11825; --red-soft:#F7E6E7; --red-tint:#FCF1F1;   /* ← המותג */
  --canvas:#FAFAF8; --white:#FFFFFF; --panel:#F3F1EB; --panel2:#EEEBE3;
  --ink:#1A1A1A; --ink2:#3A3A38; --muted:#6B6B66; --faint:#9A9A93;
  --line:#E7E4DD; --line2:#DEDAD1;
  --gold:#E0A93B; --green:#2E7D5B;
  --sans:'Heebo',system-ui,Arial,sans-serif;
  direction:rtl; text-align:right; background:var(--canvas); color:var(--ink);
  font-family:var(--sans); line-height:1.6; overflow-x:hidden;
}
```

**כללי עיצוב:**
- כותרות: Heebo 800. גוף: 400. קפיצות גודל גדולות (`clamp()`).
- פינות 10–14px. צללים רכים (`0 8px 24px rgba(26,26,26,.05)`). קווים דקים, לא מסגרות קשות.
- אדום רק ל: CTA, מחיר, באדג'ים, קו הדגשה, עיגול אישור. **לא** לרקעי סקשן שלמים.
- ריווח סקשן אחיד: `padding:clamp(58px,7vw,98px) 0`.
- כל `.wrap` עם `max-width` אחיד (1180 / 980 / 820 / 720).

**פונט:** טען Heebo גלובלית ב-`theme.liquid` + override עם `!important` על body/כותרות/כפתורים/מחירים.

---

## 3. שלד הדף — 14 הסקשנים לפי הסדר

| # | סקשן | תפקיד |
|---|---|---|
| 0 | בס״ד (ימין למעלה) | מיתוג עדין |
| 1 | מרקיזת הכרזות | משלוח חינם · תשלום מאובטח · X לקוחות |
| 2 | ניווט sticky | **שמאל:** לוגו + "לרכישה" · **ימין:** המבצעים/איך זה עובד/תקנון/דברו איתנו |
| 3 | HERO (flexbox!) | אזהרה⚠️ → pill → H1 → lead → צ'קליסט → CTA+דירוג \| תמונה |
| 4 | סרטון מוצר + טקסט רגשי | וידאו אנכי/רוחב + סיפור רגשי לצד |
| 5 | לפני/אחרי | זוגות תמונות עם תוויות לפני/אחרי |
| 6 | OFFER (המבצעים) | עיגול אישור תקנון + 3 כרטיסים (מוצר/קומבו1/קומבו2 מומלץ) |
| 7 | איך זה עובד | 3 צעדים, X דקות ביום |
| 8 | ביקורות | דירוג ממוצע + כרטיסי ביקורת (שמות מלאים) |
| 9 | FAQ | אקורדיון `<details>` + FAQPage Schema |
| 10 | השוואה | טבלה: אנחנו מול קליניקה מול תרופות |
| 11 | מרקיזת מבצע | פס אדום עם ההצעה |
| 12 | CTA סופי | מחיר + כפתור "מתחילים היום" |
| 13 | פוטר | לוגו + קישורים + מדיניות |
| צף | טוסט "נרכש עכשיו" + כפתור "צרו קשר" + חלון תקנון קופץ |

---

## 4. הקומפוננטות הרב-פעמיות (העתק-הדבק)

### א. כרטיס הצעה (3 בשורה, המומלץ מוגדל)
- Grid `1fr 1fr 1fr`. הכרטיס המומלץ: `transform:scale(1.05)` + מסגרת אדומה + באדג' "⭐ הכי משתלם".
- כרטיס בסיסי = טופס Shopify אמיתי. קומבו = כפתור `data-combo-add="id1,id2"` → JS fetch ל-`cart/add.js`.

### ב. עיגול אישור תקנון (חוסם רכישה) 🔴
- `<input type="checkbox">` מעוצב כעיגול (appearance:none, border-radius:50%, וי לבן ב-`:checked::after`).
- JS `termsOK()` חוסם גם טופס וגם קומבו אם לא מסומן — עם רעידה (`scm-shake`).

### ג. חלון תקנון קופץ (בלי תלות בעמוד/404)
- כל קישורי התקנון = `href="#" data-open-terms`. JS פותח modal עם `{% render 'regrow-terms-content' %}`.
- **למה:** מבטל תלות בקיום Page בשופיפיי → אין 404 לעולם.

### ד. וידאו עם CTA בסוף
- `<video controls>` + overlay `data-vcta`. JS: ב-`ended`/2.5 שניות אחרונות → מציג "לחצו כאן ←" שמפנה ל-`#offer`.
- **קידוד וידאו:** `.mov` לא מתנגן בכרום → המר ל-MP4 faststart: `ffmpeg -i in.mov -vf scale=1280:-2 -c:v libx264 -crf 23 -c:a aac -movflags +faststart out.mp4` (דרך `pip install imageio-ffmpeg`).

### ה. טוסט "נרכש עכשיו" + טופס יצירת קשר צף
- טוסט מחזורי משמות (שם מלא + פועל מגדרי). טופס → `mailto:`.
- ⚠️ שמות/ביקורות = **בדויים**. להחליף לאמיתיים בהקדם (חוק הגנת הצרכן).

### ו. FAQ + Schema
- אקורדיון נטיבי `<details><summary>` (בלי JS). ה-`+` מסתובב ל-`×` ב-`[open]`.
- לשכפל את אותן שאלות ב-JSON-LD `FAQPage`.

---

## 5. פטרנים טכניים חשובים

- **HERO ב-Flexbox, לא Grid!** Grid עם `fr` + תמונה = הכותרת נשברת מילה-בשורה. השתמש:
  ```css
  .hero .in{display:flex;flex-wrap:wrap;align-items:center;gap:clamp(32px,5vw,64px)}
  .hero .copy{flex:1 1 380px;min-width:0}
  .hero .media{flex:1 1 300px;min-width:0;max-width:440px}
  ```
- **מחיר ב-₪ תמיד** (בלי תלות במטבע החנות):
  `{{ product.price | money_without_currency | replace: ',', '' | replace: '.00','' }} ש"ח`
- **JSON-LD:** Organization + Product + FAQPage. להימנע ממרכאות כפולות בתוך הטקסט (שוברות JSON).
- **קומבו add-to-cart:** `fetch('{{ routes.cart_add_url }}.js', {items:[...]})` → redirect ל-`{{ routes.cart_url }}`.
- **scroll-reveal:** IntersectionObserver מוסיף `.in`. לכבד `prefers-reduced-motion`. לא לשים על הכרטיס המומלץ (הורס scale).
- **RTL:** `dir="rtl"` ב-`<html>` + `inset-inline-start/end` במקום left/right.

---

## 6. Workflow פריסה (Git)

```bash
# פיתוח
git checkout claude/regrow-handoff
# ...עריכות...
git add -A && git commit -m "..."
git push -u origin claude/regrow-handoff
# מיזוג לענף החי (שופיפיי מושכת ממנו)
git checkout shopify-theme
git merge --ff-only claude/regrow-handoff
git push -u origin shopify-theme
git checkout claude/regrow-handoff
```
- שופיפיי דוחפת לפעמים "Update from Shopify" (הגדרות עורך) → למזג פנימה עם `git merge origin/shopify-theme` לפני שממשיכים.
- ולידציה לפני פוש: איזון `<div>`, תקינות JSON-LD, ספירת H1=1.

---

## 7. צ'קליסט ידני בשופיפיי (לכל מוצר חדש)

- [ ] **Theme Editor** → סקשן הדף → לחבר `main_product` + `addon_serum` + `addon_shampoo`
- [ ] **Automatic Discount** — הנחות שמורידות את הקומבו למחיר היעד (למשל 219 / 249)
- [ ] **מטבע** = ILS ₪ (Settings → General)
- [ ] מחירי מוצרים + Compare-at
- [ ] **handle** של עמוד התקנון = `terms`
- [ ] תמונות/סרטון הועלו ל-assets, שמות נכונים בהגדרות הסקשן
- [ ] בדיקה בחנות החיה (RTL, ₪, כותרת אופקית, כפתורי קנייה, חוסם תקנון)
- [ ] Google Rich Results Test ל-Schema

---

## 8. תוכנית עבודה למוצר הבא (Step-by-step)

1. **תוכן:** לאסוף — שם מוצר, כאב, פתרון, 4 יתרונות, 3 צעדים, מחירים, תמונות, סרטון, לפני/אחרי.
2. **Duplicate** את `landing-page.liquid` (או להשתמש כמו שהוא ולשנות טקסטים).
3. **החלף `--red`** לצבע המותג החדש (שאר הטוקנים נשארים).
4. **עדכן טקסטים:** H1 (כותרת נגד-התנגדות), lead, צ'קליסט, בעיה, פתרון, צעדים, FAQ, ביקורות.
5. **החלף נכסים:** תמונות `tz-*`, `ba-*`, וידאו (המר ל-MP4 faststart).
6. **עדכן Schema:** שם מוצר, תיאור, מחיר, שאלות FAQ.
7. **חבר מוצרים** ב-Theme Editor + Automatic Discount.
8. **בדוק** לפי צ'קליסט סעיף 7 → פרוס לפי סעיף 6.

---

*נבנה עבור צומחים מחדש. שכפל, החלף מותג, שגר. 🚀*
