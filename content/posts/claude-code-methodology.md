---
title: "Claude Code Methodology — نظام تشغيل للتطوير بالذكاء الاصطناعي"
date: 2026-06-17
draft: false
tags: ["Claude Code", "CCM", "AI Development", "Methodology", "منهجية", "أدوات", "ذكاء اصطناعي"]
description: "CCM ليس runtime ولا orchestrator — هو طبقة اتفاقيات تجعل جلسات Claude Code المتعددة دائمة وموثوقة."
weight: 1
---

> **v3.11.0 "Engine"** · صُمّم من عبدالله × Claude

---

## ما هو CCM؟

**Claude Code Methodology** طبقة اتفاقيات للعمل الجاد في Claude Code.

ليس runtime. ليس orchestrator. ليس kernel.

هو **مجموعة اتفاقيات** تجعل عمل Claude Code عبر جلسات متعددة **دائماً ومتسقاً**.

---

## ماذا يحتوي؟

| المكوّن | العدد | الوظيفة |
|---------|-------|---------|
| `/arib-*` skills مُسمّاة | 27 | كل skill تنفّذ دوراً محدداً |
| Specialist agents | 15 | عملاء متخصصون قابلون للاستدعاء |
| Enforcement hooks | — | تطبيق على مستوى kernel |
| Path-scoped rules | — | قواعد حسب مسار الملف |
| Persistent memory files | — | ذاكرة تعيش بين الجلسات |
| Bootstrap protocols | 5 أوضاع | بدء تشغيل موحّد |
| Wave delivery overlay | — | تنفيذ تلقائي للخطوات |
| Compliance layer | — | امتثال وحوكمة |
| CI/PR governance | — | إدارة كاملة للـ PRs |

---

## الإنجاز الأبرز: 84% تخفيض في التكلفة

في v3.8.0، قُلّص context بداية الجلسة من **~45.9K token إلى ~7.4K** — أقل من الهدف البالغ 8K لأول مرة.

**كيف؟**
- فقط brain الرئيسي + القواعد الصارمة + الحالة الحالية تبقى دائمة
- كل وثيقة مرجعية (DECISIONS، SECURITY، ERROR_PATTERNS، schemas) تُحمَّل **عند الطلب** عبر الـ skill أو agent الذي يحتاجها

---

## طبقة الإنفاذ

قبل v3.7، كانت الـ block() تستخدم `exit 1` — وهو ما يُعامله Claude Code **كتحذير لا كحظر**.

في v3.7: تغيير جوهري.

```bash
# قبل — غير فعّال
exit 1  # Claude Code يتجاهله

# بعد — يُغلق فعلاً
exit 2  # يُوقف التنفيذ
```

الآن البوابات التالية تعمل فعلاً:
- كشف الأسرار (secrets)
- أوامر bash الخطرة
- فحوصات OWASP
- بوابات دمج الـ wave

---

## Wave Overlay — الأمواج التنفيذية

قبل v3.6: كل خطوة في الـ wave تنتظر "هل أكمل؟"

بعد v3.6: `/arib-wave-run` يقرأ عقد Steps في PLAN ويتقدم تلقائياً.

**يتوقف فقط عند:**
- خطوة فاشلة
- `checkpoint: true` صريح (عمليات لا يمكن التراجع عنها)
- غموض حقيقي
- عائق
- انقطاع من المستخدم

commit واحد لكل خطوة. `/arib-wave-end` هو بوابة النهاية الصريحة.

---

## نظام الذاكرة

CCM يحل مشكلة أساسية: **Claude يبدأ من صفر في كل جلسة**.

الحل: ملفات ذاكرة دائمة تعيش في الـ repo:

```
CLAUDE.md          ← brain الرئيسي (دائماً محمّل)
project_status.md  ← الحالة الحالية
DECISIONS.md       ← قرارات معمارية (يُحمَّل عند الطلب)
ERROR_PATTERNS.md  ← أخطاء مسبقة (يُحمَّل عند الطلب)
SECURITY.md        ← سياسات الأمان (يُحمَّل عند الطلب)
```

---

## ADRs — Architecture Decision Records

كل قرار معماري موثّق:

| ADR | القرار |
|-----|--------|
| ADR-026 | اعتماد AEPG methodology + /arib-engine |
| ADR-019 | تخفيض token إلى <8K |
| ADR-016/017 | طبقة الإنفاذ الحقيقية |
| ADR-015 | Wave auto-execution |
| ADR-014 | PROTOCOL_PRINCIPLES + drift detection |

---

## مسار الهجرة

CCM يدعم الهجرة من أي بيئة:
- Cursor
- Windsurf
- GitHub Copilot
- Kiro
- أي نظام قديم

---

## كيف تبدأ؟

```bash
git clone https://github.com/AribSudia/claude-code-methodology
cd claude-code-methodology
```

ثم اتبع BOOTSTRAP_PROTOCOL المناسب لوضعك الحالي.

الـ Situation Router يختار البروتوكول تلقائياً — لم تعد تحتاج لاختياره يدوياً.

---

## الفلسفة

> "It is not a runtime, an orchestrator, or a kernel —  
> it is a set of conventions that make multi-session Claude Code work durable."

القوة في البساطة: اتفاقيات واضحة + تطبيق صارم + ذاكرة دائمة = تطوير بالذكاء الاصطناعي يمكن الاعتماد عليه.

---

**المستودع:** [github.com/AribSudia/claude-code-methodology](https://github.com/AribSudia/claude-code-methodology)

*نُشر يونيو 2026*
