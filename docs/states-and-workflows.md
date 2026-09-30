# وثيقة الحالات ومسارات العمل | States & Workflows — مشروع عصبون (osboon)

## 1. الهدف والمبادئ العامة

تهدف هذه الوثيقة إلى تحديد وتوثيق **الحالات (States)** و**مسارات العمل (Workflows)** المعتمدة في النظام استنادًا إلى وثيقتي *Terminology* و *Domain Boundaries*.

### المبادئ التوجيهية الحاكمة:
1. **تجنب التضخيم (No State Bloating):** الكيانات الثابتة أو التي لا تمر بمراحل اتخاذ قرار لا تُمنح آلات حالات (State Machines) غير ضرورية.
2. **الحتمية والوضوح (Determinism):** لكل انتقال (Transition) حالة بداية، حدث/محفز مسبب (Trigger)، جهة مسؤولة (Responsible Domain)، وحالة وصول محددة.
3. **الفصل بين المحلي والعام:** التعديلات داخل مساحات العمل خاصة تمامًا ولا تؤثر على دورات الحياة العامة إلا بطلب صريح من المستخدم.
4. **ثبات الإصدارات المعتمدة (Immutability of Approved Versions):** الإصدار المعتمد (`Course Version`) لا يدخل في دورات تعديل؛ التعديل ينشئ مقترحًا جديدًا دائمًا.

---

## 2. دورة النشر الأولي للمقرر (Course Initial Publishing Workflow)

نقل مقرر تم إنشاؤه داخل `Workspace` ليصبح متاحًا لأول مرة داخل `Public Library`.

```text
[ Draft in Workspace ]
         │
         ▼ (Submit for Publishing)
   [ Submitted ]
         │
         ▼ (Collaborative Review Passed)
   [ Approved ] ────► [ Published (v1.0) in Public Library ]
         │
         ▼ (Review Rejected)
   [ Rejected ]
```

### الحالات:
- **Draft (مسودة):** المقرر يعيش داخل الـ Workspace كإنشاء محلي خالص، غير معروف للمكتبة العامة.
- **Submitted (مرفوع للنشر):** قدم المستخدم طلب نشر للمقرر في المكتبة؛ يتجمد تعديل الهيكل المشترك أثناء التقييم.
- **Under Review (قيد المراجعة):** دخل المقرر مرحلة المراجعة والتحقق الجماعي.
- **Approved (معتمد للنشر):** اجتاز متطلبات الفحص ويُجهز لإنشاء أول إصدار (`v1.0`).
- **Published (منشور):** تم إنشاء أول إصدار رسمي، ونُقلت ملكية الملفات المشتركة إلى `Public Library` (`Library Ownership`).
- **Rejected (مرفوض):** لم يستوفِ المقرر معايير النشر في المكتبة العامة، وتعود إمكانية تعديله للمستخدم داخل الـ Workspace مع توضيح أسباب الرفض.

---

## 3. دورة حياة مقترح التحديث (Update Proposal Lifecycle)

هذا المسار مخصص لتعديل أو تطوير مقرر منشور مسبقًا في `Public Library`.

```text
[ Draft Proposal ]
       │
       ▼ (Submit Proposal)
  [ Submitted ]
       │
       ├──► [ Cancelled ] (By Author)
       ▼ (Assign Reviewers)
 [ Under Review ]
       │
       ├──► [ Rejected ]
       ▼ (Consensus / Approval)
  [ Approved Update ] ──► (Generates new Course Version in Public Library)
```

### الحالات:

