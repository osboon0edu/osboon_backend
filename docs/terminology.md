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

**Workspace** هي الحاوية التنظيمية التي يعيش داخلها عمل Account وCourses الخاصة به. يبدأ استخدامها ضمن معمارية **Local-First** على جهاز المستخدم، ويمكن مزامنة بياناتها الخاصة سحابيًا عند اختيار Cloud Sync.

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

**Public Library** هي مكتبة تعاونية عامة مسؤولة عن نشر المقررات التعليمية ومشاركتها واكتشافها وتحديثها والتحقق من التحديثات وإدارة إصداراتها.

الأهداف الأساسية لـPublic Library:
- تمكين المستخدم من نشر مقرر مفيد للآخرين.
- جعل Public Library قناة المشاركة العامة للمقررات بين المستخدمين.
- تقليل تكرار المقررات المتشابهة.
- مساعدة الطلاب على العثور على المقرر المناسب لسياقهم بسرعة.
- تمكين المستخدم من إضافة مقرر المكتبة إلى Workspace دون نسخ الملفات المشتركة كاملة.
- السماح باقتراح تحديثات للمقررات المنشورة.
- استخدام مراجعة جماعية للتحقق من التحديثات قبل اعتمادها.
- الاحتفاظ بتاريخ الإصدارات والمساهمين والمراجعين.

بعد اعتماد المقرر في Public Library، تصبح الملفات المشتركة الخاصة به مملوكة للمكتبة، ويمكن لعدة Workspaces الإشارة إليها بدل إنشاء نسخ كاملة منها.

## 8. مصطلحات Public Library

### Published Course

**Published Course** هو Course تم نشره واعتماده في Public Library وأصبح متاحًا للمشاركة والاكتشاف والاستخدام.

### Library Course

**Library Course** هو Course كما هو متاح داخل Public Library، بما يشمل إصداراته المعتمدة وتاريخه ومساهميه ومراجعيه.

### Course Version

**Course Version** هو إصدار محدد ومعتمد من Course منشور في Public Library، ويمثل حالة المقرر المعتمدة في نقطة زمنية محددة.

الإصدار الجديد لا يعني إنشاء نسخة كاملة من Course؛ بل يحتوي على الملفات التي تغيرت، بينما يمكن إعادة استخدام الملفات غير المتغيرة.

### Current Version

**Current Version** هو أحدث Course Version معتمد حاليًا في Public Library.

### Version History

**Version History** هو سجل الإصدارات المعتمدة التي مر بها Course داخل Public Library.

### Version Update

**Version Update** هو تحديث مقترح لمحتوى Course منشور، يصبح Course Version جديدًا بعد اعتماده.

### Changed Content

**Changed Content** هو الملفات التعليمية التي تغيرت ضمن Course Version معين.

### Shared Content

**Shared Content** هو المحتوى الذي تديره Public Library ويمكن أن تشير إليه عدة Workspaces بدل إنشاء نسخة كاملة منه لكل مستخدم. يُقرأ هذا المحتوى مرجعيًا من الإصدار المثبت، ولا يلزم تكراره في قاعدة البيانات السحابية لمساحة العمل.

### Workspace Course Reference

**Workspace Course Reference** هو ارتباط Course داخل Workspace بإصدار محدد من Course موجود في Public Library، وليس نسخة كاملة مستقلة من جميع الملفات المشتركة.

### Pinned Version / Selected Version

**Pinned Version / Selected Version** هو الإصدار الذي اختاره المستخدم ليكون الإصدار المرتبط بالمقرر داخل Workspace.

المستخدم لا ينتقل إلى الإصدارات الأحدث تلقائيًا؛ يمكنه اختيار الانتقال إلى إصدار أحدث عندما يقرر ذلك.

### Private Modification

**Private Modification** هو تعديل يجريه المستخدم محليًا على محتوى مرتبط بـPublic Library داخل Workspace، ويصبح هذا التعديل خاصًا بالمستخدم، ويمكن مزامنته سحابيًا اختياريًا، إلى أن يقرر نشره كتحديث.

