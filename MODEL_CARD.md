# بطاقة النموذج | Model Card
## الحالة
READY_FOR_REVIEW — جودة التفسير تحتاج مراجعة بشرية، وليست درجة آلية.
## الغرض والاستخدام | Purpose and use
This is a fictional teaching project on synthetic data, and decision=1 is a simulated review flag, not a refusal. decision=0 is not a safety guarantee or financing approval, and the model must not be used for real financing decisions.
بيانات Tamweel Lite اصطناعية؛ الهدف حدث خلال 90 يومًا بعد الطلب. 22 خاصية متاحة وقت الطلب؛ لا معرفات أو تواريخ أو هدف في المدخلات. الاستخدام التعليمي فقط؛ لا قرارات تمويل فعلية.
## البيانات والتحقق | Data and validation
حوض التدريب والاختيار: 6576 طلبًا. المعايرة: 836 طلبًا و78 موجبًا. حجز عملاء المعايرة، 90 يومًا لنضج التسميات، وOOF أمامي متداخل مع فصل العملاء في المستويين. طيات المقارنة: 2023Q1 و2023Q3 و2024Q1. نستبعد النتائج التي لم تنضج قبل الأدوار التالية؛ آخر بيانات التدريب لا تستخدم تلقائيًا.
Evidence comes from nested forward OOF over 2023Q1, 2023Q3 and 2024Q1 (2,155 OOF rows), with 90-day label maturity and customer separation, and the ensemble weights and stacker were learned on inner OOF only. It is development evidence from only three overlapping periods, not an independent final test, and the fold SD is descriptive, not a confidence interval.
## النموذج والقرار | Model selection
KEEP SINGLE / Logistic. انحراف AP المرجعي: 0.029808. ارجع إلى artifacts/ensemble_comparison.csv وday5_fold_scores.csv للأرقام الكاملة.
The best single model is Logistic Regression (mean AP 0.3917, fold SD 0.0298), and no ensemble passed the gate: the weighted average changed AP by -0.0022, the equal average by -0.0200 and the stack by -0.0085, none exceeding the fold SD. The stack also worsened ECE by +0.0122 (limit 0.01) and Brier by +0.0028, so the decision is KEEP SINGLE (Logistic) and the extra complexity did not earn its place.
## المعايرة | Calibration
Sigmoid على عينة محجوزة من التدريب والاختيار؛ الرسم والمقاييس تشخيص على عينة تعلم المعاير، وليسا اختبارًا مستقلاً. لا ادعاء بتحسن على تحدٍّ مجهول التسميات.
A sigmoid was learned on the reserved July-September 2024 calibration period (836 rows, 78 positives), and the metrics are fit diagnostics on those same rows, not independent performance. On these rows the sigmoid did not improve the raw Logistic scores (Brier 0.0765 to 0.0781, ECE 0.0211 to 0.0349, log-loss 0.2665 to 0.2773), and no improvement is claimed on the unlabeled challenge.
## السياسة والسعة والمناطق | Policy and regions
خسارة 10 FN + FP، عتبة OOF الخام 0.16892161427109176 والمنقولة 0.12225843144286948. سعة الدفعة 300؛ المرشحون 330؛ الإشارات النهائية 300. كتلة الدرجات المتساوية لا تقسم. 1=إشارة مراجعة تعليمية، 0=عدم رفع الإشارة.
The 12% capacity was applied once to the full 2,500-request batch: 330 requests exceeded the transported threshold (0.1223) and 300 were flagged, with 30 removed by the capacity and equal-score blocks never split. Regional OOF flag rates among negatives range from 6.31% (eastern, 507 negatives) to 10.29% (western, 486 negatives), a gap of 3.98 percentage points, which is a descriptive diagnostic that needs review and not a fairness certification.
## التفسير وحدوده | Explanation scope
تفسير اليوم الرابع يخص نموذج اليوم الرابع؛ لا يُنسب تلقائيًا إلى هذه النسخة. تغيير النموذج أو خصائصه أو معايرته يستلزم مراجعة التفسير.
The Day 4 SHAP explanations belong to the Day 4 weighted LightGBM, while the final model is Logistic Regression, a different model family, so they do not transfer. Feature importance and explanations would have to be recomputed for the final model (for example from its coefficients) before any explanation is attributed to it.
## المتابعة والقيود | Monitoring and limitations
Monitor per batch the flag volume against the 12% capacity (300 per 2,500 requests), the score distribution and prevalence for drift, calibration (Brier and ECE on newly matured labels), and regional flag rates with their denominators. Retrain, recalibrate and re-select the threshold together if any of these shift.
OOF يستخدم للاختيار، وثلاث فترات ليست اختبار دلالة. العتبة قد تتغير سعتها عند نقلها إلى نموذج معاد التدريب. لا تسميات للتحدي، ولا مقاييس أداء أو شهادة عدالة له. البيانات لا تمثل أشخاصًا أو مناطق حقيقية.
## إعادة الإنتاج | Reproducibility
seed=211; trees=80; CPU مجاني. الإصدارات في artifacts/environment.json. المصادر/بصماتها في artifacts/day5_run.json. النموذج artifacts/final_model؛ inference.predict يعيد ID واحتمالًا؛ السياسة تطبق بعد جمع الدفعة. replay_final يعيد التنبؤ المحفوظ؛ rebuild_final يعيد التدريب. لا تدرب النموذج بعد تثبيت المعاير.
## ملكيتك للتسليم | Submission ownership
أكمل أدلة الأيام السابقة والعرض، واحفظ الدفتر المنفذ، ثم سجل SHA وtag مستودعك في قناة التسليم الخاصة. دعم الدورة f486fc50dd9ac8403016facc58cf6a62beb4abf4 ليس SHA تسليمك.