| الحالة | المعنى المرجعي | الشروط والمحددات |
|---|---|---|
| **Draft Proposal** | مقترح التحديث قيد التجهيز داخل مساحة العمل الخاصة بالمستخدم. | لا يراه أحد سوى صاحب المقترح. |
| **Submitted** | تم إرسال المقترح إلى المنصة، وهو بانتظار تشكيل لجنة المراجعة. | لا يمكن للمستخدم تعديل المحتوى بعد رفعه إلا بسحبه. |
| **Under Review** | المقترح وُزع على المراجعين المختارين والعملية جارية. | يخضع لفترة زمنية محددة. |
| **Approved Update** | نال المقترح الأصوات والتقييمات الكافية للاعتماد. | حالة نهائية ضمن نطاق المراجعة، ينتقل بعدها مباشرة إلى نطاق المكتبة لإنشاء إصدار جديد. |
| **Rejected** | تم رفض المقترح من قبل المراجعين. | تُغلق المراجعة، وتعود الملاحظات للمستخدم. |
| **Cancelled** | سحب المستخدم المقترح قبل صدور القرار النهائي. | يُلغى المسار ولا يؤثر على المقرر الأصلي. |

### جدول انتقالات مقترح التحديث:

| الحالة السابقة | الحدث / الفعل (Trigger) | الجهة المسببة (Actor / Domain) | الحالة اللاحقة |
|---|---|---|---|
| Draft Proposal | رفع المقترح (Submit) | صاحب المقترح (Workspace Domain) | Submitted |
| Submitted | بدء تشكيل المراجعة وقبول المراجعين | منصة المراجعة (Review Domain) | Under Review |
| Submitted / Under Review | سحب المقترح (Cancel) | صاحب المقترح (Workspace Domain) | Cancelled |
| Under Review | اكتمال المراجعة بالقبول | نظام المراجعة (Collaborative Review) | Approved Update |
| Under Review | اكتمال المراجعة بالرفض | نظام المراجعة (Collaborative Review) | Rejected |

---

## 4. دورة حياة دعوة ومهمة المراجعة التعاونية (Review & Reviewer Lifecycle)

تدار هذه الدورة بالكامل داخل **Collaborative Review Domain** لضمان الحيادية والجودة.

```text
               [ Review Invitation Sent ]
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
        [ Accepted ]  [ Declined ]  [ Expired ]
             │
      (Becomes Reviewer)
             │
             ▼
     [ Review Active ]
             │
      ┌──────┴──────┐
      ▼             ▼
[ Completed ]  [ Timed Out ]
```

### 1. حالات دعوة المراجعة (Review Invitation States):
- **Pending (معلقة):** تم إرسال الدعوة آليًا إلى المستخدم المؤهل (سواء كان طالبًا أو معلمًا) وبانتظار قراره.
- **Accepted (مقبولة):** قبل المستخدم الدعوة، وأصبح رسميًا `Reviewer` لهذا المقترح.
- **Declined (معتذر عنها):** رفض المستخدم الدعوة؛ يقوم النظام بترشيح مراجع بديل آليًا.
- **Expired (منتهية الصلاحية):** انقضت المهلة المحددة للرد على الدعوة دون قبول أو رفض؛ تلغى الدعوة ويُرشح بديل.

### 2. حالات مهمة المراجع (Reviewer Task States):
- **Active (جارية):** المراجع يمتلك صلاحية الاطلاع على التغييرات (`Changed Content`) وتقديم التقييم والملاحظات.
- **Completed (مكتملة):** قدم المراجع صوته النهائي وملاحظاته.
- **Timed Out (متعثرة زمنياً):** لم يقدم المراجع تقييمه ضمن المهلة المحددة للمراجعة؛ يتم استبعاده واحتساب النصاب بناءً على القواعد المعتمدة أو تعيين بديل.

---

## 5. دورة اعتماد التحديث وإنشاء الإصدار الجديد (Version Generation Workflow)

هذا المسار يربط بين **Collaborative Review Domain** و **Public Library Domain**.

```text
[ Approved Update ]
         │
         ▼ (Library Release Trigger)
[ File-level Differencing & Assembly ]
         │
         ▼
[ Course Version Published (Immutable) ]
         │
         ▼
[ Current Version Pointer Updated ]
```

### الخطوات المنطقية:
1. **الاستلام:** يستلم `Public Library Domain` إشعارًا بوجود `Approved Update`.
2. **المعالجة (File-level Versioning):**
   - حفظ الملفات الجديدة أو المعدلة (`Changed Content`).
   - الإبقاء على مراجع الملفات غير المتغيرة كما هي مستندة إلى أصولها المشتركة (`Shared Content`).
