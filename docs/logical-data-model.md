# وثيقة نموذج البيانات المنطقي | Logical Data Model — مشروع عصبون (osboon)

## 1. الهدف ونطاق النموذج (Objective & Scope)

تهدف هذه الوثيقة إلى تحويل الأساس المفاهيمي المعتمد في المراحل السابقة (المصطلحات، حدود النطاقات، الحالات ومسارات العمل، تصنيف الكيانات والعلاقات، والمحاذاة الشاملة) إلى **نموذج بيانات منطقي (Logical Data Model)** دقيق ومتكامل، يحدد:
- الكيانات المنطقية (Logical Entities) وحقولها (Attributes).
- المفاتيح الأساسية (Primary Keys) والقيود الفريدة (Unique Constraints).
- المفاتيح الأجنبية (Foreign Keys) والعلاقات المنطقية والتعددية (Cardinality & Optionality).
- القواعد والقيود المنطقية الحاكمة لتكامل البيانات (Integrity Rules & Invariants).
- حسم النقاط الثلاث المؤجلة في مرحلة المحاذاة الشاملة.

> **حدود النطاق:** يقتصر هذا النموذج على المستوى المنطقي التحليلي فقط؛ ولا يتضمن أي تفاصيل تنفيذية تقنية مثل أوامر SQL أو سكريبتات الترحيل (Migrations)، أو أنواع بيانات PostgreSQL الخاصة، أو سياسات RLS، أو كود الـ Backend.

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
  2. `VersionUnit`, `VersionLesson`, `VersionManifestItem`: تمثل البنية الهيكلية للإصدار وبيان محتواه، بحيث يشير كل `VersionManifestItem` إلى `VersionLesson` وإلى `SharedContentItem` بحسب `content_type`، دون تكرار عنوان الوحدة أو الدرس أو ترتيب كل ملف داخل المانيفست.
- **التبرير:** يضمن عدم استنساخ الملفات غير المتغيرة عبر الإصدارات؛ فالإصدار `v1.1` يشير إلى نفس سجلات `SharedContentItem` الخاصة بـ `v1.0` للملفات الثابتة، بينما يسجل فقط إشارات جديدة للملفات المتغيرة (`Changed Content`).

### النقطة الثالثة: نمذجة الأنواع الخمسة للمحتوى التعليمي (Educational Content Typing)
- **القرار المنطقي:** اعتماد نمط **الكيان الموحد متعدد الأنواع (Single Logical Content Entity with Discriminator)**.
- **التبرير:** تشترك ملفات المحتوى الخمسة (`Source`, `Objectives`, `Flashcards`, `Understanding Cards`, `Lesson Quiz/Test`) في نفس دورة الحياة، وطرق التعديل، والمزامنة، والاستضافة داخل الدرس، والاشتقاق للنسخ الخاصة (`Private Copy`). يتم التمييز بينها منطقيًا عبر سمة تمييزية (`content_type`) مع إلزام الدرس باستضافة ملف واحد كحد أقصى من كل نوع عبر قيد فريد مركب:
  `UNIQUE(lesson_id, content_type)`.

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
- **الغرض المفاهيمي:** الحساب الوظيفي المملوك لـ `Identity` ويمثل سياقًا تعليميًا محددًا.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `identity_id`: Identifier → يشير إلى `Identity.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `identity_id`: Identifier, إلزامية (FK).
  - `display_name`: Text, إلزامية.
  - `status`: String, إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - يمكن لـ `Identity` واحدة امتلاك أكثر من `Account`.
  - كل `Account` يتبع `Identity` واحدة فقط.

#### 3. `StudentAccount`
- **الغرض المفاهيمي:** التخصص الوظيفي للحساب في سياق الطالب.
- **المفتاح الأساسي (PK):** `account_id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `account_id`: Identifier → يشير إلى `Account.id`, إلزامية, فريدة (UK).
- **السمات (Attributes):**
  - `account_id`: Identifier, إلزامية (PK, FK).
- **القيود المنطقية (Constraints):**
  - يمثل `StudentAccount` تخصصًا لـ `Account` في سياق الطالب.
  - لا يجمع `Account` نفسه بين `StudentAccount` و`TeacherAccount`؛ يكون التخصص في أحدهما فقط.

