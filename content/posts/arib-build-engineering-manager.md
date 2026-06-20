---
title: "/arib-build — فريقك الهندسي في أمر واحد"
date: 2026-06-21
draft: false
tags: ["Claude Code", "AI Agents", "arib-build", "arib-engine", "CCM", "ذكاء اصطناعي", "أدوات"]
description: "أعطه الهدف — يُحدّد وضع التنفيذ المناسب، يُفكّك، يوزّع على المتخصصين، يجمع، ويُغلق. الجديد في v3.18: ثلاثة أوضاع للتنفيذ تتكيف مع حجم الهدف."
weight: 1
---

أحياناً تعرف **بالضبط** ما تريد.

لا تحتاج شيئاً يكتشف المهام — تحتاج شيئاً **يُنجزها**.

هذا ما بُني له `/arib-build`.

---

## الفكرة بجملة واحدة

أعطه هدفاً محدداً — يُحدّد أصغر وضع تنفيذ مناسب، يُفكّك، يوزّع على المتخصصين، يجمع، ويُمرّر بوابة الدمج.

```
size goal → choose mode → decompose → dispatch → integrate → reconcile → merge gate
```

---

## الجديد: ثلاثة أوضاع للتنفيذ

أهم ما جاء في **v3.16.0** — الأداة لا تعمل بطريقة واحدة. تختار **أصغر آلية تُناسب الهدف**:

| الهدف | الوضع | ما يحدث |
|-------|-------|---------|
| تغيير محدود، بضعة متخصصين، جولة واحدة | **Inline** (الافتراضي) | `Task(engineer-manager)` مباشرة |
| واسع / متوازٍ / يحتاج تحقق مكثّف | **Workflow** | الأداة تُطلق Workflow — pipeline بتزامن محدود |
| حملة لا تنتهي في جولة واحدة | **`/loop`** | جولة لكل وحدة، تُغلق لما يجف الـ backlog |

**المبدأ:** اصعد فقط إذا احتجت. لا Workflow لتعديل ملف واحد. لا `/loop` لهدف ينتهي في دقيقتين.

---

## الحلقة الداخلية (الـ engineer-manager)

بعد اختيار الوضع، يأخذ **engineer-manager** زمام الأمور:

```
┌─────────────────────────────────────────────────┐
│                 /arib-build <goal>               │
│                                                 │
│  0. SIZE GOAL  → يختار الوضع المناسب            │
│       ↓                                         │
│  1. DECOMPOSE  → يُفكّك + يبني task graph        │
│       ↓                                         │
│  2. DISPATCH   → متخصصون بالتوازي (حيث آمن)    │
│    Architect  Reviewer  Tester  Security…       │
│       ↓                                         │
│  3. INTEGRATE  → يجمع كل المخرجات              │
│       ↓                                         │
│  4. RECONCILE  → verification-agent يتحقق       │
│       ↓                                         │
│  5. MERGE GATE → الدمج أو HOLD للإنسان          │
└─────────────────────────────────────────────────┘
```

---

## أمثلة عملية

```bash
# Inline — هدف محدود، جولة واحدة
/arib-build add a soft-delete flag to the Patient entity
# architect + planner ▶ database-guardian ▶ implement
# ▶ code-reviewer + security-auditor + test-engineer ▶ RECONCILED
# migration = high-stakes → HOLD for human merge

# Workflow — هدف واسع، يتوازى
/arib-build migrate all 40 controllers to the new auth guard
# → decompose → 40 وحدات → LAUNCH Workflow: pipeline(units, migrate, verify)
# كل وحدة: verification-agent → auto-merge أو HOLD

# /loop — حملة مستمرة
/loop /arib-build harden every money-math path in the billing module
# كل جولة: path واحد → build → reconcile → (auto/HOLD) → re-arm
# تتوقف لما يجف الـ backlog
```

---

## بوابة الدمج — لا تتغير بالوضع

**الوضع يُغيّر "كيف" تُوزَّع المهام — لا "من" يأذن بالدمج.**

في كل الأوضاع الثلاثة، الدمج إلى main يستوجب:
- CI أخضر
- `verification-agent` يرجع RECONCILED
- الفئات الحساسة (مال، مصادقة، migration، أسرار) → **HOLD للإنسان دائماً**

زيادة النطاق لا تُزيد الصلاحية.

---

## الربط مع `/arib-wave-plan` (v3.17)

لما الهدف كبير ومُعقّد، `/arib-wave-plan` يُقفل المتطلبات **قبل** إطلاق البيلد:

```bash
# أولاً: اقفل ما تريد بالضبط
/arib-wave-plan "redesign the checkout flow"
# يُنتج waves/<id>/PLAN.md → قفل المتطلبات

# ثم: نفّذ على الخطة المقفولة
/arib-build @waves/<id>/PLAN.md
```

تفيد حين لا تريد الـ decomposition تُعيد تفسير الهدف في كل جولة.

---

## المقارنة مع `/arib-engine`

هما أخوان — لا بديلان:

| | `/arib-engine` | `/arib-build` |
|--|---------------|---------------|
| **نقطة البداية** | مشكلة مفتوحة | هدف محدّد |
| **يكتشف العمل؟** | ✅ adversarial find→refute→confirm | ❌ أنت تُحدّده |
| **أوضاع التنفيذ** | مع `/loop` دائماً | Inline / Workflow / /loop |
| **"هل انتهينا؟"** | الدليل يُقرّر | الهدف يُقرّر |
| **المناسب لـ** | حملة مفتوحة / اكتشاف | تسليم هندسي محدّد |

**الجمع بينهما:**
```
/loop /arib-engine discover    ← يكتشف الأهداف
         ↓
/arib-build <كل هدف>          ← يُنجز كل هدف بالتوازي
```

---

## متى تختار أيهما؟

**استخدم `/arib-build`** حين:
- الهدف واضح ومحدّد
- عندك مواصفات جاهزة للتنفيذ
- تريد تسليماً سريعاً ومُتحقَّقاً منه

**استخدم `/arib-engine`** حين:
- المشكلة مفتوحة ("صلّح ما يحتاج تصليح")
- تحتاج من يكتشف الأولويات بنفسه
- تريد حملة مستمرة

---

## المصدر

جزء من **CCM v3.18.0 "Compression & Lean"**:

[claude-code-methodology](https://github.com/AribSudia/claude-code-methodology) — `.claude/skills/arib-build/SKILL.md`

---

*آخر تحديث: يونيو 2026 | CCM v3.18.0*
