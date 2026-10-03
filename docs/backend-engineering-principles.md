# مبادئ هندسة الـBackend وإدارة قاعدة البيانات | Backend Engineering Principles

## 1. الهدف

تحدد هذه الوثيقة المبادئ الهندسية الحاكمة لإدارة وبناء الـBackend في مشروع **عصبون | osboon**، وبالأخص العلاقة بين **Git Repository** و**Database Schema** و**Migrations** و**GitHub CI/CD** وبيئات الـSupabase.

هذه المبادئ تمثل **Engineering Constraints** للمراحل الحالية والقادمة، وتُستخدم كمرجع عند تصميم وتنفيذ الـBackend، ولا تُعد خطة تنفيذ تفصيلية لمرحلة بعينها.

تهدف الوثيقة إلى تثبيت الفلسفة التي تقوم عليها إدارة الـBackend بحيث لا تُعاد مناقشتها أو افتراضها ضمن كل مرحلة تنفيذية لاحقة، إلا عند وجود قرار موثق يغيّرها.

---

## 2. النطاق

تنطبق هذه المبادئ على **Backend Database Management** ودورة تغييرات الـBackend المرتبطة بها.

ولا تعني هذه الوثيقة إلغاء أو تغيير معمارية **Local-First** الخاصة بتطبيق العميل (**Flutter + Drift**) المعتمدة في `docs/backend-architecture.md`.

بالتالي:

- **Drift / Local DB** في Client Tier يبقى جزءًا من معمارية العميل.
- **Persistent Local Backend Database** ليست جزءًا من معمارية إدارة الـBackend في المشروع.
- الـTemporary Database التي تنشأ داخل CI للاختبارات والتحقق لا تُعد Persistent Local Backend Database.

---

## 3. المبادئ الأساسية

### 3.1 Git Repository هو Source of Truth

يُعتبر **Git Repository** المرجع الأساسي لإدارة حالة وتعريف قاعدة بيانات الـBackend.

ويشمل ذلك على الأقل:

- Declarative Database Schema.
- Database Migrations.
- ملفات الاختبارات المرتبطة بقاعدة البيانات.
- Configuration اللازمة لإدارة دورة قاعدة البيانات عندما تكون جزءًا من الـRepository.

لا تُعتبر قاعدة بيانات Supabase السحابية مصدر الحقيقة الذي تُبنى منه تغييرات المشروع.

**Supabase Cloud Database هي Deployment Target وليست Source of Truth.**

---

### 3.2 Database Schema وMigrations تُدار كـCode / Versioned Artifacts

يجب أن تكون تغييرات قاعدة البيانات:

- قابلة للتتبع عبر Git.
- مرتبطة بتاريخ وتغيير واضح.
- قابلة للمراجعة ضمن Pull Request.
- قابلة لإعادة التحقق آليًا.
- قابلة للنشر عبر دورة CI/CD المعتمدة.

لا تعتمد إدارة قاعدة البيانات على تغييرات يدوية غير ممثلة في Repository.

---

### 3.3 لا توجد Persistent Local Backend Database

لا يدير المشروع **Persistent Local Database للـBackend** في بيئة المطور.

ولا تُعتبر بيئة التطوير المحلية مصدرًا مستقلًا لحالة قاعدة بيانات الـBackend.

الهدف هو أن تكون دورة إدارة قاعدة البيانات قابلة لإعادة البناء والتحقق من خلال Repository وCI/CD بدل الاعتماد على حالة قاعدة بيانات محلية مستمرة.

> هذا المبدأ خاص بقاعدة بيانات الـBackend، ولا يلغي استخدام التطبيق العميل لـDrift وفق معمارية Client Tier المعتمدة.

---

### 3.4 Database Lifecycle يُدار عبر GitHub CI/CD

يجب أن تمر عمليات إدارة دورة حياة قاعدة بيانات الـBackend عبر **GitHub CI/CD**.

ويشمل ذلك بحسب المرحلة وحاجة النظام:

- Schema Validation.
- Migration Generation / Validation.
- Database Testing.
- Deployment إلى بيئات Supabase السحابية.
- التحقق من حالة التغييرات قبل وبعد النشر عندما يكون ذلك مطلوبًا.

الهدف هو أن تكون إدارة قاعدة البيانات **Automated, Reproducible, Traceable**.

---

### 3.5 Temporary Database داخل CI مسموحة للاختبارات والتحقق فقط

يُسمح بإنشاء **Temporary / Ephemeral Database** داخل بيئة CI عند الحاجة إلى:

