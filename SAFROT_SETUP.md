# הגדרת ספרות מקצועית - Google Sheets + Apps Script

מדריך זה מסביר כיצד להגדיר את מערכת הספרות המקצועית כך שניתן לעדכן את רשימת המסמכים ישירות מ-Google Sheets.

## שלב 1: יצירת Google Sheet

1. פתח את ה-Google Sheet שכבר קיים (אותו שיש בו את sheet של הלומדות)
2. צור sheet חדש בשם **safrot** (חשוב! השם חייב להיות בדיוק safrot)
3. בשורה הראשונה (כותרות), הזן:
   - עמודה A: `emoji`
   - עמודה B: `title`
   - עמודה C: `url`

## שלב 2: הוספת נתונים לדוגמה

הוסף מסמכים בשורות הבאות:

| emoji | title | url |
|-------|-------|-----|
| 📕 | פרוטוקול הלם | https://drive.google.com/file/d/YOUR_FILE_ID/view |
| 📘 | פרוטוקול כוויות | https://drive.google.com/file/d/YOUR_FILE_ID/view |
| 📗 | פרוטוקול טראומה | https://drive.google.com/file/d/YOUR_FILE_ID/view |
| 📙 | הנחיות החייאה | https://drive.google.com/file/d/YOUR_FILE_ID/view |
| 📔 | מדריך תרופות | https://drive.google.com/file/d/YOUR_FILE_ID/view |
| 🩺 | פרוטוקול אנפילקסיס | https://drive.google.com/file/d/YOUR_FILE_ID/view |
| 💊 | מינוני תרופות חירום | https://drive.google.com/file/d/YOUR_FILE_ID/view |
| 🫀 | פרוטוקולי לב | https://drive.google.com/file/d/YOUR_FILE_ID/view |

### איך לקבל קישור ל-PDF בדרייב:
1. העלה את ה-PDF ל-Google Drive
2. לחץ ימני → "קבל קישור"
3. שנה את ההרשאות ל-"כל מי שיש לו את הקישור יכול לצפות"
4. העתק את הקישור המלא

## שלב 3: עדכון Apps Script

אם כבר יש לך Apps Script מהלומדות, עדכן אותו:

```javascript
function doGet(e) {
  const sheet = e.parameter.sheet || 'videos'; // ברירת מחדל: videos
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const ws = ss.getSheetByName(sheet);

  if (!ws) {
    return ContentService.createTextOutput(
      JSON.stringify({
        success: false,
        error: 'Sheet not found: ' + sheet
      })
    ).setMimeType(ContentService.MimeType.JSON);
  }

  const data = ws.getDataRange().getValues();
  const headers = data[0];
  const rows = data.slice(1);

  const results = rows
    .filter(row => row[0] && row[1] && row[2]) // רק שורות עם נתונים
    .map(row => {
      let obj = {};
      headers.forEach((header, index) => {
        obj[header] = row[index];
      });
      return obj;
    });

  // החזרת שם שונה בהתאם ל-sheet
  const dataKey = sheet === 'safrot' ? 'documents' : 'videos';

  return ContentService.createTextOutput(
    JSON.stringify({
      success: true,
      [dataKey]: results
    })
  ).setMimeType(ContentService.MimeType.JSON);
}
```

### אם אין לך Apps Script:
1. ב-Google Sheet, לחץ על **Extensions** → **Apps Script**
2. מחק את הקוד הקיים והדבק את הקוד למעלה
3. שמור (Ctrl+S)
4. לחץ על **Deploy** → **New deployment**
5. בחר **Web app**
6. הגדרות:
   - Execute as: **Me**
   - Who has access: **Anyone**
7. לחץ **Deploy**
8. העתק את ה-URL
9. עדכן את ה-URL ב-`safrot-page.html` (שורה 205)

## שלב 4: בדיקה

1. פתח את `safrot-page.html` בדפדפן
2. אמור לראות את כל המסמכים שהגדרת
3. לחיצה על כרטיס תפתח את ה-PDF בדרייב

## שלב 5: עדכון נתונים

מעכשיו, כדי להוסיף/לערוך/למחוק מסמכים:
1. פתח את Google Sheet
2. עבור ל-sheet **safrot**
3. ערוך את השורות (הוסף/מחק/שנה)
4. השינויים יופיעו מיד באתר!

## טיפים

### אימוגי מומלצים למסמכים:
- 📕 📘 📗 📙 📔 - ספרים בצבעים שונים
- 📄 📃 📑 - דפים
- 🩺 💊 💉 - ציוד רפואי
- 🫀 🧠 🫁 - איברים
- ⚕️ 🏥 - רפואה כללי
- ⚠️ 🚨 - דחוף/חירום

### אבטחה
- ודא שקבצי ה-PDF נגישים לכולם (view only)
- לא להעלות מידע רגיש/סודי
- המערכת בודקת שהקישורים הם מ-Google Drive בלבד

## פתרון תקלות

### המסמכים לא נטענים
1. בדוק שה-sheet נקרא בדיוק **safrot**
2. בדוק שיש 3 עמודות: emoji, title, url
3. בדוק שה-Apps Script מעודכן עם הקוד החדש
4. פתח Console (F12) וראה מה השגיאה

### "Sheet not found"
- ה-sheet חייב להיקרא **safrot** (אותיות קטנות)

### הקישורים לא נפתחים
- ודא שקישורי הדרייב מוגדרים ל-"כל מי שיש לו את הקישור יכול לצפות"

---

**נוצר עבור מדור רפואה** 🩺
