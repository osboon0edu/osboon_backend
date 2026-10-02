# وثيقة نموذج البيانات المنطقي | Logical Data Model — مشروع عصبون (osboon)

## 1. الهدف ونطاق النموذج (Objective & Scope)

تهدف هذه الوثيقة إلى تحويل الأساس المفاهيمي المعتمد في المراحل السابقة (المصطلحات، حدود النطاقات، الحالات ومسارات العمل، تصنيف الكيانات والعلاقات، والمحاذاة الشاملة) إلى **نموذج بيانات منطقي (Logical Data Model)** دقيق ومتكامل، يحدد:
- الكيانات المنطقية (Logical Entities) وحقولها (Attributes).
- المفاتيح الأساسية (Primary Keys) والقيود الفريدة (Unique Constraints).
- المفاتيح الأجنبية (Foreign Keys) والعلاقات المنطقية والتعددية (Cardinality & Optionality).
- القواعد والقيود المنطقية الحاكمة لتكامل البيانات (Integrity Rules & Invariants).
- حسم النقاط الثلاث المؤجلة في مرحلة المحاذاة الشاملة.

> **حدود النطاق:** يقتصر هذا النموذج على المستوى المنطقي التحليلي فقط؛ ولا يتضمن أي تفاصيل تنفيذية تقنية مثل أوامر SQL أو سكريبتات الترحيل (Migrations)، أو أنواع بيانات PostgreSQL الخاصة، أو سياسات RLS، أو كود الـ Backend.

> **طبيعة النموذج والسياق السحابي (Local-First Context):**
> يمثل هذا النموذج قاعدة البيانات السحابية (Cloud Backend). يعتمد النظام فلسفة **Local-First**؛ حيث تتم معظم عمليات القراءة والدراسة والتعديل على قاعدة بيانات محلية في جهاز المستخدم. وتمثل كيانات `Workspace*` في هذا النموذج طبقة المزامنة السحابية الخاصة (Cloud Sync) لبيانات المستخدم المحلية، ولا تُنشأ أو تُحتفظ على الخادم إلا عندما يختار المستخدم مزامنة مسوداته أو تعديلاته الخاصة سحابيًا، تمهيدًا للنشر أو كنسخة احتياطية سحابية.

---

## 2. حسم النقاط المؤجلة من مرحلة المحاذاة (Resolution of Deferred Points)

### النقطة الأولى: نمذجة التصنيف وسياق النشر (Classification & Release Scope Schema)
- **القرار المنطقي:** اعتماد نموذج **التصنيف الهرمي متعدد الأبعاد (Faceted Hierarchical Taxonomy)** عبر كيانين منطقيين:
  1. `ClassificationTaxonomy`: يمثل عُقد التصنيف وسياقاتها (الدولة، المؤسسة/الجامعة، الكلية/التخصص، المستوى التعليمي) مع إمكانية التسلسل الذاتي الأبوي (`parent_id`) لتمثيل التبعية الطبيعية (مثال: جامعة تتبع دولة، كلية تتبع جامعة).
  2. `LibraryCourseClassification`: كيان ربط وتعيين متعدد الأطراف (`M : N`) يربط المقرر المنشور (`LibraryCourse`) بعُقد التصنيف المعنية.
- **التبرير:** يمنح النظام مرونة كاملة لتصنيف المقررات الجامعية، والمدرسية، والمهنية الحرة دون فرض أعمدة مخصصة جامدة لكل سياق، مع تجنب الغموض.

### النقطة الثانية: نمذجة إصدارات الملفات والمانيفست (File-Manifest / Version-Content Linkage)
- **القرار المنطقي:** تطبيق مبدأ **إدارة الإصدارات على مستوى الملفات المتغيرة (File-level Versioning)** عبر الفصل بين المستودع المشترك للملفات وبيان الإصدار (Manifest):
  1. `SharedContentItem`: يمثل المحتوى الفعلي للأصل التعليمي المعتمد والمملوك للمكتبة العامة (`Library Ownership`). هذا الكيان غير قابل للتعديل نهائيًا (`Immutable`) ومحايد تجاه رقم الإصدار.
  2. `VersionUnit`, `VersionLesson`, `VersionManifestItem`: تمثل البنية الهيكلية للإصدار وبيان محتواه، بحيث يشير كل `VersionManifestItem` إلى `VersionLesson` وإلى `SharedContentItem`، مع السماح بأكثر من ملف من النوع نفسه داخل الدرس، والتمييز بين الملفات عبر `title`.
- **التبرير:** يضمن عدم استنساخ الملفات غير المتغيرة عبر الإصدارات؛ فالإصدار `v1.1` يشير إلى نفس سجلات `SharedContentItem` الخاصة بـ `v1.0` للملفات الثابتة، بينما يسجل فقط إشارات جديدة للملفات المتغيرة (`Changed Content`).