3. **إصدار النسخة (Course Version):**
   - إعطاء رقم إصدار جديد تتابعي (مثلًا: `v1.1` أو `v2.0`).
   - تثبيت تاريخ الإصدار وهوية المساهمين (`Contributors`) والمراجعين (`Reviewers`).
4. **تحديث المؤشر العام:**
   - يصبح الإصدار الجديد هو الـ `Current Version` للمقرر في المكتبة العامة.
   - الإصدارات السابقة تنتقل إلى الأرشيف التاريخي (`Version History`) دون حذفها أو تعديلها.

---

## 6. مسار ارتباط المقرر وترقيته في مساحة العمل (Adoption & Upgrade Workflow)

يصف حالة العلاقة بين المقرر في `Workspace` والإصدار المعتمد في `Public Library`.

```text
[ Library Course Selected ]
            │
            ▼ (Adopt Version)
   [ Pinned to Version X ] ── (Up to Date)
            │
            │ (New Course Version Released in Library)
            ▼
   [ Update Available Notice ]
            │
            ▼ (User Explicit Action: Upgrade)
   [ Pinned to Version X+1 ]
```

### حالات ارتباط المقرر في مساحة العمل (Workspace Course Reference States):
- **Up to Date (محدث):** المقرر في مساحة العمل مرتبط بـ `Pinned Version` يطابق أحدث إصدار معتمد (`Current Version`) في المكتبة العامة.
- **Update Available (يتوفر تحديث):** صدر إصدار معتمد أحدث في المكتبة العامة، لكن مساحة عمل المستخدم **تظل مستقرة** على الإصدار الذي اختاره (`Pinned Version`) دون ترقية إجبارية.

### أحداث الترقية والربط:
- **Version Adoption (التبني):** إضافة المقرر من المكتبة العامة إلى مساحة العمل لأول مرة، واختيار إصدار محدد (`Pinned Version`).
- **Version Upgrade (الترقية الصريحة):** انتقال واعٍ ومتعمد من المستخدم بطلب ترقية المقرر الداخلي إلى الإصدار الأحدث.

---

## 7. دورة التعديلات الخاصة والنسخ المستقلة (Private Modification & Private Copy Lifecycle)

تصف سلوك الملفات التعليمية داخل مساحة العمل عند تعديل مقرر مستورد من المكتبة العامة.

```text
[ Shared File Reference (Read-only from Library) ]
                     │
                     ▼ (User edits file locally)
       [ Private Copy Created (Workspace-scoped) ]
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
(Revert Changes)        (Submit as Update Proposal)
         │                       │
[ Restored to Shared ]   [ Linked to Proposal ]
```

### حالات الملف التعليمي داخل الدرس (Lesson):
- **Shared Reference (مرجع مشترك):** الملف يطابق تمامًا النسخة المعتمدة في المكتبة العامة ضمن الـ `Pinned Version`، ولا يحتل مساحة تعديل خاصة.
- **Private Copy (نسخة خاصة):** بمجرد قيام المستخدم بأول تعديل على الملف، ينفصل الملف محليًا ويتحول إلى نسخة خاصة معزولة داخل نطاق مساحة العمل (`Private Modification`).

### مسارات إنهاء النسخة الخاصة:
1. **Revert (التراجع):** يقرر المستخدم إلغاء تعديلاته الخاصة، فيتم حذف الـ `Private Copy` ويعود الدرس للإشارة إلى النسخة المشتركة للمكتبة (`Shared Content`).
2. **Propose (اقتراح للنشر):** يضم المستخدم هذه النسخة إلى `Update Proposal` للمراجعة العامة.

---

## 8. دورات حياة الكيانات الداعمة (Minimal Supporting Lifecycles)

تطبيقًا لمبدأ عدم تضخيم الحالات، تُعرف الحالات الأساسية لهذه الكيانات دون تعقيد:

