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
  2. `VersionManifestItem`: يمثل كيان التقاطع المنطقي (المانيفست) الذي يربط بين إصدار محدد للمقرر (`CourseVersion`) وأصول المحتوى (`SharedContentItem`)، مع تحديد مسار الأصل داخل هيكل الإصدار (`unit_order`, `lesson_order`, `content_type`).
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
- **الغرض المفاهيمي:** الحساب الوظيفي للمستخدم داخل المنصة وسياق أدواره التعليمية.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `identity_id`: Identifier → يشير إلى `Identity.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `identity_id`: Identifier, إلزامية (FK).
  - `account_type`: Enum (`STUDENT`, `TEACHER`), إلزامية.
  - `display_name`: Text, إلزامية.
  - `status`: Enum (`ACTIVE`, `SUSPENDED`), إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(identity_id, account_type)`: يمنع تكرار نوع الحساب لنفس الهوية (لكل هوية حساب طالب واحد و/أو حساب معلم واحد كحد أقصى).

### النطاق 2: نطاق مساحات العمل (Workspace Domain)

#### 3. `Workspace`
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

#### 4. `WorkspaceCourse`
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

#### 5. `WorkspaceCourseReference`
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

#### 6. `WorkspaceUnit`
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

#### 7. `WorkspaceLesson`
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

#### 8. `WorkspaceContentItem`
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
  - `content_payload`: Document / Text, إلزامية في حال التعديل أو الإنشاء المحلي.
  - `checksum`: String, إلزامية.
  - `created_at`: Timestamp, إلزامية.
  - `updated_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(lesson_id, content_type)`: لا يمكن وجود أكثر من ملف واحد من نفس النوع داخل الدرس.
  - `CHECK (is_private_copy = FALSE IMPLIES shared_content_id IS NOT NULL)`: إذا لم يكن الملف نسخة خاصة، يجب أن يشير حتمًا إلى ملف مشترك معتمد.

### النطاق 5: نطاق المكتبة العامة وإدارة الإصدارات (Public Library & Versioning Domain)

#### 9. `LibraryCourse`
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

#### 10. `CourseVersion`
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

#### 11. `SharedContentItem`
- **الغرض المفاهيمي:** الأصول التعليمية المشتركة المستقرة المملوكة للمكتبة العامة (`Library Ownership`).
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
  - `file_format`: Enum (`MARKDOWN`, `JSON`), إلزامية.
  - `content_payload`: Document / Text, إلزامية (محتوى الأصل المعتمد).
  - `checksum`: String, إلزامية, فريدة منطقيًا لمحتوى النوع.
  - `created_at`: Timestamp, إلزامية.
- **القيود المنطقية (Constraints):**
  - **قيد عدم التعديل (Immutability Invariant):** أصل معتمد وثابت، لا يُعدل ولا يُحذف إذا كان مرتبطًا بإصدار معتمد.

