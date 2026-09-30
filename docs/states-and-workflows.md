# الحالات ومسارات العمل | States & Workflows — مشروع عصبون (osboon)

## 1. الهدف
تحديد الحالات ومسارات العمل الرئيسية في المشروع، وتوضيح الـTriggers والانتقالات المسموحة على المستوى المفاهيمي. لا تتناول الوثيقة Database أو Backend أو APIs.

## 2. قواعد عامة
- الـState تصف حالة مفهومية مستقرة، وليست مجرد إجراء في الواجهة.
- كل انتقال له Trigger واضح.
- وجود Course داخل Workspace لا يعني أنها منشورة أو معتمدة في Public Library.
- تعديل المحتوى داخل Workspace لا ينشئ Course Version عامة تلقائيًا.
- إنشاء Course Version جديد يحدث بعد اعتماد Update Proposal.
- لا يتم تحديث Workspaces تلقائيًا عند صدور إصدار جديد.

## 3. Account & Workspace
### Account
الحالات: `Created → Active`.
- Created: تم إنشاء Account وربطه بـIdentity.
- Active: Account متاح للاستخدام.

### Workspace
الحالات: `Created → Active`.
- Created: تم إنشاء Workspace.
- Active: Workspace متاحة للعمل.

## 4. Course داخل Workspace
الحالات المفاهيمية: `Local → Linked → Locally Modified`.
- Local: Course مملوكة لسياق Workspace ولم تُربط بإصدار من Public Library.
- Linked: Course تشير إلى إصدار محدد من Public Library.
- Locally Modified: يوجد تعديل خاص على ملف أو أكثر دون أن يصبح إصدارًا عامًا.
الانتقالات: `Local → Linked` عند إضافة Course من Public Library، و`Linked → Locally Modified` عند تعديل ملف مشترك، ويمكن العودة إلى المرجع المشترك عند التخلي عن التعديل الخاص.

## 5. Public Library Course
مسار النشر الأولي:
`Draft in Workspace → Proposed for Publication → Under Review → Published`.
- Draft in Workspace: Course موجودة داخل Workspace قبل النشر.
- Proposed for Publication: المستخدم قدم Course للنشر العام.
- Under Review: المقترح دخل دورة المراجعة.
- Published: Course أصبحت معتمدة ومتاحة في Public Library.
بعد النشر لا يمكن للمستخدم العادي حذف المحتوى المنشور؛ إدارة منصة عصبون هي الجهة المخولة بإزالته وفق سياسات المنصة.

## 6. Update Proposal
المسار:
`Draft Modification → Update Proposed → Under Review → Rejected / Approved → New Version`.
- Draft Modification: تعديل خاص داخل Workspace.
- Update Proposed: تم تقديم التعديل للمكتبة.
- Under Review: المقترح في المراجعة الجماعية.
- Rejected: لم يعتمد التعديل.
- Approved: اعتمد التعديل.
- New Version: تم تحويل التحديث المعتمد إلى Course Version جديد.
الإصدار الجديد لا ينسخ Course كاملة؛ يحتفظ بالملفات التي تغيرت فقط ويعيد استخدام الملفات غير المتغيرة.

## 7. Review Invitation
المسار:
`Invited → Accepted → Reviewer`
أو:
`Invited → Declined / Expired`.
- Invited: أرسلت المنصة دعوة للمستخدم.
- Accepted: قبل المستخدم الدعوة.
- Reviewer: أصبح المستخدم مراجعًا فعليًا.
يُحتسب المستخدم مراجعًا عند قبول الدعوة، وليس بمجرد اختياره.

## 8. Collaborative Review
الحالات:
`Not Started → In Review → Decision → Rejected / Approved`.
- Not Started: دورة المراجعة لم تبدأ.
- In Review: المراجعة الجماعية جارية.
- Decision: وصلت العملية لمرحلة القرار.
- Rejected / Approved: نتيجة المراجعة.
تختار المنصة المراجعين وفق آلية الاختيار المعتمدة؛ تفاصيل الخوارزمية خارج نطاق هذه الوثيقة.