- Schema Validation.
- Database Linting.
- Database Tests.
- التحقق من تطبيق الـMigrations.
- أي Validation آلي يتطلب قاعدة بيانات فعلية مؤقتة.

هذه القاعدة المؤقتة:

- تُنشأ داخل CI.
- تُستخدم لغرض الاختبار أو التحقق.
- لا تمثل بيئة تشغيل دائمة.
- لا تمثل Source of Truth.
- لا تتحول إلى Persistent Local Backend Database.

---

### 3.6 لا توجد إدارة يدوية مباشرة لحالة قاعدة البيانات كمسار اعتيادي

لا تعتمد دورة المشروع على قيام المطور بتعديل حالة Database السحابية يدويًا كجزء من المسار الاعتيادي لتغيير الـBackend.

التغيير المعتمد يجب أن يبدأ من Repository، ويمر عبر المراجعة والـCI/CD، ثم يصل إلى البيئة المستهدفة من خلال آلية النشر المعتمدة.

إذا ظهرت حاجة استثنائية إلى إجراء يدوي، فلا يُفترض اعتباره جزءًا من الـWorkflow الاعتيادي، ويجب توثيق القرار والمسار المناسب له قبل اعتماده.

---

### 3.7 GitHub Actions هو Automation Path المعتمد

تُستخدم **GitHub Actions** كمسار الأتمتة المعتمد لإدارة عمليات CI/CD الخاصة بالـBackend Database.

وتُحدد Workflows التفصيلية لاحقًا ضمن مرحلة تأسيس وتنفيذ الـCI/CD، ولا تفرض هذه الوثيقة أسماء Workflows أو Branch Strategy أو تفاصيل تنفيذية لم يتم اعتمادها بعد.

---

### 3.8 Cloud Environments تُدار من خلال CI/CD

تُعتبر بيئات **Staging** و**Production** في Supabase بيئات تشغيل سحابية مستهدفة للنشر.

ويجب أن تتم إدارة التغييرات عليها من خلال مسار CI/CD المعتمد، مع استخدام آليات GitHub Environments / Secrets المناسبة لحماية بيانات الوصول وإدارة إعدادات كل بيئة.

لا تعني هذه الوثيقة اعتماد Branch Strategy محددة مثل `develop` أو غيرها؛ تفاصيل Branching وPromotion تُحسم ضمن تصميم CI/CD المعتمد.

---

### 3.9 Secrets وCredentials ليست جزءًا من Repository

لا تُخزن:

- Database Passwords.
- Access Tokens.
- Credentials.
- Signing Secrets.
- أي أسرار تشغيلية أخرى

داخل Repository.

تُدار هذه القيم من خلال آلية Secrets / Environments المعتمدة في GitHub، وفق احتياجات كل Workflow وEnvironment.

---

## 4. دورة إدارة قاعدة البيانات

الفلسفة العامة لدورة التغيير هي:

```text
Repository
    │
    ▼
Pull Request
    │
    ▼
GitHub CI/CD
    │
    ├── Schema Validation
    ├── Migration Validation / Generation
    ├── Temporary DB
    ├── Database Tests
    │
    ▼
Approved Change
    │
    ▼
Cloud Deployment
    │
    ├── Staging
    └── Production
```

يمثل هذا المخطط **المبدأ العام** وليس Workflow تنفيذيًا نهائيًا.

تفاصيل:

- Branch Strategy.
- Environment Promotion.
- Required Reviews.
- Workflow Triggers.
- Migration Generation.
- Deployment Commands.
- Rollback / Recovery Procedures.

تُحدد ضمن وثائق وIssues التنفيذية المناسبة، ولا تُستنتج من هذه الوثيقة دون قرار موثق.

---

## 5. ما الذي تضمنه هذه المبادئ

تؤسس هذه المبادئ للخصائص التالية في إدارة الـBackend:

### Traceability

كل تغيير في Database Schema أو Migration يجب أن يكون قابلاً للتتبع داخل Git.

### Reproducibility

يجب أن يمكن إعادة بناء والتحقق من حالة قاعدة البيانات من الـRepository والـCI/CD دون الاعتماد على حالة محلية مستمرة.

### Automation

يجب أن تكون عمليات التحقق والاختبار والنشر قابلة للأتمتة عبر GitHub Actions.

### Reviewability

تغييرات قاعدة البيانات يجب أن تمر عبر Pull Request ومراجعة المشروع المعتمدة.