### النقطة الثالثة: نمذجة الأنواع الخمسة للمحتوى التعليمي (Educational Content Typing)
- **القرار المنطقي:** اعتماد نمط **الكيان الموحد متعدد الأنواع (Single Logical Content Entity with Discriminator)** مع السماح بتعدد الملفات من النوع نفسه داخل الدرس.
- **التبرير:** تشترك ملفات المحتوى الخمسة (`Source`, `Objectives`, `Flashcards`, `Understanding Cards`, `Lesson Quiz/Test`) في نفس دورة الحياة وطرق التعديل والمزامنة والاستضافة داخل الدرس، مع التمييز بينها عبر `content_type`، بينما يميز `title` بين الملفات المتعددة من النوع نفسه. لا يُفرض قيد فريد على `content_type` وحده داخل الدرس.

---

## 3. قاموس الكيانات المنطقية (Logical Entities Data Dictionary)

### النطاق 1: نطاق الهوية والحسابات (Identity & Account Domain)

#### 1. `Identity`
- **الغرض المفاهيمي:** يمثل المستخدم المصادق عليه خارجيًا عبر Supabase Auth.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `email`: Text / Email, إلزامية, فريدة (UK).
  - `created_at`: Timestamp, إلزامية.

#### 2. `Account`
- **الغرض المفاهيمي:** الحساب الوظيفي الأساسي الذي يمثل الحاوية العامة للحساب، دون تحديد سياق الطالب أو المعلم داخله.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - لا توجد مفاتيح أجنبية مباشرة.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `display_name`: Text, إلزامية.
  - `status`: Enum (`ACTIVE`, `SUSPENDED`), إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - لا يخزن `Account` مرجع `Identity` مباشرة.
  - يحدد `StudentAccount` أو `TeacherAccount` السياق التعليمي للحساب ويربطه بالـ`Identity` المالكة.
  - يمكن للـ`Identity` نفسها امتلاك `StudentAccount` و`TeacherAccount` معًا، ولكل سياق `Account` مستقل.
  - **قيد التبعية الإلزامية:** كل سجل `Account` يجب أن يرتبط حتمًا بكيان `StudentAccount` واحد أو `TeacherAccount` واحد، ويُحظر وجود حساب عام لا يرتبط بهوية عبر أحد هذين السياقين.

#### 3. `StudentAccount`
- **الغرض المفاهيمي:** الحساب المتخصص في سياق الطالب والمملوك للـ `Identity` عبر حساب `Account` مستقل.
- **المفتاح الأساسي (PK):** `identity_id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `identity_id`: Identifier → يشير إلى `Identity.id`, إلزامية.
  - `account_id`: Identifier → يشير إلى `Account.id`, إلزامية, فريدة (UK).
- **السمات (Attributes):**
  - `identity_id`: Identifier, إلزامية (PK, FK).
  - `account_id`: Identifier, إلزامية (FK, UK).
- **القيود المنطقية (Constraints):**
  - يمكن للـ `Identity` امتلاك `StudentAccount` واحد فقط.
  - يجب أن يشير `account_id` إلى `Account` مملوك لنفس `Identity`.
  - `StudentAccount` و`TeacherAccount` سياقان مستقلان، ويمكن للـ `Identity` نفسها امتلاكهما معًا.

#### 4. `TeacherAccount`
- **الغرض المفاهيمي:** الحساب المتخصص في سياق المعلم والمملوك للـ `Identity` عبر حساب `Account` مستقل.
- **المفتاح الأساسي (PK):** `identity_id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `identity_id`: Identifier → يشير إلى `Identity.id`, إلزامية.
  - `account_id`: Identifier → يشير إلى `Account.id`, إلزامية, فريدة (UK).
- **السمات (Attributes):**
  - `identity_id`: Identifier, إلزامية (PK, FK).
  - `account_id`: Identifier, إلزامية (FK, UK).
- **القيود المنطقية (Constraints):**
  - يمكن للـ `Identity` امتلاك `TeacherAccount` واحد فقط.
  - يجب أن يشير `account_id` إلى `Account` مملوك لنفس `Identity`.
  - `StudentAccount` و`TeacherAccount` سياقان مستقلان، ويمكن للـ `Identity` نفسها امتلاكهما معًا.

### النطاق 2: نطاق مساحات العمل (Workspace Domain)

#### 5. `Workspace`
- **الغرض المفاهيمي:** الحاوية المعزولة لعمل الحساب ومقرراته.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `account_id`: Identifier → يشير إلى `Account.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `account_id`: Identifier, إلزامية (FK).
  - `name`: Text, إلزامية.
  - `description`: Text, اختيارية.
  - `status`: Enum (`ACTIVE`, `ARCHIVED`), إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.

#### 6. `WorkspaceCourse`
- **الغرض المفاهيمي:** التمثيل السحابي المتزامن للمقرر داخل مساحة العمل؛ يمثل النسخة السحابية من بيانات المستخدم المحلية عند تفعيل Cloud Sync، سواء كان المقرر منشأً محليًا أو مرتبطًا بالمكتبة. بالنسبة للمقرر المرتبط بالمكتبة، لا يتطلب هذا الكيان تكرار البنية والمحتوى المشتركين؛ يبقى كحاوية ارتباط، ويُقرأ المحتوى المرجعي من `WorkspaceCourseReference` و`CourseVersion`.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `workspace_id`: Identifier → يشير إلى `Workspace.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `workspace_id`: Identifier, إلزامية (FK).
  - `name`: Text, إلزامية.
  - `description`: Text, اختيارية.
  - `origin_type`: Enum (`LOCAL_AUTHOR`, `LIBRARY_REFERENCE`), إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.

