# خارطة طريق مشروع عصبون | osboon

## 1. الهدف

توضح هذه الوثيقة الصورة العامة لمراحل مشروع **عصبون | osboon** على مستوى المراحل الكبرى، وتحدد العلاقة العامة بينها قبل الانتقال إلى التنفيذ التفصيلي.

هذه الوثيقة هي **Roadmap** للمشروع، وليست بديلًا عن الـRoot Issues أو الـParent/Child Issues.

## 2. مبادئ استخدام الـRoadmap

- توضح الـRoadmap المراحل الكبرى وتسلسلها العام.
- لا تحتوي على تفاصيل التنفيذ اليومية أو الـChild Issues.
- لا تحتوي على Branches أو خطوات تنفيذية خاصة بمهمة محددة.
- الـRoot Issue هي المرجع التنفيذي للمرحلة الكبرى الحالية.
- `docs/project-management/issues.md` هي المرجع لقواعد إدارة الـIssues والـBranches والـPull Requests.
- تُنشأ الـIssues تدريجيًا وفق قاعدة **Progressive Issue Creation**.
- يمكن أن تتغير تفاصيل المراحل المستقبلية عند توفر معلومات أو قرارات جديدة، دون تحويل الـRoadmap إلى قائمة مهام تفصيلية.

## 3. الصورة العامة للمراحل الكبرى

```text
Project
│
├── Foundation
│   └── Root #1 — تأسيس مرحلة التوثيق وإدارة العمل قبل بناء الـBackend
│
├── Backend Foundation
│   └── Root #2 — تأسيس الـSupabase Backend
│
├── Identity & Access
│   └── Root #3 — بناء نطاق الهوية والوصول
│
├── Core Domain
│   └── Root #4 — بناء النطاق الأساسي للمجال
│
├── Collaborative Review
│   └── Root #5 — بناء نطاق المراجعة التعاونية والاعتماد
│
├── Services & Integrations
│   └── Root #6 — بناء الخدمات والتكاملات
│
└── Testing & Production Readiness
    └── Root #7 — الاختبارات والأمان والجاهزية للإنتاج
```

> أرقام الـRoot Issues المستقبلية في هذه الوثيقة تمثل الترتيب المقترح للمراحل الكبرى، ولا تعني أن هذه الـIssues قد أُنشئت أو اعتُمدت بعد.

## 4. Foundation — Root #1

### تأسيس مرحلة التوثيق وإدارة العمل قبل بناء الـBackend

تهدف هذه المرحلة إلى تثبيت الأساس المرجعي والتنظيمي قبل بدء بناء الـBackend.

تشمل على المستوى العالي:

- قواعد إدارة العمل والتوثيق.
- Terminology.
- Domain Boundaries.
- States & Workflows.
- Entity / Relationship Classification.
- Final Alignment.
- Logical Data Model.
- Data Access & Security Model.
- Backend Architecture & Handoff.

نقطة الخروج من هذه المرحلة:

**مرجع معماري ومفاهيمي ومنطقي واضح يسمح بالانتقال المنظم إلى بناء الـBackend على Supabase.**

## 5. Backend Foundation — Root #2

### تأسيس الـSupabase Backend

تهدف هذه المرحلة إلى تأسيس قاعدة التنفيذ الخاصة بالـBackend بعد اكتمال الأساس المعماري والمنطقي.

على المستوى العالي تشمل:

- تأسيس بنية مشروع الـSupabase.
- تأسيس منهجية إدارة تغييرات قاعدة البيانات والـBackend.
- ربط دورة العمل مع CI/CD المعتمدة للمشروع.
- تأسيس الأساس المشترك الذي ستبنى عليه نطاقات الـBackend اللاحقة.

هذه المرحلة تعتمد على مخرجات Root #1، ولا يُفترض فيها إعادة تصميم الأساس المفاهيمي أو المنطقي الذي تم اعتماده سابقًا إلا إذا ظهر قرار موثق يستدعي ذلك.

## 6. Identity & Access — Root #3

### بناء نطاق الهوية والوصول

تهدف هذه المرحلة إلى تنفيذ نطاق Identity & Access كأساس لاستخدام بقية أجزاء النظام.

على المستوى العالي تشمل:

- Identity.
- Account.
- Student Account.
- Teacher Account.
- Access control والسياسات المرتبطة بها.
- تكامل هذا النطاق مع حدود الـDomain والـSecurity Model المعتمد.

## 7. Core Domain — Root #4

### بناء النطاق الأساسي للمجال

تهدف هذه المرحلة إلى بناء النطاقات الأساسية المرتبطة بالهيكل التعليمي والمحتوى ومساحات العمل والمكتبة العامة.

على المستوى العالي تشمل:

- Workspace.
- Learning Structure.
- Educational Content.
- Public Library & Versioning.
- العلاقات والقواعد الحاكمة التي تم تثبيتها في المرحلة السابقة.