#### 12. `VersionManifestItem`
- **الغرض المفاهيمي:** بيان ربط الإصدار بمكوناته من الملفات المشتركة، محددًا هيكل الإصدار ومساراته (File-level Versioning Manifest).
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `course_version_id`: Identifier → يشير إلى `CourseVersion.id`, إلزامية.
  - `shared_content_id`: Identifier → يشير إلى `SharedContentItem.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `course_version_id`: Identifier, إلزامية (FK).
  - `shared_content_id`: Identifier, إلزامية (FK).
  - `unit_title`: Text, إلزامية.
  - `unit_order`: Integer, إلزامية.
  - `lesson_title`: Text, إلزامية.
  - `lesson_order`: Integer, إلزامية.
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
- **القيود المنطقية (Constraints):**
  - `UNIQUE(course_version_id, unit_order, lesson_order, content_type)`: هيكل لا يقبل التكرار لنفس نوع الملف داخل الدرس في الإصدار الواحد.
  - **قيد عدم التعديل (Immutability Invariant):** بيان الإصدار ثابت بمجرد قفل الإصدار ونشره.

#### 13. `ClassificationTaxonomy`
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

#### 14. `LibraryCourseClassification`
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

#### 15. `UpdateProposal`
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

#### 16. `ProposalContentChange`
- **الغرض المفاهيمي:** تفصيل التغييرات المقترحة في المحتوى والمستخرجة من النسخ الخاصة في مساحة العمل.
- **المفتاح الأساسي (PK):** `id` (Identifier)
- **المفاتيح الأجنبية (FK):**
  - `proposal_id`: Identifier → يشير إلى `UpdateProposal.id`, إلزامية.
- **السمات (Attributes):**
  - `id`: Identifier, إلزامية (PK).
  - `proposal_id`: Identifier, إلزامية (FK).
  - `change_action`: Enum (`ADD`, `MODIFY`, `DELETE`), إلزامية.
  - `unit_title`: Text, إلزامية.
  - `lesson_title`: Text, إلزامية.
  - `content_type`: Enum (`SOURCE`, `OBJECTIVES`, `FLASHCARDS`, `UNDERSTANDING_CARDS`, `LESSON_QUIZ`), إلزامية.
  - `file_format`: Enum (`MARKDOWN`, `JSON`), إلزامية.
  - `proposed_payload`: Document / Text, اختيارية (إلزامية في حالات الإضافة والتعديل).
  - `proposed_checksum`: String, إلزامية.
  - `created_at`: Timestamp, إلزامية.

#### 17. `ReviewInvitation`
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

#### 18. `ReviewerAssignment`
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

#### 19. `CourseAttribution`
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

| الكيان المصدر (Source) | العلاقة | الكيان الهدف (Target) | التعددية (Cardinality) | الإلزامية (Optionality) | قاعدة التكامل المرجعي المنطقية (Referential Action) |
|---|---|---|---|---|---|
| `Account` | يتبع لـ | `Identity` | `N : 1` | إلزامية من جهة الحساب | يُحظر حذف الهوية إذا كان لها حسابات نشطة (Restrict). |
| `Workspace` | مملوكة لـ | `Account` | `N : 1` | إلزامية | حذف مساحة العمل محليًا يتبع سياسة الحساب (Cascade logical delete). |
| `WorkspaceCourse` | ينتمي لـ | `Workspace` | `N : 1` | إلزامية | يُحذف بحذف مساحة العمل (Cascade). |
| `WorkspaceCourseReference` | يحدد ارتباط | `WorkspaceCourse` | `1 : 1` | إلزامية للارتباط | علاقة تبعية تامة 1:1 للارتباط بالمكتبة. |
| `WorkspaceCourseReference` | يشير إلى | `LibraryCourse` | `N : 1` | إلزامية | مرجع محمي؛ لا يمكن حذف مقرر مكتبة عامة مستخدم. |
| `WorkspaceCourseReference` | يثبت إصدار | `CourseVersion` | `N : 1` | إلزامية | الإصدار المعتمد غير قابل للحذف (Restrict). |
| `WorkspaceUnit` | تتبع لـ | `WorkspaceCourse` | `N : 1` | إلزامية | تُحذف بحذف مقرر مساحة العمل (Cascade). |
| `WorkspaceLesson` | يتبع لـ | `WorkspaceUnit` | `N : 1` | إلزامية | يُحذف بحذف الوحدة (Cascade). |
| `WorkspaceContentItem` | يستضاف في | `WorkspaceLesson` | `N : 1` | إلزامية | يُحذف بحذف الدرس (Cascade). |
| `WorkspaceContentItem` | يشير كأصل إلى | `SharedContentItem` | `N : 0..1` | اختيارية (فقط إذا لم يكن محليًا خالصًا) | الأصل المشترك محمي من الحذف نهائيًا (Restrict). |
| `CourseVersion` | يوثق إصدارًا لـ | `LibraryCourse` | `N : 1` | إلزامية | يُحظر حذف مقرر المكتبة طالما صدرت له إصدارات (Restrict). |
| `VersionManifestItem` | ينتمي لبيان | `CourseVersion` | `N : 1` | إلزامية | سجلات المانيفست تتبع الإصدار وتصبح ثابتة معه. |
| `VersionManifestItem` | يربط الأصل | `SharedContentItem` | `N : 1` | إلزامية | لا يمكن حذف أي أصل تعليمي مدرج في مانيفست معتمد. |
| `LibraryCourseClassification` | يربط تصنيف | `LibraryCourse` & `ClassificationTaxonomy` | `M : N` | إلزامية للطرفين | جدول وسيط للربط متعدد الأطراف. |
| `UpdateProposal` | يرفع من | `WorkspaceCourse` | `N : 1` | إلزامية | المقترح يوثق مصدر التحديث. |
| `ProposalContentChange` | يتبع لمقترح | `UpdateProposal` | `N : 1` | إلزامية | تفاصيل التغييرات تحذف بإلغاء أو حذف المقترح مسودة. |
| `ReviewInvitation` | تخص مقترح | `UpdateProposal` | `N : 1` | إلزامية | ترتبط بدورة حياة المقترح. |
| `ReviewerAssignment` | تنشأ من | `ReviewInvitation` | `1 : 1` | إلزامية | علاقة حصرية 1:1 لكل دعوة مقبولة. |
| `CourseAttribution` | يوثق الفضل في | `CourseVersion` | `N : 1` | إلزامية | سجل غير قابل للتعديل يوثق أصحاب الإنجاز. |

---

## 5. القيود المنطقية للحفاظ على الثوابت (Logical Invariants & Rules)

1. **ثبات الإصدارات المعتمدة (Immutability of Released Versions):**
   - بمجرد إنشاء سجل `CourseVersion` ونشره، ومعه سجلات `VersionManifestItem` التابعة له، تصبح جميع هذه السجلات للقراءة فقط (`Read-Only`). لا يُسمح بتعديل أي سمة أو مسار أو مرجع داخلها.
2. **انفصال النسخة الخاصة (Private Copy Decoupling Invariant):**
   - عندما يكون `is_private_copy = TRUE` في `WorkspaceContentItem`، يمتلك السجل محتواه الخاص في `content_payload`، ولا يتأثر بأي تعديلات تطرأ على المكتبة العامة، ولا تتغير نسخته إلا بإجراء ترقية واعية وصريحة (`Explicit Upgrade`).
3. **صحة ارتباط الإصدار المثبت (Pinned Version Consistency):**
   - في `WorkspaceCourseReference`، يجب أن يتحقق منطقيًا أن الإصدار المربوط `pinned_version_id` يتبع حصريًا لنفس المقرر المربوط `library_course_id`.
4. **تجميد المحتوى المقترح أثناء المراجعة (Proposal Content Freezing):**
   - بمجرد انتقال `UpdateProposal` إلى حالة `SUBMITTED` أو `UNDER_REVIEW`، يُمنع تعديل أو إضافة أي سجلات تابعة في `ProposalContentChange`.
5. **استقلال مقرر مساحة العمل عن مقرر المكتبة:**
   - الحذف أو التعديل في `WorkspaceCourse` لا يملك أي تأثير على `LibraryCourse` المقابل.

---

## 6. مصفوفة التحقق من الاتساق (Traceability Matrix)

| المتطلب من المراحل السابقة | كيفية تحقيقه في النموذج المنطقي (Logical Data Model) |
|---|---|
| فصل الهوية عن الحسابات (Phase 2 & 3) | تمثيل منفصل لـ `Identity` و `Account` مع قيد فريد يمنع تكرار الدور للهوية. |
| استقلال بيئة مساحة العمل وعدم الاستنساخ التلقائي (Phase 3 & 4) | كيان `WorkspaceCourseReference` يشير فقط للإصدار المثبت (`Pinned Version`) دون نسخ البيانات. |
| فصل الهيكل عن المحتوى (Phase 2 & 5) | `WorkspaceLesson` تمثل الحاوية، و `WorkspaceContentItem` يمثل ملف المحتوى بنوعه المخصص. |
| إدارة الإصدارات على مستوى الملفات المتغيرة (Phase 2 & 6) | الفصل المنطقي بين `SharedContentItem` (الأصل الثابت) و `VersionManifestItem` (بيان التجميع لكل إصدار). |
| حسم تصنيف وسياق النشر (Phase 6 Deferred Point) | نمذجة `ClassificationTaxonomy` كشجرة متعددة الأبعاد وربطها بـ `LibraryCourseClassification`. |
| استيعاب مسار المراجعة والاعتماد والمساهمين (Phase 4 & 5) | كيانات كاملة للمقترح (`UpdateProposal`)، الدعوات (`ReviewInvitation`)، المهام (`ReviewerAssignment`)، وحفظ الحقوق (`CourseAttribution`). |

---

## 7. الخلاصة والخطوة التالية

يغطي هذا النموذج المنطقي كافة الكيانات، والعلاقات، والمحددات المفاهيمية المتفق عليها في منظومة **عصبون**، ويحسم بشكل قاطع النقاط الثلاث المؤجلة دون الدخول في تفاصيل قواعد البيانات الفيزيائية.

بهذا تكتمل متطلبات **Phase 7 — Logical Data Model**، وتصبح هذه الوثيقة المرجع الأساسي المعتمد للانتقال إلى المرحلة التالية: **نموذج الوصول وأمن البيانات (Phase 8 — Data Access & Security Model)**.