### Environment Separation

تبقى بيئات الاختبار المؤقتة وبيئات Staging وProduction منفصلة في الغرض والدورة التشغيلية.

### Controlled Deployment

لا تنتقل تغييرات قاعدة البيانات إلى Cloud Environment إلا من خلال مسار النشر المعتمد.

---

## 6. حدود هذه الوثيقة

لا تحدد هذه الوثيقة:

- تصميم Logical Data Model.
- تصميم RLS Policies.
- تفاصيل Authorization.
- تصميم جداول PostgreSQL.
- أسماء أو تقسيمات Migrations.
- أسماء Branches النهائية.
- تفاصيل GitHub Actions Workflows.
- تفاصيل Supabase Project Configuration.
- تفاصيل Secrets الفعلية.
- آلية Rollback النهائية.
- Deployment Strategy تفصيلية لم تعتمد بعد.
- تفاصيل API أو Edge Functions.
- تفاصيل Storage.
- أي Feature أو Business Logic.

هذه الموضوعات تُحسم ضمن الوثائق والـIssues والـRoots المختصة بها.

---

## 7. العلاقة مع Backend Architecture

تعمل هذه المبادئ جنبًا إلى جنب مع:

`docs/backend-architecture.md`

وتكملها من زاوية **Engineering & Operational Governance**.

يمكن تلخيص العلاقة كالتالي:

| المرجع | المسؤولية |
|---|---|
| `docs/backend-architecture.md` | كيف تتكون معمارية الـBackend وما حدود مكوناتها |
| `docs/backend-engineering-principles.md` | ما المبادئ الهندسية التي تحكم إدارة وبناء الـBackend |
| Backend Handoff | ما الذي يتم تسليمه من Root #1 إلى Root #2 |
| Root #2 | كيف تُنفذ مرحلة Backend Foundation ضمن هذه القيود |

---

## 8. العلاقة مع Root #2

يُعتبر هذا المستند أحد **Architectural / Engineering Constraints** التي يجب أن يستلمها **Root #2 — Backend Foundation**.

عند تنفيذ Root #2:

- لا يُعاد تصميم هذه المبادئ ضمن Scope التنفيذ العادي.
- لا يُفترض وجود Persistent Local Backend Database.
- يجب أن يكون Git هو Source of Truth لتغييرات Database Schema وMigrations.
- يجب أن تكون Database Validation / Testing / Deployment جزءًا من مسار CI/CD المعتمد.
- يجوز استخدام Temporary Database داخل CI لأغراض الاختبار والتحقق.
- يجب أن تبقى تفاصيل التنفيذ التي لم تُحسم بعد ضمن Issues ووثائق Root #2 أو المراحل اللاحقة المناسبة.

إذا ظهرت حاجة لتغيير أحد هذه المبادئ، يجب توثيق التغيير واعتماده كقرار جديد قبل اعتباره جزءًا من الـArchitecture المعتمدة.

---

## 9. العلاقة مع المرجع الخارجي

تمت مراجعة `suhaib0edu/supabase-cicd-template` كمرجع هندسي لفهم نمط إدارة Supabase عبر GitHub CI/CD.

تم اعتماد **المبادئ العامة** التي يدعمها المرجع، ومنها:

- Git كـSource of Truth.
- عدم الحاجة إلى Persistent Local Database.
- استخدام Temporary Local Supabase داخل CI للتحقق والاختبارات.
- إدارة Staging وProduction عبر CI/CD.
- استخدام GitHub Environments / Secrets لإدارة بيانات الوصول.

ولا يعني ذلك اعتماد جميع تفاصيل الـTemplate أو نسخ Branch Strategy أو أسماء Workflows أو أوامر التنفيذ منه.

يبقى الـTemplate **Reference** وليس مصدرًا ملزمًا لتفاصيل تنفيذ osboon.

---

## 10. قاعدة التغيير

هذه الوثيقة جزء من المرجع الهندسي للمشروع.

أي تغيير في أحد المبادئ الأساسية الواردة فيها يجب أن:

1. يكون موثقًا.
2. يوضح سبب التغيير وأثره.
3. يراجع الوثائق والـIssues المتأثرة.
4. يمر عبر دورة المراجعة والاعتماد المعتمدة.
5. يُدمج في `main` قبل اعتباره مبدأً جديدًا للمشروع.

لا يجوز إدخال تغيير جوهري في هذه المبادئ بشكل ضمني أثناء تنفيذ Issue أخرى.
