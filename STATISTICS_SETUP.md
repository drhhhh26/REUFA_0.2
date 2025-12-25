# הגדרת מערכת סטטיסטיקות - Statistics Dashboard

מדריך זה מסביר כיצד להגדיר את מערכת הסטטיסטיקות האנונימית.

---

## 📊 שלב 1: יצירת Google Sheet לסטטיסטיקות

### 1.1 צור Google Sheet חדש או הוסף גליון
- **אפשרות A:** הוסף גליון חדש לקובץ הקיים "מאגר שאלות - מדור רפואה"
- **אפשרות B:** צור קובץ חדש בשם "סטטיסטיקות מבחנים"

### 1.2 צור גליון בשם `exam_statistics`

שנה את השם של הגליון ל-`exam_statistics` **בדיוק**!

---

## 📝 שלב 2: מבנה העמודות

בגליון `exam_statistics`, הוסף את העמודות הבאות (שורה 1 = כותרות):

| A | B | C | D | E | F |
|---|---|---|---|---|---|
| timestamp | exam_name | question_text | topic | difficulty | is_correct |

### הסבר על העמודות:

- **timestamp** - תאריך ושעה (אוטומטי)
- **exam_name** - שם המבחן (מבחן פתיחה חובש, מבחן סיום מט״ב, וכו')
- **question_text** - טקסט השאלה המלא
- **topic** - נושא (trauma / rescue)
- **difficulty** - רמת קושי (Easy / Medium / Hard)
- **is_correct** - האם הנבחן ענה נכון (TRUE / FALSE)

**⚠️ חשוב:** אין עמודת שם, אין עמודת ת"ז - **אנונימי לחלוטין!**

---

## 💻 שלב 3: Apps Script לסטטיסטיקות

### 3.1 פתיחת Apps Script
1. ב-Google Sheet עם ה-statistics, לחץ על **Extensions** → **Apps Script**
2. זה אותו Apps Script כמו למבחנים (אם השתמשת באפשרות A)

### 3.2 עדכן את הקוד הקיים או הוסף פונקציה חדשה:

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);

    // אם זה שליחת תוצאות רגילות (עם name, id)
    if (data.name && data.id) {
      const ss = SpreadsheetApp.getActiveSpreadsheet();
      const ws = ss.getSheetByName('exam_results');

      if (!ws) {
        return ContentService.createTextOutput(
          JSON.stringify({ success: false, error: 'Sheet not found: exam_results' })
        ).setMimeType(ContentService.MimeType.JSON);
      }

      ws.appendRow([
        new Date(),
        data.name,
        data.id,
        data.unit || '',
        data.exam,
        data.score,
        data.total
      ]);

      return ContentService.createTextOutput(
        JSON.stringify({ success: true, message: 'Results saved' })
      ).setMimeType(ContentService.MimeType.JSON);
    }

    // אם זה שליחת סטטיסטיקות (array של תשובות)
    if (data.statistics && Array.isArray(data.statistics)) {
      const ss = SpreadsheetApp.getActiveSpreadsheet();
      const ws = ss.getSheetByName('exam_statistics');

      if (!ws) {
        return ContentService.createTextOutput(
          JSON.stringify({ success: false, error: 'Sheet not found: exam_statistics' })
        ).setMimeType(ContentService.MimeType.JSON);
      }

      // הוספת כל שורה
      data.statistics.forEach(stat => {
        ws.appendRow([
          new Date(),
          stat.exam_name,
          stat.question_text,
          stat.topic,
          stat.difficulty,
          stat.is_correct
        ]);
      });

      return ContentService.createTextOutput(
        JSON.stringify({ success: true, message: 'Statistics saved', count: data.statistics.length })
      ).setMimeType(ContentService.MimeType.JSON);
    }

    return ContentService.createTextOutput(
      JSON.stringify({ success: false, error: 'Invalid data format' })
    ).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService.createTextOutput(
      JSON.stringify({ success: false, error: error.toString() })
    ).setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  const action = e.parameter.action;

  // אם מבקשים סטטיסטיקות
  if (action === 'statistics') {
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const ws = ss.getSheetByName('exam_statistics');

    if (!ws) {
      return ContentService.createTextOutput(
        JSON.stringify({ success: false, error: 'Sheet not found: exam_statistics' })
      ).setMimeType(ContentService.MimeType.JSON);
    }

    const data = ws.getDataRange().getValues();
    const headers = data[0];
    const rows = data.slice(1);

    const statistics = rows.map(row => {
      let obj = {};
      headers.forEach((header, index) => {
        obj[header] = row[index];
      });
      return obj;
    });

    return ContentService.createTextOutput(
      JSON.stringify({ success: true, statistics: statistics, total: statistics.length })
    ).setMimeType(ContentService.MimeType.JSON);
  }

  // קוד קיים לשאלות...
  const sheetName = e.parameter.sheet || 'exam_hovesh_1';
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const ws = ss.getSheetByName(sheetName);

  if (!ws) {
    return ContentService.createTextOutput(
      JSON.stringify({ success: false, error: 'Sheet not found: ' + sheetName })
    ).setMimeType(ContentService.MimeType.JSON);
  }

  const data = ws.getDataRange().getValues();
  const headers = data[0];
  const rows = data.slice(1);

  const questions = rows
    .filter(row => row[0] && row[0].toString().trim() !== '')
    .map(row => {
      let obj = {};
      headers.forEach((header, index) => {
        obj[header] = row[index];
      });
      return obj;
    });

  return ContentService.createTextOutput(
    JSON.stringify({ success: true, questions: questions, total: questions.length })
  ).setMimeType(ContentService.MimeType.JSON);
}
```

### 3.3 פריסה מחדש
1. שמור את הקוד (Ctrl+S)
2. לחץ על **Deploy** → **Manage deployments**
3. לחץ על ✏️ (Edit) ליד הפריסה הקיימת
4. **New version** - תיאור: "Added statistics support"
5. לחץ **Deploy**
6. **העתק את ה-URL** (זה אותו URL כמו קודם)

---

## 🔗 שלב 4: עדכון קבצי המבחנים

קבצי ה-HTML כבר מעודכנים אוטומטית - הם ישלחו סטטיסטיקות!

כל מה שצריך:
- להשתמש באותו RESULTS_URL כמו קודם
- הקוד ישלח אוטומטית גם results וגם statistics

---

## 📊 שלב 5: גישה ל-Dashboard

פשוט פתח את הקובץ:
```
instructor-dashboard.html
```

**אין צורך בסיסמה!** כל מי שיודע את ה-URL יכול לראות.

---

## ✅ בדיקה

1. השלם מבחן אחד
2. פתח את Google Sheet → `exam_statistics`
3. אמור לראות 25 שורות חדשות (שורה לכל שאלה)
4. כל שורה מכילה: timestamp, exam_name, question_text, topic, difficulty, is_correct

---

## 🎯 מה הדאשבורד יציג?

- **📈 Success Rate per Question** - אילו שאלות הכי קלות/קשות
- **🥧 Performance by Topic** - טראומה vs חילוץ
- **📊 Difficulty Breakdown** - Easy vs Medium vs Hard
- **📉 Timeline** - מגמות לאורך זמן
- **🔝 Hardest Questions** - 5 השאלות הקשות ביותר

---

**נוצר עבור מדור רפואה** 🩺
