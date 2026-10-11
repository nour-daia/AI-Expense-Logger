03 — Data Design

 Trigger
ورود رد جديد على Google Form اسمه `Expense Entry`. كل رد جديد يشغّل الـZap مرة واحدة.

 Inputs

| Field | Type | Required | Example |
|-------|------|----------|---------|
| Expense_Text | نص حر (Paragraph) | نعم | دفعت 45 شيكل غدا مع عميل أمس |
| Receipt_Link | رابط / رفع ملف | لا | رابط صورة الفاتورة |
| Submitted_At | Timestamp تلقائي من الفورم | تلقائي | 2026-10-11 19:12 |

Unstructured Data
- **Expense_Text:** جملة عربية حرة. قد تحتوي على مبلغ، جهة، تاريخ نسبي ("أمس"، "اليوم"، "قبل يومين")، وأرقام مكتوبة بالأرقام العربية (٤٥) أو الإنجليزية (45).
- **Receipt_Link:** صورة فاتورة اختيارية، تُحفظ كرابط فقط ولا يحللها الـAI.

 Structured Data
هذه الحقول يستخرجها الـAI من `Expense_Text`، ويضيف Zapier بعضها:

| Field | Source | Type | Allowed values / Format |
|-------|--------|------|-------------------------|
| Amount | AI | رقم | أكبر من 0 |
| Currency | AI | نص | ILS (الافتراضي), USD, JOD, EUR |
| Expense_Date | AI | تاريخ | YYYY-MM-DD |
| Vendor | AI | نص | اسم الجهة، أو فارغ إذا لم يُذكر |
| Category | AI | نص | طعام، مواصلات، برامج واشتراكات، مواد ومستلزمات، أخرى |
| Confidence | AI | نص | High / Medium / Low |
| Status | Rule | نص | Logged / Needs Review |
| Review_Reason | Rule | نص | سبب التعليم للمراجعة، أو فارغ |

 Data Schema (Google Sheets: `Expenses`)

| Column | Type | Description |
|--------|------|-------------|
| Submitted_At | Datetime | وقت إرسال الفورم |
| Expense_Text | Text | النص الأصلي كما كُتب |
| Amount | Number | المبلغ |
| Currency | Text | العملة |
| Expense_Date | Date | تاريخ المصروف |
| Vendor | Text | الجهة |
| Category | Text | التصنيف |
| Confidence | Text | ثقة الـAI |
| Status | Text | Logged / Needs Review |
| Review_Reason | Text | سبب المراجعة |
| Receipt_Link | URL | رابط الفاتورة (اختياري) |

 Missing Information

| Missing | How it is handled |
|---------|-------------------|
| لا يوجد مبلغ في النص | لا يُسجَّل صف، ويصلني إيميل طلب توضيح |
| لا يوجد تاريخ | يُستخدم تاريخ إرسال الفورم |
| لا توجد عملة | يُفترض ILS (شيكل) |
| لا توجد جهة | يُترك الحقل فارغًا، ولا يؤثر على التسجيل |
| لا يوجد تصنيف واضح | `أخرى`، وتصبح الحالة `Needs Review` |

 Validation

| Check | Rule | If it fails |
|-------|------|-------------|
| النص غير فارغ | Expense_Text ليس فارغًا | لا يُشغَّل باقي الـZap |
| يحتوي رقمًا | النص يحتوي رقمًا واحدًا على الأقل | إيميل طلب توضيح |
| المبلغ صالح | Amount رقم وأكبر من 0 | `Needs Review` |
| التصنيف مسموح | Category ضمن القائمة | `أخرى` + `Needs Review` |
| العملة متوقعة | Currency ضمن القائمة | `Needs Review` |
| التاريخ منطقي | Expense_Date ليس في المستقبل | `Needs Review` |
| حد المراجعة | Amount أكبر من 500 | `Needs Review` |
| ثقة الـAI | Confidence = Low | `Needs Review` |

 Notes
- النص الأصلي يُحفظ دائمًا في الصف، حتى أقارنه بما فهمه الـAI وأكتشف الأخطاء.
- حد 500 شيكل يُطبَّق على المبلغ كما هو مسجَّل. المبالغ بعملات أخرى تذهب لـ`Needs Review` في هذه النسخة بدل تحويلها (لتبسيط المشروع).
- لا توجد أعمدة للتكرار، لأن كل إدخال يُسجَّل كصف مستقل (قرار موثّق في `02-workflow-decomposition.md`).