#### 7. `WorkspaceCourseReference`
- **الغرض المفاهيمي:** ربط مقرر مساحة العمل بإصدار محدد ومثبت (`Pinned Version`) من المكتبة العامة.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `workspace_course_id`: Identifier → يشير إلى `WorkspaceCourse.id`, إلزامية, فريدة (UK - علاقة 1:1).
  - `library_course_id`: Identifier → يشير إلى `LibraryCourse.id`, إلزامية.
  - `pinned_version_id`: Identifier → يشير إلى `CourseVersion.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `workspace_course_id`: Identifier, إلزامية (FK, UK).
  - `library_course_id`: Identifier, إلزامية (FK).
  - `pinned_version_id`: Identifier, إلزامية (FK).
  - `last_synced_at`: Timestamp, إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - يجب أن يتبع `pinned_version_id` لنفس `library_course_id`.
  - `sync_status` ليست سمة مخزنة؛ تُشتق آنيًا بمقارنة `pinned_version_id` مع `LibraryCourse.current_version_id`:
    - `UP_TO_DATE`: عندما يتطابق الإصداران.
    - `UPDATE_AVAILABLE`: عندما يختلف الإصداران.

### النطاق 3: نطاق البنية التعليمية (Learning Structure Domain)

#### 8. `WorkspaceUnit`
- **الغرض المفاهيمي:** التمثيل السحابي المتزامن للوحدة التعليمية داخل مقرر مساحة العمل؛ تُحفظ على الخادم عندما يفعّل المستخدم Cloud Sync لمقرر خاص أو مسودة محلية تحتاج إلى مزامنة سحابية. لا تُنشأ لهذه الوحدات نسخة سحابية مكررة لمقرر مكتبة تتم قراءته مرجعيًا.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `workspace_course_id`: Identifier → يشير إلى `WorkspaceCourse.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `workspace_course_id`: Identifier, إلزامية (FK).
  - `title`: Text, إلزامية.
  - `order_index`: Integer, إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(workspace_course_id, order_index)`: ترتيب فريد للوحدات داخل المقرر.

#### 9. `WorkspaceLesson`
- **الغرض المفاهيمي:** التمثيل السحابي المتزامن لأصغر حاوية هيكلية في المقرر؛ يستضيف عناصر المحتوى التعليمي عند مزامنة بيانات مساحة العمل الخاصة. لا تُنشأ نسخة سحابية مكررة للدرس المشترك في مقرر مكتبة؛ تتم قراءة الدرس من `CourseVersion` محليًا وفق الإصدار المثبت.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `unit_id`: Identifier → يشير إلى `WorkspaceUnit.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `unit_id`: Identifier, إلزامية (FK).
  - `title`: Text, إلزامية.
  - `order_index`: Integer, إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(unit_id, order_index)`: ترتيب فريد للدروس داخل الوحدة.

### النطاق 4: نطاق المحتوى التعليمي (Educational Content Domain)

#### 10. `WorkspaceContentItem`
- **الغرض المفاهيمي:** التمثيل السحابي المتزامن لملف محتوى تعليمي محلي داخل `Workspace`؛ يُستخدم للمقررات المنشأة محليًا أو لحفظ التعديلات الخاصة المستقلة (`Local Override / Private Copy`) التي أنشأها المستخدم واختار مزامنتها سحابيًا. أما المحتوى المشترك لمقرر المكتبة فيُقرأ مرجعيًا ولا يُكرر داخل هذا الكيان.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `lesson_id`: Identifier → يشير إلى `WorkspaceLesson.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `lesson_id`: Identifier, إلزامية (FK).
  - `title`: Text, إلزامية.
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
  - `file_format`: Enum (`MARKDOWN`, `JSON`), إلزامية.
  - `storage_path`: String, إلزامية؛ يشير إلى الملف المحلي المستقل في `Object Storage`.
  - `checksum`: String, إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - يمكن أن يحتوي الدرس على أكثر من ملف من النوع نفسه.
  - `UNIQUE(lesson_id, content_type, title)`: لا يمكن تكرار ملف بنفس العنوان والنوع داخل الدرس الواحد.
  - لا يمثل هذا الكيان مرجعًا مباشرًا إلى `SharedContentItem`; قراءة المحتوى المشترك للمقرر المرتبط بالمكتبة تتم مرجعيًا عبر `WorkspaceCourseReference` و`CourseVersion`.

