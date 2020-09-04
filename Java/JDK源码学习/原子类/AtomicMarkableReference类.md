 # 简介
 `AtomicMarkableReference` 维护一个boolean类型的标记，标记值有修改，对标记是否修改来解决是否哟ABA问题。`AtomicStampedReference`通过维护一个版本号来解决ABA问题。