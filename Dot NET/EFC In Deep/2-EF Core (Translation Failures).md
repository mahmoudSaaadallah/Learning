## 🗒️ ملخص حالات فشل الترجمة في EF Core (Translation Failures)

الـ EF Core بيفشل لما الـ **Expression Tree** تحتوي على عناصر "مالهاش قاموس" في الـ SQL. إليك أشهر الحالات:

### 1. مناداة دالة خاصة (Custom Methods / Logic)

أي دالة إنت كاتبها بإيدك جوه الـ C# هي "صندوق أسود". الـ EF Core مش هيقدر يدخل جوه الدالة ويقرأ السطور اللي فيها عشان يحولها لـ SQL.

- **مثال:** `Where(u => CheckStatus(u))`
    
- **لماذا فشلت؟** الـ SQL لا يعرف شيئاً عن `CheckStatus`.
    

### 2. الدوال المنطقية داخل الـ Value Objects

الـ Value Object غالباً بيحتوي على Business Logic (دوال). الـ EF Core بيقدر يترجم "البيانات" (Properties) بس مش "التصرفات" (Methods).

- **مثال:** `Where(p => p.Price.IsExpensive())`
    
- **لماذا فشلت؟** الـ EF Core بيشوف `IsExpensive()` كدالة C# مجهولة، مش كعملية حسابية (`Price > 1000`).
    

### 3. استخدام مكتبات خارجية (Third-party Libraries)

لو استخدمت مكتبة لتشفير البيانات، أو معالجة النصوص، أو حسابات جغرافية مش مدعومة رسمياً من الـ Provider.

- **مثال:** `Where(u => MyCryptoLib.Decrypt(u.Email) == "ali@test.com")`
    
- **لماذا فشلت؟** كود المكتبة الخارجية "مترجم جاهز" (Compiled)، الـ EF Core مش شايف "الخريطة" بتاعته عشان يترجمها.
    

### 4. العمليات البرمجية البحتة (Programming-Only Logic)

بعض الأدوات في C# مالهاش مقابل مباشر أو سهل في الـ SQL، زي الـ Regex أو بعض دوال الـ Reflection.

- **مثال:** `Where(u => Regex.IsMatch(u.Phone, @"^\d{11}$"))`
    
- **لماذا فشلت؟** محرك الـ SQL (في الغالب) مابيفهمش الـ Regex Patterns مباشرة من خلال الـ EF Core.
    

### 5. التحويلات المعقدة بين الأنواع (Unsupported Casting)

لما تحاول تحول نوع بيانات لنوع تاني بطريقة الـ SQL مبيعرفش يعملها في خطوة واحدة.

- **مثال:** `Where(u => u.Id.ToString().Contains("5"))` _(ملحوظة: دي كانت بتفشل زمان، حالياً بعض الـ Providers بقوا يدعموها، لكنها مثال جيد للعمليات اللي بتجهد المترجم)._
    

---

### 💡 القاعدة الذهبية للحل (The Fix)

عشان تتجنب الفشل ده، دايماً "فكك" (Deconstruct) طلبك لبيانات أولية (Primitive Types) الـ SQL بيفهمها:

- بدل `IsSpecial()` استخدم `Balance > 1000`.
    
- بدل `IsExpensive()` استخدم `Amount > 1000`.





## التحليل التقني العميق: لماذا تفشل الترجمة؟ (The Technical Why)

### 1. حاجز الـ IL (Intermediate Language Boundary)

عندما تكتب دالة عادية (Custom Method)، الـ Compiler يحولها إلى **IL Code**. الـ IL هو عبارة عن تعليمات منخفضة المستوى (Low-level instructions) موجهة للـ CLR لتنفيذها على المعالج.