### النطاق 5: نطاق المكتبة العامة وإدارة الإصدارات (Public Library & Versioning Domain)

#### 11. `LibraryCourse`
- **الغرض المفاهيمي:** السجل الفهرسي العام المعتمد للمقرر المنشور في المكتبة.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `current_version_id`: Identifier → يشير إلى `CourseVersion.id`, اختيارية (تُحدث عند اعتماد أول إصدار وما يليه).
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `slug`: String, إلزامية, فريدة (UK).
  - `title`: Text, إلزامية.
  - `description`: Text, اختيارية.
  - `current_version_id`: Identifier, اختيارية (FK).
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.

#### 12. `CourseVersion`
- **الغرض المفاهيمي:** لقطة تاريخية معتمدة وغير قابلة للتعديل (`Immutable Snapshot`) لمقرر المكتبة.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `library_course_id`: Identifier → يشير إلى `LibraryCourse.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `library_course_id`: Identifier, إلزامية (FK).
  - `version_number`: String (مثل: '1.0.0'), إلزامية.
  - `release_notes`: Text, اختيارية.
  - `published_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(library_course_id, version_number)`: عدم تكرار رقم الإصدار لنفس المقرر.
  - **قيد عدم التعديل (Immutability Invariant):** بعد إنشاء السجل، يُحظر تعديل أي من سماته نهائيًا.

#### 13. `SharedContentItem`
- **الغرض المفاهيمي:** الأصول التعليمية المشتركة المستقرة المملوكة للمكتبة العامة (`Library Ownership`).
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
  - `file_format`: Enum (`MARKDOWN`, `JSON`), إلزامية.
  - `storage_path`: String, إلزامية، يشير إلى المحتوى الفعلي في `Object Storage`.
  - `checksum`: String, إلزامية, فريدة منطقيًا لمحتوى النوع.
  - `created_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - **قيد عدم التعديل (Immutability Invariant):** أصل معتمد وثابت، لا يُعدل ولا يُحذف إذا كان مرتبطًا بإصدار معتمد.

#### 14. `VersionUnit`
- **الغرض المفاهيمي:** الوحدة التعليمية كما تظهر داخل إصدار محدد من مقرر المكتبة.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `course_version_id`: Identifier → يشير إلى `CourseVersion.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `course_version_id`: Identifier, إلزامية (FK).
  - `title`: Text, إلزامية.
  - `order_index`: Integer, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(course_version_id, order_index)`: ترتيب فريد للوحدات داخل الإصدار.

#### 15. `VersionLesson`
- **الغرض المفاهيمي:** الدرس كما يظهر داخل وحدة محددة في إصدار مقرر المكتبة.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `version_unit_id`: Identifier → يشير إلى `VersionUnit.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `version_unit_id`: Identifier, إلزامية (FK).
  - `title`: Text, إلزامية.
  - `order_index`: Integer, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(version_unit_id, order_index)`: ترتيب فريد للدروس داخل الوحدة.

#### 16. `VersionManifestItem`
- **الغرض المفاهيمي:** كيان التقاطع الذي يربط درسًا داخل إصدار محدد بأصل محتوى مشترك، مع السماح بأكثر من ملف من النوع نفسه داخل الدرس.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `version_lesson_id`: Identifier → يشير إلى `VersionLesson.id`, إلزامية.
  - `shared_content_id`: Identifier → يشير إلى `SharedContentItem.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `version_lesson_id`: Identifier, إلزامية (FK).
  - `shared_content_id`: Identifier, إلزامية (FK).
  - `title`: Text, إلزامية.
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
- **القيود المنطقية (Constraints):**
  - يمكن أن يحتوي `VersionLesson` على أكثر من `VersionManifestItem` من النوع نفسه.
  - `UNIQUE(version_lesson_id, content_type, title)`: لا يمكن تكرار ملف بنفس العنوان والنوع داخل الدرس في الإصدار الواحد.
#### 17. `ClassificationTaxonomy`
- **الغرض المفاهيمي:** عُقد شجرة التصنيف والسياق التعليمي (الدول، الجامعات، التخصصات، المراحل).
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `parent_id`: Identifier → يشير ذاتيًا إلى `ClassificationTaxonomy.id`, اختيارية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `parent_id`: Identifier, اختيارية (FK).
  - `dimension`: Enum (`COUNTRY`, `INSTITUTION`, `FIELD_OF_STUDY`, `EDUCATION_LEVEL`), إلزامية.
  - `code`: String, إلزامية, فريدة (UK).
  - `name`: Text, إلزامية.
  - `created_at`: Timestamp, إلزامية.

#### 18. `LibraryCourseClassification`
- **الغرض المفاهيمي:** جدول تقاطع يربط مقرر المكتبة العامة بعُقد التصنيف والسياق.
- **المفتاح الأساسي (PK):** مركب (`library_course_id, taxonomy_id`)
- **المفاتيح الأجنبية (FK):**
  - `library_course_id`: Identifier → يشير إلى `LibraryCourse.id`, إلزامية.
  - `taxonomy_id`: Identifier → يشير إلى `ClassificationTaxonomy.id`, إلزامية.
- **السمات (Attributes):**
  - `library_course_id`: Identifier, إلزامية (PK, FK).
  - `taxonomy_id`: Identifier, إلزامية (PK, FK).
  - `created_at`: Timestamp, إلزامية.

### النطاق 6: نطاق المراجعة والاعتماد التعاوني (Collaborative Review Domain)

#### 19. `UpdateProposal`
- **الغرض المفاهيمي:** مقترح نشر مقرر جديد أو تحديث مقرر منشور موجود.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `library_course_id`: Identifier → يشير إلى `LibraryCourse.id`, اختيارية (NULL في حال النشر الأولي لمقرر جديد كليًا).
  - `source_workspace_course_id`: Identifier → يشير إلى `WorkspaceCourse.id`, إلزامية.
  - `author_account_id`: Identifier → يشير إلى `Account.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `library_course_id`: Identifier, اختيارية (FK).
  - `source_workspace_course_id`: Identifier, إلزامية (FK).
  - `author_account_id`: Identifier, إلزامية (FK).
  - `proposal_type`: Enum (`INITIAL_PUBLISH`, `VERSION_UPDATE`), إلزامية.
  - `proposed_version_number`: String, إلزامية.
  - `change_summary`: Text, إلزامية.
  - `status`: Enum (`DRAFT`, `SUBMITTED`, `UNDER_REVIEW`, `APPROVED`, `REJECTED`, `CANCELLED`), إلزامية.
  - `submitted_at`: Timestamp, اختيارية.
  - `resolved_at`: Timestamp, اختيارية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.