## 9. Course Version
الحالات:
`Created → Published → Superseded`.
- Created: تم إنشاء الإصدار نتيجة Update Proposal معتمد.
- Published: الإصدار متاح في Public Library.
- Superseded: يوجد إصدار أحدث، مع بقاء الإصدار محفوظًا وقابلًا للاختيار وفق السياسة المعتمدة.
الإصدار الجديد ليس نسخة كاملة من Course؛ يتم الاحتفاظ بالملفات التي تغيرت فقط.

## 10. Workspace Version Reference
المسار:
`Version Selected → Pinned → New Version Available → User Requests Upgrade → New Version Pinned`.
- Version Selected: المستخدم اختار إصدارًا.
- Pinned: Workspace تشير إلى الإصدار المحدد.
- New Version Available: صدر إصدار أحدث دون تغيير المرجع الحالي.
- User Requests Upgrade: المستخدم طلب الانتقال.
- New Version Pinned: أصبح الإصدار الجديد مرجع Workspace.
الإصدار الأحدث لا يفرض تحديث Workspace تلقائيًا.

## 11. Private Modification
المسار:
`Shared → Private Copy → Modified`.
- Shared: الملف يشير إلى المحتوى المشترك في الإصدار المرجعي.
- Private Copy: أنشئت نسخة خاصة لهذا الملف.
- Modified: النسخة الخاصة أصبحت معدلة.
النسخة الخاصة لا تدخل Version History العامة ما لم يقدم المستخدم تحديثًا ويُعتمد.

## 12. Publication Workflow
`Course in Workspace → Publication Proposal → Classification Information → Review → Approved → Published Course → Course Version`.
قد تطلب المنصة معلومات تصنيفية مثل الدولة أو الجامعة أو المستوى التعليمي لمساعدة الاكتشاف. التفاصيل لم تحسم بعد، لذلك لا تثبت هنا كحقول أو متطلبات تنفيذية.

## 13. Update Workflow
`Published Version → Private Modification → Update Proposal → Collaborative Review → Rejected / Approved → New Course Version`.
الإصدار الجديد لا يستبدل السابق، ولا يحذف تاريخ الإصدارات، ولا يفرض نفسه تلقائيًا على Workspaces الحالية.

## 14. Ownership & Deletion
عند اعتماد Course في Public Library تنتقل ملكية المحتوى المنشور إلى Public Library ويصبح أصلًا عامًا للمكتبة. الناشر لا يملك حذف المحتوى المنشور، وإدارة عصبون هي الجهة المخولة بإزالته.
التعديل المحلي لا يغير الأصل المنشور؛ مشاركته تتطلب Update Proposal ومراجعة واعتمادًا.

## 15. انتقالات لا تحدث تلقائيًا
- إصدار جديد لا يغير الإصدار المثبت في Workspace تلقائيًا.
- تعديل Workspace لا يغير Public Library.
- تعديل ملف مشترك لا ينشئ Version عامة تلقائيًا.
- رفض Update Proposal لا يحذف النسخة الخاصة من Workspace.
- وجود New Version لا يعني اعتمادها تلقائيًا من المستخدم.

## 16. نقاط تحتاج قرارًا لاحقًا
- العدد المطلوب من المراجعين.
- كيفية احتساب قرار المراجعة الجماعية.
- مدة Review Invitation وانتهائها.
- شروط أهلية المراجع.
- تعارض المصالح للمراجع.
- إعادة إرسال Update Proposal بعد الرفض.
- سياسات إزالة Course المنشورة بواسطة إدارة المنصة.
- تفاصيل مقارنة الإصدارات على مستوى الملفات والدروس.
- Classification Information الخاصة بالاكتشاف.

## 17. الخلاصة
تثبت هذه الوثيقة المسارات الأساسية بين Workspace وPublic Library وCollaborative Review وCourse Versions، وتفصل بين التعديل المحلي والتحديث المنشور. وهي مرجع مفاهيمي للحالات ومسارات العمل قبل الانتقال إلى Entity / Relationship Classification.