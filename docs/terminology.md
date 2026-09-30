# المصطلحات الأساسية في مشروع عصبون | osboon

## 1. الهدف

هذه الوثيقة هي المرجع المعتمد للمصطلحات الأساسية المستخدمة في المشروع، وتهدف إلى توحيد معنى المصطلحات قبل الانتقال إلى مراحل التوثيق التالية.

هذه الوثيقة تحسم معنى المصطلحات، ولا تحسم تصميم الـBackend أو الـDatabase.

## 2. Identity / Account

### Identity

**Identity** هي هوية Authentication القادمة من **Supabase Auth**، وتمثل الهوية الأساسية للمستخدم داخل النظام.

الـIdentity ليست Account.

### Account

**Account** هو حساب يملكه Identity داخل المشروع.

يمكن للـIdentity نفسها أن تملك:
- Student Account.
- Teacher Account.
- أو كلاهما في مراحل مختلفة من حياة المستخدم.

كل Account له مساحات العمل الخاصة به.

### العلاقة

```text
Identity
├── Student Account
└── Teacher Account
```

ولا يعني وجود Student Account أو Teacher Account أن هناك Identity مختلفة.

## 3. Workspace

**Workspace** هي الحاوية التنظيمية التي يعيش داخلها عمل Account وCourses الخاصة به.

- كل Workspace تنتمي إلى Account واحد.
- يمكن لـAccount الواحد امتلاك عدة Workspaces.
- كل Workspace تحتوي Courses.
- الـCourses داخل Workspace قد تكون:
  - منشأة بواسطة المستخدم.
  - أو مرتبطة بـCourses موجودة في Public Library.

```text
Account
├── Workspace
│   ├── Course
│   └── Course
└── Workspace
    └── Course
```

## 4. Course

**Course** هي الحاوية التنظيمية الكبرى للمحتوى التعليمي أو المادة التعليمية بشكل عام.

Course هي الوحدة التي يمكن مشاركتها في **Public Library**.

وجود Course داخل Workspace لا يعني بالضرورة أن محتواها منشأ من الصفر؛ يمكن أن تكون Course منشأة بواسطة المستخدم أو مرتبطة بـCourse متاحة في Public Library.

## 5. Learning Structure

**Learning Structure** تصف البنية الهرمية التنظيمية للمحتوى التعليمي.

التسلسل المعتمد:

```text
Course
  ↓
Unit
  ↓
Lesson
```

هذه العناصر هي حاويات تنظيمية.

### Lesson

**Lesson** هي أصغر حاوية تنظيمية في Learning Structure.

Lesson ليست أصغر وحدة محتوى؛ المحتوى التعليمي يعيش داخلها.

## 6. Educational Content

**Educational Content** يصف الملفات التعليمية التي تعيش داخل Lesson.

الأنواع الأساسية في Core Domain هي:

| النوع | الصيغة |
|---|---|
| Source | `.md` |
| Objectives | `.json` |
| Flashcards | `.json` |
| Understanding Cards | `.json` |
| Lesson Quiz / Test | `.json` |

Educational Content منفصل مفاهيميًا عن Learning Structure.

## 7. Public Library

**Public Library** هي المكان الذي تتم فيه مشاركة Courses بين المستخدمين.

في هذه المرحلة نثبت فقط علاقتها الأساسية بـCourse، بينما تفاصيل Public Library نفسها تُحسم في توثيق مستقل لاحقًا.

## 8. المصطلحات المرجعية

المصطلحات المعتمدة في هذه المرحلة:

| المصطلح | المعنى المرجعي |
|---|---|
| Identity | هوية Authentication القادمة من Supabase |
| Account | حساب تملكه Identity |
| Student Account | حساب يمثل المسار التعليمي للطالب |
| Teacher Account | حساب يمثل مساحة العمل المهنية/التعليمية للمعلم |
| Workspace | حاوية تنظيمية مملوكة لـAccount |
| Course | الحاوية التنظيمية الكبرى للمحتوى التعليمي |
| Learning Structure | البنية الهرمية Course → Unit → Lesson |
| Unit | حاوية تنظيمية داخل Course |
| Lesson | أصغر حاوية تنظيمية داخل Learning Structure |
| Educational Content | الملفات التعليمية التي تعيش داخل Lesson |
| Public Library | نطاق مشاركة Courses بين المستخدمين |

## 9. قواعد استخدام المصطلحات

- لا نستخدم **Identity** للإشارة إلى Account.
- لا نستخدم **Account** للإشارة إلى Identity.
- لا نستخدم **Workspace** للإشارة إلى Account.
- لا نستخدم **Lesson** للدلالة على المحتوى التعليمي نفسه.
- نستخدم **Course → Unit → Lesson** عند وصف البنية التنظيمية.
- نستخدم **Educational Content** عند وصف الملفات التي تعيش داخل Lesson.
- نستخدم **Public Library** عند الحديث عن مشاركة Courses بين المستخدمين.

## 10. نطاق هذه الوثيقة

هذه الوثيقة تثبت Terminology فقط.

لا تحسم:
- Domain Boundaries.
- States & Workflows.
- Entity / Relationship Classification.
- Logical Data Model.
- Backend.
- Database أو تفاصيل تنفيذ Supabase.