#### 20. `ProposalContentChange`
- **الغرض المفاهيمي:** تفصيل التغييرات المقترحة في المحتوى وموضعها الهيكلي تمهيدًا لاعتمادها.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `proposal_id`: Identifier → يشير إلى `UpdateProposal.id`, إلزامية.
  - `target_version_lesson_id`: Identifier → يشير إلى `VersionLesson.id`, اختيارية.
  - `source_workspace_lesson_id`: Identifier → يشير إلى `WorkspaceLesson.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `proposal_id`: Identifier, إلزامية (FK).
  - `target_version_lesson_id`: Identifier, اختيارية (FK).
  - `source_workspace_lesson_id`: Identifier, إلزامية (FK).
  - `title`: Text, إلزامية (عنوان الملف لتمييزه وتحديده بدقة).
  - `change_action`: Enum (`ADD`, `MODIFY`, `DELETE`), إلزامية.
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
  - `file_format`: Enum (`MARKDOWN`, `JSON`), إلزامية.
  - `storage_path`: String, اختيارية (إلزامية في حالتي `ADD` و`MODIFY`)، وتشير إلى المحتوى المقترح في `Object Storage`.
  - `proposed_checksum`: String, إلزامية.
  - `created_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `ADD` لملف يتبع درسًا قائمًا: يجب أن يكون `target_version_lesson_id IS NOT NULL` للإشارة إلى الدرس القائم.
  - `ADD` لملف يتبع درسًا جديدًا: يجب أن يكون `target_version_lesson_id = NULL`، ويستمد النظام عنوان الدرس وترتيبه من `source_workspace_lesson_id`.
  - `MODIFY` و`DELETE`: يجب أن يكون `target_version_lesson_id IS NOT NULL` للإشارة الصريحة إلى الدرس القائم في الإصدار المستهدف.
  - لا تُستخدم العناوين النصية (`unit_title`, `lesson_title`) كمرجع هيكلي للموقع.

#### 21. `ReviewInvitation`
- **الغرض المفاهيمي:** دعوة موجهة آليًا لمستخدم مؤهل لتقييم مقترح تحديث.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `proposal_id`: Identifier → يشير إلى `UpdateProposal.id`, إلزامية.
  - `account_id`: Identifier → يشير إلى `Account.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `proposal_id`: Identifier, إلزامية (FK).
  - `account_id`: Identifier, إلزامية (FK).
  - `status`: Enum (`PENDING`, `ACCEPTED`, `DECLINED`, `EXPIRED`), إلزامية.
  - `expires_at`: Timestamp, إلزامية.
  - `responded_at`: Timestamp, اختيارية.
  - `created_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(proposal_id, account_id)`: لا تُرسل أكثر من دعوة لنفس الحساب على نفس المقترح.

#### 22. `ReviewerAssignment`
- **الغرض المفاهيمي:** مهمة التحكيم النشطة التي تنشأ بمجرد قبول المراجع للدعوة وتوثق تقييمه.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `invitation_id`: Identifier → يشير إلى `ReviewInvitation.id`, إلزامية, فريدة (UK - علاقة 1:1).
  - `proposal_id`: Identifier → يشير إلى `UpdateProposal.id`, إلزامية.
  - `reviewer_account_id`: Identifier → يشير إلى `Account.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `invitation_id`: Identifier, إلزامية (FK, UK).
  - `proposal_id`: Identifier, إلزامية (FK).
  - `reviewer_account_id`: Identifier, إلزامية (FK).
  - `status`: Enum (`ACTIVE`, `COMPLETED`, `TIMED_OUT`), إلزامية.
  - `decision`: Enum (`APPROVE`, `REJECT`), اختيارية (تصبح إلزامية عند حالة `COMPLETED`).
  - `feedback_notes`: Text, اختيارية.
  - `deadline_at`: Timestamp, إلزامية.
  - `submitted_at`: Timestamp, اختيارية.
  - `created_at`: Timestamp, إلزامية.