#### 4. `TeacherAccount`
- **الغرض المفاهيمي:** التخصص الوظيفي للحساب في سياق المعلم.
- **المفتاح الأساسي (PK):** `account_id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `account_id`: Identifier → يشير إلى `Account.id`, إلزامية, فريدة (UK).
- **السمات (Attributes):**
  - `account_id`: Identifier, إلزامية (PK, FK).
- **القيود المنطقية (Constraints):**
  - يمثل `TeacherAccount` تخصصًا لـ `Account` في سياق المعلم.
  - لا يجمع `Account` نفسه بين `StudentAccount` و`TeacherAccount`؛ يكون التخصص في أحدهما فقط.

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
- **الغرض المفاهيمي:** التمثيل المستقل للمقرر داخل مساحة العمل (سواء كان محليًا أو مستندًا للمكتبة).
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
  - `sync_status`: Enum (`UP_TO_DATE`, `UPDATE_AVAILABLE`), إلزامية.
  - `last_synced_at`: Timestamp, إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - يجب أن يتبع `pinned_version_id` لنفس `library_course_id`.

### النطاق 3: نطاق البنية التعليمية (Learning Structure Domain)

#### 8. `WorkspaceUnit`
- **الغرض المفاهيمي:** الحاوية التنظيمية للوحدة التعليمية داخل مقرر مساحة العمل.
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
- **الغرض المفاهيمي:** أصغر حاوية هيكلية في المقرر؛ تستضيف عناصر المحتوى التعليمي.
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
- **الغرض المفاهيمي:** ملف المحتوى التعليمي المستضاف داخل الدرس؛ إما مرجع لمحتوى مشترك أو نسخة خاصة معدلة (`Private Copy`).
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `lesson_id`: Identifier → يشير إلى `WorkspaceLesson.id`, إلزامية.
  - `shared_content_id`: Identifier → يشير إلى `SharedContentItem.id`, اختيارية (NULL للملفات المحلية الخالصة).
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `lesson_id`: Identifier, إلزامية (FK).
  - `shared_content_id`: Identifier, اختيارية (FK).
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
  - `file_format`: Enum (`MARKDOWN`, `JSON`), إلزامية.
  - `is_private_copy`: Boolean, إلزامية (افتراضيًا `FALSE` للمراجع المشتركة).
  - `storage_path`: String, اختيارية؛ تكون فارغة عند الاعتماد على `SharedContentItem.storage_path`، وتصبح إلزامية عند وجود نسخة خاصة مستقلة.
  - `checksum`: String, إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(lesson_id, content_type)`: لا يمكن وجود أكثر من ملف واحد من نفس النوع داخل الدرس.
  - `CHECK (is_private_copy = FALSE IMPLIES (shared_content_id IS NOT NULL AND storage_path IS NULL))`: إذا لم يكن الملف نسخة خاصة، يعتمد على `SharedContentItem` ولا يخزن مسارًا محليًا.
  - `CHECK (is_private_copy = TRUE IMPLIES storage_path IS NOT NULL)`: إذا كان نسخة خاصة، يجب أن يملك مسار تخزين مستقلًا.

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
- **الغرض المفاهيمي:** كيان التقاطع الذي يربط درسًا داخل إصدار محدد بأصل محتوى مشترك، دون تكرار عنوان الوحدة أو الدرس أو ترتيبهما.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `version_lesson_id`: Identifier → يشير إلى `VersionLesson.id`, إلزامية.
  - `shared_content_id`: Identifier → يشير إلى `SharedContentItem.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `version_lesson_id`: Identifier, إلزامية (FK).
  - `shared_content_id`: Identifier, إلزامية (FK).
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(version_lesson_id, content_type)`: لا يمكن وجود أكثر من ملف واحد من نفس النوع داخل الدرس في الإصدار.
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
  - `change_action`: Enum (`ADD`, `MODIFY`, `DELETE`), إلزامية.
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
  - `file_format`: Enum (`MARKDOWN`, `JSON`), إلزامية.
  - `storage_path`: String, اختيارية (إلزامية في حالتي `ADD` و`MODIFY`)، وتشير إلى المحتوى المقترح في `Object Storage`.
  - `proposed_checksum`: String, إلزامية.
  - `created_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `ADD`: يجب أن يكون `target_version_lesson_id = NULL`، ويستمد النظام عنوان الدرس وترتيبه من `source_workspace_lesson_id`.
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

| الكيان المصدر | العلاقة | الكيان الهدف | Cardinality | Optionality |
|---|---|---|---|---|
| `Account` | يتبع لـ | `Identity` | `N : 1` | إلزامية |
| `StudentAccount` | يخص | `Account` | `1 : 1` | إلزامية من جهة الملف |
| `TeacherAccount` | يخص | `Account` | `1 : 1` | إلزامية من جهة الملف |
| `Workspace` | مملوكة لـ | `Account` | `N : 1` | إلزامية |
| `WorkspaceCourse` | ينتمي لـ | `Workspace` | `N : 1` | إلزامية |
| `WorkspaceCourseReference` | يحدد ارتباط | `WorkspaceCourse` | `1 : 1` | إلزامية |
| `WorkspaceCourseReference` | يشير إلى | `LibraryCourse` | `N : 1` | إلزامية |
| `WorkspaceCourseReference` | يثبت إصدار | `CourseVersion` | `N : 1` | إلزامية |
| `WorkspaceUnit` | تتبع لـ | `WorkspaceCourse` | `N : 1` | إلزامية |
| `WorkspaceLesson` | يتبع لـ | `WorkspaceUnit` | `N : 1` | إلزامية |
| `WorkspaceContentItem` | يستضاف في | `WorkspaceLesson` | `N : 1` | إلزامية |
| `WorkspaceContentItem` | يشير إلى | `SharedContentItem` | `N : 0..1` | اختيارية |
| `CourseVersion` | يوثق إصدارًا لـ | `LibraryCourse` | `N : 1` | إلزامية |
| `VersionUnit` | تتبع لـ | `CourseVersion` | `N : 1` | إلزامية |
| `VersionLesson` | تتبع لـ | `VersionUnit` | `N : 1` | إلزامية |
| `VersionManifestItem` | ينتمي لدرس | `VersionLesson` | `N : 1` | إلزامية |
| `VersionManifestItem` | يربط الأصل | `SharedContentItem` | `N : 1` | إلزامية |
| `LibraryCourseClassification` | يربط تصنيف | `LibraryCourse` & `ClassificationTaxonomy` | `M : N` | إلزامية |
| `UpdateProposal` | يرفع من | `WorkspaceCourse` | `N : 1` | إلزامية |
| `ProposalContentChange` | يتبع لمقترح | `UpdateProposal` | `N : 1` | إلزامية |
| `ReviewInvitation` | تخص مقترح | `UpdateProposal` | `N : 1` | إلزامية |
| `ReviewerAssignment` | تنشأ من | `ReviewInvitation` | `1 : 1` | إلزامية |
| `CourseAttribution` | يوثق الفضل في | `CourseVersion` | `N : 1` | إلزامية |

---

## 5. القيود المنطقية للحفاظ على الثوابت (Logical Invariants & Rules)

1. **ثبات الإصدارات المعتمدة:** عند نشر `CourseVersion` تصبح `CourseVersion` و`VersionUnit` و`VersionLesson` و`VersionManifestItem` للقراءة فقط.
2. **فصل تخزين المحتوى عن البيانات الوصفية:** لا تُخزن محتويات الملفات الفعلية داخل جداول الـmetadata؛ تستخدم الكيانات الملفية `storage_path` إلى `Object Storage` مع `checksum`.
3. **انفصال النسخة الخاصة:** `WorkspaceContentItem` الخاص يستخدم `storage_path` لمحتواه المستقل، بينما المرجع المشترك يعتمد على `SharedContentItem.storage_path` ولا يستنسخ المسار أو الملف محليًا.
4. **صحة ارتباط الإصدار المثبت:** `pinned_version_id` يجب أن يتبع نفس `library_course_id`.
5. **سلامة موضع التغيير المقترح:** `source_workspace_lesson_id` إلزامي لكل `ProposalContentChange`، بينما `target_version_lesson_id` يكون مطلوبًا في `MODIFY` و`DELETE`، ويكون `NULL` في `ADD` لدرس جديد.
6. **تجميد المحتوى المقترح أثناء المراجعة:** بعد `SUBMITTED` أو `UNDER_REVIEW` لا تُعدل سجلات `ProposalContentChange`.
7. **استقلال مقرر مساحة العمل عن مقرر المكتبة:** التعديل أو الحذف في `WorkspaceCourse` لا يغير `LibraryCourse` المقابل.

---

## 6. مصفوفة التحقق من الاتساق (Traceability Matrix)

| المتطلب من المراحل السابقة | كيفية تحقيقه في النموذج المنطقي (Logical Data Model) |
|---|---|
| فصل الهوية عن الحسابات (Phase 2 & 3) | تمثيل منفصل لـ `Identity` و `Account`، مع السماح للهوية بامتلاك أكثر من حساب، وتخصص الحساب في `StudentAccount` أو `TeacherAccount`. |
| استقلال بيئة مساحة العمل وعدم الاستنساخ التلقائي (Phase 3 & 4) | كيان `WorkspaceCourseReference` يشير فقط للإصدار المثبت (`Pinned Version`) دون نسخ البيانات. |
| فصل الهيكل عن المحتوى (Phase 2 & 5) | `WorkspaceLesson` تمثل الحاوية، و `WorkspaceContentItem` يمثل ملف المحتوى بنوعه المخصص. |
| إدارة الإصدارات على مستوى الملفات المتغيرة (Phase 2 & 6) | الفصل بين `SharedContentItem` وبنية الإصدار عبر `VersionUnit` و`VersionLesson` والربط عبر `VersionManifestItem` دون تكرار العناوين والترتيب. |
| حسم تصنيف وسياق النشر (Phase 6 Deferred Point) | نمذجة `ClassificationTaxonomy` كشجرة متعددة الأبعاد وربطها بـ `LibraryCourseClassification`. |
| استيعاب مسار المراجعة والاعتماد والمساهمين (Phase 4 & 5) | كيانات كاملة للمقترح (`UpdateProposal`)، الدعوات (`ReviewInvitation`)، المهام (`ReviewerAssignment`)، وحفظ الحقوق (`CourseAttribution`). |

---

## 7. الخلاصة والخطوة التالية

يغطي هذا النموذج المنطقي الكيانات والعلاقات والمحددات المفاهيمية المتفق عليها، ويفصل أدوار الحسابات، ويطبع بنية الإصدارات، ويفصل تخزين المحتوى الفعلي عن بياناته الوصفية.

بهذا تكتمل متطلبات **Phase 7 — Logical Data Model**، وتصبح هذه الوثيقة المرجع الأساسي المعتمد للانتقال إلى المرحلة التالية: **نموذج الوصول وأمن البيانات (Phase 8 — Data Access & Security Model)**.