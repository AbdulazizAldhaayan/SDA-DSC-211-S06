# تقريرك: التفسير والمعايرة — Tamweel Lite

**الحالة:** جاهز للمراجعة؛ لا يعني اعتمادًا أو درجة

**مصدر التفسير:** LIVE. **النموذج والمعايرة:** LIVE. **السعة:** CAPACITY_REVIEW_REQUIRED.

## النموذج والأدوار
LightGBM موزون، 80 شجرة. الهدف حدث تعثر اصطناعي خلال90 يومًا بعد الطلب. الأدوار منفصلة زمنيًا وبالعملاء: تدريب 2516، معايرة 584 (40 موجب)، سياسة 589، تقييم 1733. الفجوات والتداخلات مستبعدة. سبق استخدام بيانات التقييم في الدورة، فهي ليست اختبارًا نهائيًا لم يمسّ.

## التفسير العام والمحلي
Permutation يقيس انخفاضAP على التقييم؛ إشارات المنطقة تُبدّل معًا. SHAP يفسر النموذج الخام بوحدةlog-odds وخلفية مسارات أشجار التدريب. base+sum(SHAP)=raw margin، ثمsigmoid للمجموع فقط. القيم ليست نقاط احتمال ولا تفسيرًا مباشرًا للنموذج المعاير.

Globally, bureau_score is the most important feature in both permutation importance (AP drop 0.1270, on all 1,733 evaluation rows) and the SHAP beeswarm (mean |SHAP| 0.9042 log-odds, on a 300-row sample), followed by dti (0.0690 and 0.5437). Locally, the waterfall for request TR-009585 explains only that one request, where bureau_score (+2.2693 log-odds) and dti (+1.1981) push the score up the most.

SHAP values are in raw log-odds: raw margin = base value + sum of SHAP, then raw probability = sigmoid(raw margin), so contributions are not probability points and must not be transformed separately. The background is the training tree-path distribution, so sigmoid(base_value) is not necessarily the mean predicted probability.

الطلب الاصطناعي TR-009585: الدرجة الخام 0.90308 والاحتمال المعاير 0.47952. اختير أعلى درجة داخل عينةSHAP دون استخدام النتيجة الفعلية.
- استخدم النموذج درجة ائتمانية اصطناعية عند الطلب بالقيمة 497 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+2.2693 log-odds؛ قيمة معوضة: False)
- استخدم النموذج نسبة الالتزام مع القسط المقترح إلى الدخل بالقيمة 1.2806 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+1.1981 log-odds؛ قيمة معوضة: False)
- استخدم النموذج مبلغ التمويل المطلوب بالقيمة 93437.3 لرفع درجته الخام؛ لا يثبت ذلك السببية. (SHAP=+0.2007 log-odds؛ قيمة معوضة: False)

The three reasons (bureau_score 497, dti 1.2806, loan_amount_sar 93,437.29) are the largest positive SHAP contributions of one request, none was imputed, and they describe the fitted model, not causes. The bureau_score +/-1 check kept the same three reasons and the same raw probability (0.9031), but it is a narrow local check, and correlated features can share signal.

## دليل المعايرة
على 1733 صفًا و139 موجب: Brier 0.113027 → 0.067112؛ ECE 0.146871 → 0.022486. عشر حاويات متساوية العرض مع أعدادها فيday4_reliability_bins.csv. AP 0.258677 → 0.258677؛ ROC-AUC 0.770804 → 0.770804. هذه نتائج هذه العينة وليست ضمانًا لتحسن مستقبلي.

On 1,733 evaluation rows (139 positives), sigmoid calibration lowered Brier from 0.113027 to 0.067112 (-0.045915) and ECE from 0.146871 to 0.022486 (-0.124385), while ROC-AUC (0.7708) and AP (0.2587) stayed the same, so it improved probability quality without changing ranking. For the top request the raw score 0.9031 maps to a calibrated 0.4795, so raw scores are not probabilities, and with only 139 positives in one evaluation window this is not a guarantee for future periods.

## الاستقرار
200 تكرارbootstrap صالح بسحب العملاء؛ فترات مئينية95% مع تثبيت النموذج والمعاير. لا تشمل تعلم النموذج أو المعايرة أو الانجراف المستقبلي، ولا تصف احتمال فرد. انحرافAP بين ربعي التقييم وصفي فقط. اختبارbureau_score±1 نُفذ؛ راجع day4_local_stability.csv.

The paired customer-cluster bootstrap (200 replicates) gives 95% intervals of [0.1974, 0.3379] for AP (identical for raw and calibrated, since a monotone sigmoid preserves ranking) and [-0.0542, -0.0374] for Brier change, with the model and calibrator fixed. It excludes training, calibration-fitting and future-drift uncertainty, and it is not an interval for one applicant's probability.

## العتبة ومنطقة المراجعة
العتبة الخام 0.5881953696965011 اختيرت علىpolicy بخسارة10×FN+FP وسقف12% ثم نُقلت إلى 0.17331013263107387. لم تعدل باستخدام التقييم. المنطقة[0.15331, 0.19331] تشخيصية بعرض±0.02 وليست فترة ثقة. الاتحاد يحسب الطلب مرة واحدة.
- 2024Q3: السقف 100، الإشارات 97، اتحاد المراجعة 109.
- 2024Q4: السقف 107، الإشارات 109، اتحاد المراجعة 122.

The raw threshold 0.588195, selected on the policy role (68 flags, loss 387, capacity 70), was transported to a calibrated threshold of 0.17331. On evaluation, 2024Q3 has 97 risk flags within its capacity of 100 but 109 once near-threshold cases are added, and 2024Q4 already has 109 risk flags above its capacity of 107 (122 with the band), so the status is CAPACITY_REVIEW_REQUIRED and I did not retune the threshold on evaluation.

عند تجاوز السعة، وثّق الحاجة إلى تصميم سياسة جديدة على بيانات تطوير وتقييمها بدليل جديد. لا ترفع السقف ولا تقص الحالات بعد رؤية النتيجة. التفسير ليس سببية أو شهادة عدالة، والخسارة وحدات تعليمية لا رسوم أو خصم درجات. لا يستخدم هذا التمرين لتمويل حقيقي.