وتبقى التفاصيل التنفيذية وتقسيم Parent/Child Issues خارج هذه الـRoadmap.

## 8. Collaborative Review — Root #5

### بناء نطاق المراجعة التعاونية والاعتماد

تهدف هذه المرحلة إلى تنفيذ المسار الخاص بمراجعة مقترحات التحديث واعتمادها.

على المستوى العالي تشمل:

- Update Proposal.
- Review Invitation.
- Reviewer Assignment.
- Review Feedback / Vote.
- Approved Update.
- Attribution Record.
- القواعد التنفيذية اللازمة لمسار المراجعة والاعتماد.

وتُحسم التفاصيل المتعلقة بـQuorum وConsensus وDiffing وMerge Conflict وفق المرحلة التنفيذية المناسبة، ولا تُفترض تفاصيلها داخل الـRoadmap.

## 9. Services & Integrations — Root #6

### بناء الخدمات والتكاملات

تهدف هذه المرحلة إلى بناء طبقة الخدمات والتكاملات المطلوبة فوق النطاقات الأساسية التي تم تنفيذها.

على المستوى العالي تشمل:

- Business Logic & Services.
- العمليات التي تتجاوز حدود قاعدة البيانات وحدها.
- التكاملات الداخلية والخارجية المطلوبة.
- الوظائف والخدمات اللازمة لدعم Workflows المعتمدة.

يُحدد التقسيم التفصيلي لهذه المرحلة لاحقًا بناءً على احتياجات النظام الفعلية.

## 10. Testing & Production Readiness — Root #7

### الاختبارات والأمان والجاهزية للإنتاج

تهدف هذه المرحلة إلى التأكد من جاهزية الـBackend والمنتج للانتقال إلى الاستخدام التشغيلي.

على المستوى العالي تشمل:

- Testing.
- Security validation.
- مراجعة سياسات الوصول والبيانات.
- التحقق من Workflows الأساسية.
- التحقق من التكاملات والخدمات.
- Production readiness.
- التحقق من آلية النشر والتشغيل المعتمدة.

لا تعني هذه المرحلة أن كل الاختبارات أو تفاصيل الأمان تؤجل إلى نهايتها؛ بل تُبنى ممارسات الجودة والأمان ضمن المراحل السابقة، بينما تمثل هذه المرحلة مراجعة شاملة للجاهزية.

## 11. مراحل مستقبلية بعد الأساس التشغيلي

بعد اكتمال مراحل الـBackend الأساسية، يمكن إضافة Root Issues جديدة حسب نطاقات المنتج التي يتم اعتمادها.

من الأمثلة على ذلك:

- Learning Execution & Study Progress.
- Spaced Repetition.
- نطاقات أو Features جديدة يقررها المنتج لاحقًا.

هذه المراحل لا تُثبت حاليًا كـIssues تنفيذية، ولا تُنشأ قبل توفر Scope واضح وDependencies معروفة.

## 12. Dependencies العامة بين المراحل

```text
Root #1
Foundation
   │
   ▼
Root #2
Backend Foundation
   │
   ▼
Root #3
Identity & Access
   │
   ▼
Root #4
Core Domain
   │
   ▼
Root #5
Collaborative Review
   │
   ▼
Root #6
Services & Integrations
   │
   ▼
Root #7
Testing & Production Readiness
   │
   ▼
Future Product Domains / Features
```

هذه Dependencies تمثل الصورة العامة فقط. يمكن أن توجد داخل كل Root مراحل متوازية أو Dependencies أدق، ويحددها الـRoot Issue الخاصة بها.

## 13. علاقة الـRoadmap بالوثائق وIssues

| العنصر | دوره |
|---|---|
| `docs/roadmap.md` | الصورة العامة للمراحل الكبرى وتسلسلها العام |
| Root Issue | Execution Plan للمرحلة الكبرى الحالية |
| Parent Issue | نطاق منطقي داخل الـRoot |
| Child Issue | وحدة عمل قابلة للتنفيذ والمراجعة بشكل مستقل |
| `docs/project-management/issues.md` | قواعد إدارة دورة العمل |
| الوثائق المعتمدة | المرجع المفاهيمي والمعماري للنظام |

### القاعدة الأساسية

**الـRoadmap يجيب: إلى أين يتجه المشروع؟**

**الـRoot Issue يجيب: ماذا ننفذ الآن، وبأي ترتيب؟**

**الـParent/Child Issues تجيب: ما وحدات العمل التي تنفذ المرحلة الحالية؟**

## 14. نقطة الانتقال الحالية

المشروع أنهى مراحل التأسيس والتوثيق السابقة داخل Root #1 حتى **Final Alignment**.

المرحلة التالية داخل Root #1 هي:

**Logical Data Model**

وبعد استكمال المراحل المتبقية من Root #1 واعتمادها، يكون الانتقال إلى **Root #2 — Backend Foundation** هو الانتقال الرئيسي التالي.