### Private Copy

**Private Copy** هي نسخة خاصة محلية من ملف تم تعديله داخل Workspace، ويمكن للمستخدم تعديلها أو حذفها، ويمكن مزامنتها سحابيًا اختياريًا، دون إنشاء Course Version في Public Library.

### Publish

**Publish** هي عملية نشر Course من Workspace إلى Public Library بغرض اعتماده وإتاحته للمستخدمين.

### Update Proposal

**Update Proposal** هو اقتراح المستخدم لتحديث Course منشور في Public Library.

### Review

**Review** هي عملية فحص Update Proposal قبل اعتماده.

### Reviewer

**Reviewer** هو مستخدم اختارته المنصة لمراجعة Update Proposal وقبل دعوة المراجعة.

### Review Invitation

**Review Invitation** هي دعوة ترسلها المنصة إلى مستخدم مختار للمشاركة في Review.

### Review Acceptance

**Review Acceptance** هو قبول المستخدم لدعوة Review، وبموجبه يصبح Reviewer لذلك التحديث.

### Collaborative Review

**Collaborative Review** هي مراجعة جماعية لـUpdate Proposal قبل اعتماده.

تختار المنصة المراجعين عشوائيًا من المستخدمين المؤهلين، ويمكن أن يكونوا Students أو Teachers.

### Approved Update

**Approved Update** هو Update Proposal اجتاز عملية Collaborative Review وأصبح معتمدًا لإنشاء Course Version جديد.

### Contributors

**Contributors** هم المستخدمون الذين ساهموا في محتوى Course أو في تحديثاته.

### Reviewers

**Reviewers** هم المستخدمون الذين شاركوا في Review لتحديثات Course.

### Version Adoption

**Version Adoption** هي إضافة المستخدم إصدارًا محددًا من Course في Public Library إلى Workspace الخاصة به.

### Version Upgrade

**Version Upgrade** هو انتقال المستخدم من الإصدار المرتبط حاليًا في Workspace إلى إصدار أحدث، بناءً على اختياره.

### Version Selection

**Version Selection** هو اختيار المستخدم لإصدار معين عند إضافة Course من Public Library أو عند الانتقال إلى إصدار آخر.

### Release Scope / Classification

**Release Scope / Classification** هي المعلومات التصنيفية المرتبطة بـCourse المنشور بهدف تسهيل اكتشافه والوصول إليه في سياقه المناسب، مثل الدولة أو الجامعة أو المستوى التعليمي أو السياق التعليمي.

تفاصيل هذه المعلومات التصنيفية تُحسم لاحقًا.

### Duplicate Course

**Duplicate Course** هو Course مستقل يتشابه في محتواه أو غرضه مع Course موجود في Public Library، وتقليل هذه الحالات أحد أهداف Public Library.

### Library Ownership

**Library Ownership** تعني انتقال ملكية الملفات المشتركة الخاصة بـCourse إلى Public Library بعد اعتماد نشره، مع تطهير سجلات البنية والمحتوى الخاصة المتزامنة سحابيًا للمقرر واستبدالها بارتباط `WorkspaceCourseReference` بالإصدار المعتمد.

### File-level Versioning

**File-level Versioning** هو أسلوب إدارة الإصدارات الذي لا يتطلب إنشاء نسخة كاملة من Course عند كل تحديث؛ يحتفظ الإصدار الجديد بالملفات التي تغيرت ويعيد استخدام الملفات غير المتغيرة.

### Version-specific Reference

**Version-specific Reference** هو ارتباط Workspace بإصدار محدد من Course في Public Library، بحيث يبقى المستخدم على الإصدار الذي اختاره ولا يتغير تلقائيًا عند صدور إصدار أحدث.