### 1. مساحة العمل (Workspace Lifecycle):
- **Active (نشطة):** المساحة متاحة للمستخدم للقراءة والتعديل والدراسة.
- **Archived (مؤرشفة):** مساحة عمل للقراءة فقط، مجمدة بقرار المستخدم لتقليل التشتت.

### 2. الحساب (Account Lifecycle):
- **Active (نشط):** الحساب مفعل ومتاح للاستخدام.
- **Suspended (معطل):** الحساب موقوف إداريًا لمخالفة معايير المنصة أو بطلب أمني.

*(ملاحظة: الـ Identity تُدار بالكامل عبر Supabase Auth خارج آلة حالات النظام الخاصة).*

---

## 9. مصفوفة الانتقالات المحظورة والاستثناءات (Forbidden Transitions & Edge Cases)

| الانتقال المحظور / الحالة الحرجة | سبب الحظر / القرار المعتمد |
|---|---|
| **الترقية التلقائية للمقرر في Workspace** | **محظور قطعيًا:** لا يتم ترقية أي مقرر في مساحة العمل إلى إصدار أحدث آليًا، لمنع تخريب تجربة الطالب أو المعلم أثناء الفصل الدراسي. |
| **تعديل Course Version معتمد** | **محظور قطعيًا:** الإصدارات المعتمدة غير قابلة للتغيير (`Immutable`). أي تغيير يجب أن يسلك مسار `Update Proposal`. |
| **تعديل Update Proposal أثناء مرحلة Under Review** | **محظور:** تجميد المحتوى ضروري لضمان أن المراجعين يفحصون نسخة ثابتة لا تتغير أثناء تقييمهم. إذا أراد التعديل، عليه سحب المقترح (`Cancelled`) ورفعه مجددًا. |
| **تعارض النسخ الخاصة عند الترقية (Upgrade with Private Copies)** | **استثناء يدار بوعي:** إذا اختار المستخدم ترقية مقرره إلى إصدار جديد، وكانت لديه ملفات بصيغة `Private Copy`، يُخير النظام المستخدم: إما الإبقاء على نسخته الخاصة وتجاهل تحديث هذا الملف المحدد، أو استبدالها بنسخة الإصدار الجديد للمكتبة. |
| **انسحاب جميع المراجعين من المقترح** | **إجراء استثنائي:** إذا اعتذر أو تأخر المراجعون حتى كسر النصاب، يعيد نظام المراجعة فتح الدعوات تلقائيًا لمرشحين جدد وتمديد المهلة الزمنية، دون إلغاء المقترح. |

---

## 10. ملخص ربط المسارات بالنطاقات (Workflow Responsibility Map)

| المسار (Workflow) | النطاق المسؤول الرئيسي (Primary Domain) | النطاقات المشتركة / الداعمة |
|---|---|---|
| **نشر المقرر لأول مرة** | Workspace Domain | Collaborative Review, Public Library |
| **دورة حياة مقترح التحديث** | Collaborative Review Domain | Workspace Domain |
| **إدارة الدعوات وتحكيم المراجعة** | Collaborative Review Domain | Identity & Account (اختيار وتكليف المراجعين) |
| **توليد إصدار جديد (Versioning)** | Public Library & Versioning | Collaborative Review (مصدر الاعتماد) |
| **ترقية وتبني الإصدارات** | Workspace Domain | Public Library (مصدر الإصدارات) |
| **إدارة النسخ الخاصة (Private Copy)** | Educational Content Domain | Workspace Domain |

---

## 11. الخلاصة

تحدد هذه الوثيقة بوضوح لا لبس فيه كافة الانتقالات المسموحة وغير المسموحة والمحفزات المسؤولة عنها عبر النطاقات الستة للنظام. 

بذلك تكتمل متطلبات **Phase 4 — States & Workflows**، ويكون النظام جاهزًا للانتقال إلى المرحلة التالية: **تصنيف الكيانات والعلاقات (Entity / Relationship Classification)**.