- **المشكلة:** الـ EF Core لا يملك "Disassembler" داخلي يقوم بقراءة الـ IL وتحويله عكسياً إلى منطق مفهوم. بمجرد تحول الكود إلى IL، تُفقد الروابط المنطقية (مثل: هل كان هذا `if` أم `switch`؟) التي يحتاجها محرك الترجمة لبناء استعلام SQL مكافئ.
    

### 2. غياب الـ Metadata في الـ Delegates

عندما تمرر `Func<T, bool>` (كود عادي)، أنت تمرر **Pointer** لعنوان في الذاكرة يحتوي على الكود القابل للتنفيذ.

- **المشكلة:** الـ `Delegate` لا يحمل معه "Metadata" تصف محتواه. هو مجرد أمر "نفذني". محرك الـ LINQ Provider في EF Core يحتاج إلى هيكل شجري (Object Model) لكي يحلله. بما أن الـ Delegate لا يوفر هذا الهيكل، يضطر الـ EF Core للتوقف لأنه لا يملك صلاحية "اختراق" الذاكرة لقراءة ما بداخل الدالة.
    

### 3. معضلة الـ SQL Dialects (التباين البرمجي)

لغة C# هي لغة General-Purpose، بينما SQL هي لغة Set-based. هناك فجوة منطقية تسمى **Impedance Mismatch**.

- **المشكلة:** الـ EF Core يعمل بنظام الـ **Pattern Matching**. لديه خريطة تقول: "إذا وجدت عقدة `MethodCall` تشير إلى `string.Contains` في C#، حولها إلى `LIKE %x%` في SQL".
    
- في حالة الـ **Value Objects** أو الـ **Custom Methods**، أنت تنشئ "أنماطاً جديدة" (New Patterns) غير موجودة في خريطة الـ Provider. المترجم ليس لديه "ذكاء اصطناعي" ليستنتج أن دالتك `IsExpensive()` هي مجرد اختصار لـ `Amount > 1000`. هو يراها كـ `Unknown Pattern`.
    

### 4. قيود الـ Sandbox في محرك قاعدة البيانات

قاعدة البيانات تعمل في بيئة معزولة (Isolated Environment).

- **لماذا لا يمكننا إرسال الدالة كما هي؟** لأن الـ SQL Server لا يملك بيئة تشغيل .NET (إلا في حالات نادرة جداً ومكلفة كـ SQL CLR). لكي يتم تنفيذ `Regex` أو دالة `Decrypt` خارجية، يجب أن يتم ذلك في بيئة الأبلكيشن. الـ EF Core يتخذ قراراً هندسياً صارماً: **إما أن يُنفذ الاستعلام بالكامل على السيرفر (للكفاءة)، أو يرفض التنفيذ**، بدلاً من الدخول في نفق الـ Client-side evaluation المظلم.
    

---

## 🗒️ الـ Notes التقنية المنهجية (The Technical Logic)

|**الظاهرة**|**التوصيف التقني (Technical Reason)**|**التأثير على الـ Query Pipeline**|
|---|---|---|
|**Custom Methods**|تفتقر إلى الـ Expression Metadata؛ تظهر كـ `Opaque Node` (عقدة معتمة).|يتوقف الـ `Query Compiler` عند محاولة حل الـ `MethodCallExpression`.|
|**Value Object Logic**|الـ Mapping يتم للبيانات (Data Members) فقط، والـ Encapsulated Logic ليس جزءاً من الـ Model Snapshot.|لا يوجد `Translation Mapping` للدوال داخل الـ Value Objects في الـ `IModel`.|
|**External Libraries**|الأكواد تكون `Pre-compiled`؛ الـ Provider لا يستطيع الوصول للـ `Expression Tree` الخاص بها.|فشل في الـ `Expression Visitor` أثناء محاولة "تفكيك" العقدة الخارجية.|
|**Regex/Complex Math**|عدم وجود "Equivalent Function" في الـ Target Database Provider.|الـ Provider يرمي `NotSupportedException` لعدم وجود ترجمة في الـ `SQL Map`.|