## 9. المصطلحات المرجعية

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
| Public Library | مكتبة تعاونية عامة لنشر ومشاركة واكتشاف وتحديث والتحقق من إصدارات Courses |
| Published Course | Course منشور ومعتمد في Public Library |
| Library Course | Course كما هو متاح داخل Public Library |
| Course Version | إصدار محدد ومعتمد من Course |
| Current Version | أحدث إصدار معتمد حاليًا |
| Version History | سجل الإصدارات المعتمدة |
| Version Update | تحديث مقترح ينتج Course Version جديدًا بعد اعتماده |
| Changed Content | الملفات التي تغيرت في إصدار معين |
| Shared Content | المحتوى المشترك الذي تديره Public Library |
| Workspace Course Reference | ارتباط Course في Workspace بإصدار محدد من Public Library |
| Pinned Version / Selected Version | الإصدار الذي اختاره المستخدم للارتباط به |
| Private Modification | تعديل خاص بالمستخدم على محتوى مرتبط بالمكتبة |
| Private Copy | نسخة خاصة من ملف معدل داخل Workspace |
| Publish | نشر Course من Workspace إلى Public Library |
| Update Proposal | اقتراح تحديث لـCourse منشور |
| Review | فحص تحديث مقترح قبل اعتماده |
| Reviewer | مستخدم قبل دعوة المراجعة وأصبح مشاركًا فيها |
| Review Invitation | دعوة للمشاركة في مراجعة |
| Review Acceptance | قبول دعوة المراجعة |
| Collaborative Review | مراجعة جماعية للتحديث قبل اعتماده |
| Approved Update | تحديث اجتاز المراجعة وأصبح مؤهلًا لإنشاء إصدار جديد |
| Contributors | المستخدمون الذين ساهموا في Course أو تحديثاته |
| Reviewers | المستخدمون المشاركون في مراجعة التحديثات |
| Version Adoption | إضافة إصدار محدد إلى Workspace |
| Version Upgrade | انتقال المستخدم إلى إصدار أحدث باختياره |
| Version Selection | اختيار إصدار محدد |
| Release Scope / Classification | المعلومات التصنيفية المستخدمة لتسهيل اكتشاف Course |
| Duplicate Course | Course مستقل مشابه لـCourse موجود في Public Library |
| Library Ownership | ملكية Public Library للملفات المشتركة بعد اعتماد النشر |
| File-level Versioning | إدارة الإصدارات على مستوى الملفات المتغيرة بدل نسخ Course كاملة |
| Version-specific Reference | ارتباط Workspace بإصدار محدد وعدم التحديث التلقائي |

## 10. قواعد استخدام المصطلحات

- لا نستخدم **Identity** للإشارة إلى Account.
- لا نستخدم **Account** للإشارة إلى Identity.
- لا نستخدم **Workspace** للإشارة إلى Account.
- لا نستخدم **Lesson** للدلالة على المحتوى التعليمي نفسه.
- نستخدم **Course → Unit → Lesson** عند وصف البنية التنظيمية.
- نستخدم **Educational Content** عند وصف الملفات التي تعيش داخل Lesson.
- نستخدم **Public Library** عند الحديث عن النشر والمشاركة والاكتشاف والتحديثات والإصدارات العامة.
- نستخدم **Course Version** للإشارة إلى إصدار معتمد، وليس إلى نسخة كاملة مستقلة من Course.
- نستخدم **Private Copy** عند الحديث عن نسخة خاصة من ملف عدله المستخدم داخل Workspace.
- لا نعتبر تعديل المستخدم الخاص Course Version جديدًا ما لم يُنشر التعديل ويُعتمد.
- لا يتم تحديث Workspace إلى Course Version أحدث تلقائيًا.
- عند الحديث عن ارتباط Workspace بإصدار محدد، نستخدم **Workspace Course Reference** أو **Version-specific Reference**.

## 11. نطاق هذه الوثيقة

هذه الوثيقة تثبت Terminology فقط.

لا تحسم:
- Domain Boundaries.
- States & Workflows.
- Entity / Relationship Classification.
- Logical Data Model.
- Backend.
- Database أو تفاصيل تنفيذ Supabase.
- التفاصيل النهائية لآلية اختيار المراجعين أو شروط اعتماد التحديث.
- التفاصيل النهائية لمعلومات Release Scope / Classification.