#### 23. `CourseAttribution`
- **الغرض المفاهيمي:** السجل الدائم لحفظ حقوق المساهمين والمراجعين للإصدارات المعتمدة.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `course_version_id`: Identifier → يشير إلى `CourseVersion.id`, إلزامية.
  - `account_id`: Identifier → يشير إلى `Account.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `course_version_id`: Identifier, إلزامية (FK).
  - `account_id`: Identifier, إلزامية (FK).
  - `role`: Enum (`CONTRIBUTOR`, `REVIEWER`), إلزامية.
  - `created_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(course_version_id, account_id, role)`: تسجيل كل دور مرة واحدة للشخص في الإصدار الواحد.

---

## 4. شبكة العلاقات المنطقية وقواعد التكامل المرجعي (Logical Relationships Matrix)

| الكيان المصدر | العلاقة | الكيان الهدف | Cardinality | Optionality | Referential Action |
|---|---|---|---|---|---|
| `Identity` | تملك | `StudentAccount` | `1 : 0..1` | اختيارية | Restrict |
| `StudentAccount` | يشير إلى | `Account` | `1 : 1` | إلزامية | Restrict |
| `Identity` | تملك | `TeacherAccount` | `1 : 0..1` | اختيارية | Restrict |
| `TeacherAccount` | يشير إلى | `Account` | `1 : 1` | إلزامية | Restrict |
| `Workspace` | مملوكة لـ | `Account` | `N : 1` | إلزامية | Restrict على حذف `Account` |
| `WorkspaceCourse` | ينتمي لـ | `Workspace` | `N : 1` | إلزامية | Cascade عند حذف `Workspace` |
| `WorkspaceCourseReference` | يحدد ارتباط | `WorkspaceCourse` | `1 : 1` | إلزامية | Cascade عند حذف `WorkspaceCourse` |
| `WorkspaceCourseReference` | يشير إلى | `LibraryCourse` | `N : 1` | إلزامية | Restrict عند حذف `LibraryCourse` |
| `WorkspaceCourseReference` | يثبت إصدار | `CourseVersion` | `N : 1` | إلزامية | Restrict عند حذف `CourseVersion` |
| `WorkspaceUnit` | تتبع لـ | `WorkspaceCourse` | `N : 1` | إلزامية | Cascade عند حذف `WorkspaceCourse` |
| `WorkspaceLesson` | يتبع لـ | `WorkspaceUnit` | `N : 1` | إلزامية | Cascade عند حذف `WorkspaceUnit` |
| `WorkspaceContentItem` | يستضاف في | `WorkspaceLesson` | `N : 1` | إلزامية | Cascade عند حذف `WorkspaceLesson` |
| `CourseVersion` | يوثق إصدارًا لـ | `LibraryCourse` | `N : 1` | إلزامية | Restrict عند حذف `LibraryCourse` |
| `LibraryCourse` | يشير إلى أحدث إصدار معتمد | `CourseVersion` | `1 : 0..1` | اختيارية (عبر `current_version_id`) | Restrict عند حذف `CourseVersion` |
| `VersionUnit` | تتبع لـ | `CourseVersion` | `N : 1` | إلزامية | Restrict عند حذف `CourseVersion` |
| `VersionLesson` | تتبع لـ | `VersionUnit` | `N : 1` | إلزامية | Restrict عند حذف `VersionUnit` |
| `VersionManifestItem` | ينتمي لدرس | `VersionLesson` | `N : 1` | إلزامية | Restrict عند حذف `VersionLesson` |
| `VersionManifestItem` | يربط الأصل | `SharedContentItem` | `N : 1` | إلزامية | Restrict عند حذف `SharedContentItem` |
| `LibraryCourseClassification` | يربط تصنيف | `LibraryCourse` & `ClassificationTaxonomy` | `M : N` | إلزامية | Restrict عند حذف المرجع؛ إزالة الربط فقط |
| `UpdateProposal` | يرفع من | `WorkspaceCourse` | `N : 1` | إلزامية | Restrict عند حذف `WorkspaceCourse` |
| `UpdateProposal` | يؤلفه | `Account` | `N : 1` | إلزامية | Restrict عند حذف `Account` |
| `UpdateProposal` | يستهدف | `LibraryCourse` | `N : 1` | إلزامية | Restrict عند حذف `LibraryCourse` |
| `ProposalContentChange` | يتبع لمقترح | `UpdateProposal` | `N : 1` | إلزامية | Cascade عند حذف `UpdateProposal` |
| `ProposalContentChange` | يستهدف درسًا معتمدًا | `VersionLesson` | `N : 0..1` | اختيارية (فقط عند التعديل/الحذف أو الإضافة لدرس قائم) | Restrict عند حذف `VersionLesson` |
| `ProposalContentChange` | يستند إلى درس مساحة العمل | `WorkspaceLesson` | `N : 1` | إلزامية | Restrict عند حذف `WorkspaceLesson` |
| `ReviewInvitation` | تخص مقترح | `UpdateProposal` | `N : 1` | إلزامية | Cascade عند حذف `UpdateProposal` |
| `ReviewInvitation` | تدعو | `Account` | `N : 1` | إلزامية | Restrict عند حذف `Account` |
| `ReviewerAssignment` | تنشأ من | `ReviewInvitation` | `1 : 1` | إلزامية | Cascade عند حذف `ReviewInvitation` |
| `ReviewerAssignment` | تسند إلى | `Account` | `N : 1` | إلزامية | Restrict عند حذف `Account` |
| `CourseAttribution` | يوثق الفضل في | `CourseVersion` | `N : 1` | إلزامية | Restrict عند حذف `CourseVersion` |
| `CourseAttribution` | ينسب إلى | `Account` | `N : 1` | إلزامية | Restrict عند حذف `Account` |

---

## 5. القيود المنطقية للحفاظ على الثوابت (Logical Invariants & Rules)
### قواعد السلوك عند الحذف (Referential Actions)

- **Cascade:** يُستخدم فقط للعناصر التابعة التي لا تحمل معنى مستقلًا خارج الكيان الأب، مثل بنية ومحتوى مساحة العمل عند حذف `WorkspaceCourse`، وعناصر المقترح عند حذف `UpdateProposal`، ومهمة المراجعة عند حذف الدعوة.
- **Restrict:** يُستخدم للكيانات المرجعية أو التاريخية أو المرتبطة بحقوق/سجل دائم، مثل `Account`, `LibraryCourse`, `CourseVersion` و`SharedContentItem`؛ يمنع حذف الأصل ما دامت هناك سجلات تعتمد عليه.
- **LibraryCourse / CourseVersion:** لا يجوز حذف `LibraryCourse` إذا كان له `CourseVersion` أو `WorkspaceCourseReference` أو `UpdateProposal` أو تصنيف مرتبط. ولا يجوز حذف `CourseVersion` إذا كان مرتبطًا بـ`WorkspaceCourseReference` أو `LibraryCourse.current_version_id` أو بنية الإصدار أو إسناد مساهمين/مراجعين.
- **Account:** لا يُحذف `Account` إذا كان مستخدمًا كمالك أو مؤلف أو مدعو أو مراجع أو مساهم؛ يُستخدم `status = SUSPENDED` بدلًا من حذف الحساب المحتفظ بسجل تاريخي.
- **Workspace:** حذف `Workspace` يزيل كياناتها التابعة عبر سلسلة `Cascade`، لكن لا يمتد الحذف إلى `Account` أو `LibraryCourse` أو `CourseVersion`.
- **الهدف من هذه القواعد:** منع فقدان السجل التاريخي أو كسر المراجع، مع السماح بالتطهير الآمن للبيانات التابعة التي لا تستقل دلاليًا عن مالكها.

1. **ثبات الإصدارات المعتمدة:** عند نشر `CourseVersion` تصبح `CourseVersion` و`VersionUnit` و`VersionLesson` و`VersionManifestItem` للقراءة فقط.
2. **فصل تخزين المحتوى عن البيانات الوصفية:** لا تُخزن محتويات الملفات الفعلية داخل جداول الـmetadata؛ تستخدم الكيانات الملفية `storage_path` إلى `Object Storage` مع `checksum`.
3. **القراءة المرجعية والنسخ عند التعديل:** المقرر المرتبط بالمكتبة يُقرأ مرجعيًا عبر `WorkspaceCourseReference` و`CourseVersion`، ولا يُنشأ `WorkspaceContentItem` لهذه الملفات المشتركة. يُستخدم `WorkspaceContentItem` للمحتوى المحلي أو للنسخ الخاصة والتعديلات المحلية المستقلة، ويكون `storage_path` إلزاميًا.
4. **صحة ارتباط الإصدار المثبت وحالة المزامنة:** `pinned_version_id` يجب أن يتبع نفس `library_course_id`، و`sync_status` قيمة مشتقة آنيًا من مقارنة `pinned_version_id` مع `LibraryCourse.current_version_id` ولا تُخزن في `WorkspaceCourseReference`.
5. **سلامة موضع التغيير المقترح:** `source_workspace_lesson_id` إلزامي لكل `ProposalContentChange`. في `MODIFY` و`DELETE` يكون `target_version_lesson_id` مطلوبًا، وفي `ADD` يكون مطلوبًا عند إضافة ملف إلى درس قائم، ويكون `NULL` فقط عند إضافة ملف يتبع درسًا جديدًا.
6. **تجميد المحتوى المقترح أثناء المراجعة:** بعد `SUBMITTED` أو `UNDER_REVIEW` لا تُعدل سجلات `ProposalContentChange`.
7. **استقلال مقرر مساحة العمل عن مقرر المكتبة:** التعديل أو الحذف في `WorkspaceCourse` لا يغير `LibraryCourse` المقابل.
8. **الترقية والتطهير بعد اعتماد النشر (Promotion & Cleanup):** عند اعتماد نشر مقرر خاص متزامن سحابيًا أو اعتماد تحديثه وتحويله إلى `CourseVersion` في المكتبة العامة:
   1. تصبح الملفات المعتمدة أصولًا مشتركة مملوكة للمكتبة (`SharedContentItem`) وتُربط بالإصدار عبر `VersionManifestItem`.
   2. تُحذف من قاعدة البيانات السحابية سجلات المسودة الخاصة المتزامنة لذلك المقرر (`WorkspaceUnit`, `WorkspaceLesson`, `WorkspaceContentItem`) بعد نجاح إنشاء الإصدار والربط المرجعي، لمنع ازدواجية التخزين.
   3. يبقى `WorkspaceCourse` كسجل حاوية وارتباط لمساحة العمل، ويُنشأ أو يُفعّل `WorkspaceCourseReference` للإشارة إلى الإصدار المعتمد (`Pinned Version`) باعتباره مصدر الحقيقة (`Source of Truth`) للمحتوى المنشور.
   4. لا يعني هذا التطهير حذف البيانات المحلية على جهاز المستخدم؛ تبقى إدارة النسخة المحلية والمزامنة اللاحقة ضمن طبقة التطبيق Local-First.

---

## 6. مصفوفة التحقق من الاتساق (Traceability Matrix)

| المتطلب من المراحل السابقة | كيفية تحقيقه في النموذج المنطقي (Logical Data Model) |
|---|---|
| فصل الهوية عن الحسابات (Phase 2 & 3) | تمثيل منفصل لـ `Identity` و `Account`، مع السماح للهوية بامتلاك أكثر من حساب، وتخصص الحساب في `StudentAccount` أو `TeacherAccount`. |
| استقلال بيئة مساحة العمل وعدم الاستنساخ التلقائي (Phase 3 & 4) | يعمل النظام وفق Local-First؛ وتظهر كيانات `Workspace*` على السحابة كطبقة Cloud Sync اختيارية. يشير `WorkspaceCourseReference` إلى الإصدار المثبت (`Pinned Version`) دون نسخ الملفات المشتركة، بينما تُستخدم كيانات `WorkspaceUnit` و`WorkspaceLesson` و`WorkspaceContentItem` لتمثيل البيانات الخاصة المتزامنة سحابيًا. |
| فصل الهيكل عن المحتوى (Phase 2 & 5) | `WorkspaceLesson` تمثل الحاوية، و `WorkspaceContentItem` يمثل ملف المحتوى بعنوانه ونوعه ومساره المستقل. |
| إدارة الإصدارات على مستوى الملفات المتغيرة (Phase 2 & 6) | الفصل بين `SharedContentItem` وبنية الإصدار عبر `VersionUnit` و`VersionLesson` والربط عبر `VersionManifestItem` دون تكرار العناوين والترتيب. |
| حسم تصنيف وسياق النشر (Phase 6 Deferred Point) | نمذجة `ClassificationTaxonomy` كشجرة متعددة الأبعاد وربطها بـ `LibraryCourseClassification`. |
| استيعاب مسار المراجعة والاعتماد والمساهمين (Phase 4 & 5) | كيانات كاملة للمقترح (`UpdateProposal`)، الدعوات (`ReviewInvitation`)، المهام (`ReviewerAssignment`)، وحفظ الحقوق (`CourseAttribution`). |

---

## 7. الخلاصة والخطوة التالية

يغطي هذا النموذج المنطقي الكيانات والعلاقات والمحددات المفاهيمية المتفق عليها، ويفصل سياقات الحسابات، ويطبع بنية الإصدارات، ويفصل القراءة المرجعية للمحتوى المشترك عن المحتوى المحلي والنسخ الخاصة، ويفصل تخزين المحتوى الفعلي عن بياناته الوصفية.

بهذا تكتمل متطلبات **Phase 7 — Logical Data Model**، وتصبح هذه الوثيقة المرجع الأساسي المعتمد للانتقال إلى المرحلة التالية: **نموذج الوصول وأمن البيانات (Phase 8 — Data Access & Security Model)**.