---
title: "كيف تستخدم /arib-build — الدليل العملي بأمثلة حقيقية"
date: 2026-06-22
draft: false
tags: ["Claude Code", "AI Agents", "arib-build", "arib-nestjs", "arib-postgres", "CCM", "ذكاء اصطناعي", "أدوات"]
description: "دليل عملي شامل لاستخدام /arib-build — كيف تكتب الأمر، أمثلة حقيقية بالعربي والإنجليزي، وكيف تجمعه مع Stack Skills لنتائج أعمق."
weight: 1
---

`/arib-build` بسيط في ظاهره — أعطه هدفاً، يُنجزه.  
لكن الفرق بين هدف محكم وهدف فضفاض هو الفرق بين تسليم في دقائق وعمل يحتاج مراجعة.

هذا الدليل يُريك **كيف تكتب الأمر** — لا ماذا يفعل من الداخل.

---

## صيغة الأمر

```
/arib-build <وصف الهدف>
```

**اكتب الهدف كأنك تُكلّف مهندساً أول يعرف الكود:**
- ماذا تريد — لا كيف يُنفَّذ
- الـ scope واضح — لا "صلّح كل شي"
- النتيجة المتوقعة — لا العملية

---

## أمثلة حقيقية — إنجليزي

### مهام محددة (Inline)

```bash
/arib-build add soft-delete to the User entity
```
```bash
/arib-build add rate limiting to the /auth/login endpoint — 5 attempts per minute per IP
```
```bash
/arib-build replace all console.log calls in the payments module with the Logger service
```
```bash
/arib-build add input validation DTOs to the reservations controller
```
```bash
/arib-build write unit tests for the PricingService — cover edge cases for zero-night stays
```

### مهام متوسطة (Workflow)

```bash
/arib-build migrate all API responses to use a unified ResponseDto wrapper
```
```bash
/arib-build add Redis caching to all GET endpoints in the RoomsModule
```
```bash
/arib-build enforce role-based guards across every controller in the HotelOS API
```

### حملة مستمرة (مع /loop)

```bash
/loop /arib-build harden every money-math path in the billing module
```
```bash
/loop /arib-build add missing indexes to the top 10 slow queries in the reservation flow
```

---

## أمثلة حقيقية — عربي

### مهام محددة

```bash
/arib-build أضف حقل soft-delete لجدول المستخدمين مع migration آمنة
```
```bash
/arib-build ضع rate limiting على endpoint تسجيل الدخول — 5 محاولات في الدقيقة لكل IP
```
```bash
/arib-build استبدل كل console.log في موديول المدفوعات بـ Logger service
```
```bash
/arib-build أضف DTOs للـ validation على كل endpoints الحجوزات
```
```bash
/arib-build اكتب unit tests لـ PricingService مع تغطية حالات الإقامة صفر ليالي
```

### مهام متوسطة

```bash
/arib-build حوّل كل responses في الـ API لـ ResponseDto موحّد
```
```bash
/arib-build أضف Redis caching لكل GET endpoints في RoomsModule
```
```bash
/arib-build فرض role-based guards على كل controllers في الـ API
```

### مع /loop

```bash
/loop /arib-build صلّح كل عمليات الحساب المالي في موديول الفواتير
```

---

## كيف تكتب هدفاً جيداً

| ❌ هدف ضعيف | ✅ هدف محكم |
|------------|------------|
| `fix the auth` | `add refresh token rotation to the JWT auth flow` |
| `improve performance` | `add Redis cache to the rooms availability query — TTL 5 min` |
| `add tests` | `write integration tests for the check-in flow — cover overbooking and invalid dates` |
| `clean the code` | `replace magic numbers in the pricing module with named constants` |
| `صلّح المشاكل` | `أصلح الـ N+1 في استعلام قائمة الحجوزات في ReservationsService` |

**القاعدة:** لو قرأ مهندس آخر الأمر وسأل "بس؟ خلاص؟" — الهدف واضح. لو سأل "تقصد إيش بالضبط؟" — اكتب أكثر.

---

## الجمع مع Stack Skills

`/arib-build` ينجز العمل. Stack Skills **تتحقق من جودته** — opt-in دائماً، أنت من يُقرر متى.

### مع `/arib-nestjs`

بعد أي بيلد يمس الـ architecture:

```bash
# أولاً: البيلد
/arib-build add a NotificationsModule with email and SMS providers

# ثانياً: مراجعة المعمار
/arib-nestjs review apps/api/src/modules/notifications
```

```bash
# إنجليزي
/arib-build implement the BookingService with full CRUD and event emission
/arib-nestjs review apps/api/src/modules/bookings

# عربي
/arib-build بنّي BookingService مع CRUD كامل وإرسال events
/arib-nestjs review apps/api/src/modules/bookings
```

`/arib-nestjs` يتحقق من: هل الـ DI صحيح؟ هل الـ DTOs مكتملة؟ هل في ثغرات أمنية في الـ guards؟

### مع `/arib-postgres`

بعد أي بيلد يمس قاعدة البيانات:

```bash
# إنجليزي
/arib-build add full-text search to the guest lookup endpoint
/arib-postgres review    # تحقق من الـ indexes والـ query plan

# عربي
/arib-build أضف full-text search لبحث النزلاء
/arib-postgres review
```

```bash
# مع تحديد
/arib-build add a reports dashboard query for monthly revenue by room type
/arib-postgres index     # اقترح indexes مناسبة للـ query الجديد
```

`/arib-postgres` يكشف: هل في N+1؟ هل الـ migration تُقفل جداول؟ هل الـ index في مكانه الصحيح؟

### مع الاثنين

```bash
# بيلد كبير يمس الكود والقاعدة
/arib-build add a housekeeping task assignment system with real-time status tracking

# مراجعة طبقة المعمار
/arib-nestjs review apps/api/src/modules/housekeeping

# مراجعة طبقة البيانات
/arib-postgres review
/arib-postgres index
```

---

## الجمع مع `/arib-engine`

```
/loop /arib-engine discover missing features    ← يكتشف ما يحتاج بناء
         ↓
/arib-build <كل هدف يكتشفه>                  ← يُنجز كل هدف
```

`/arib-engine` يُجيب على "ماذا نبني؟" — `/arib-build` يُجيب على "كيف نبنيه؟"

---

## الجمع مع `/arib-wave-plan`

لما الهدف كبير وتريد تقفل المتطلبات قبل التنفيذ:

```bash
# أولاً: ضع الخطة
/arib-wave-plan "redesign the multi-property billing system"
# → ينتج waves/<id>/PLAN.md

# ثانياً: نفّذ على الخطة المقفولة
/arib-build @waves/<id>/PLAN.md
```

يُجنّبك أن يُعيد `/arib-build` تفسير الهدف في كل جولة.

---

## سيناريوهات متكاملة

### سيناريو ١: إضافة feature جديدة

```bash
# ١. البيلد
/arib-build add a guest loyalty points system — earn on checkout, redeem on booking

# ٢. مراجعة المعمار
/arib-nestjs review apps/api/src/modules/loyalty

# ٣. مراجعة قاعدة البيانات
/arib-postgres review
```

### سيناريو ٢: تحسين أداء

```bash
# ١. اكتشف الأبطأ
/arib-engine find the top 5 performance bottlenecks in the reservations flow

# ٢. صلّح كل واحد
/arib-build optimize the availability check query — currently scanning 2M rows
/arib-postgres index    # تأكد من الـ indexes
```

### سيناريو ٣: تأمين API موجود

```bash
# ١. البيلد
/arib-build add API key authentication to all public-facing endpoints

# ٢. مراجعة أمنية
/arib-nestjs review apps/api/src/modules/auth

# ٣. loop للتأكد من التغطية الكاملة
/loop /arib-build verify every endpoint in the API has the correct guard applied
```

---

## بوابة الدمج — لا تتغير

في كل الأوامر السابقة، الدمج إلى `main` يستوجب:

- CI أخضر
- `verification-agent` يرجع **RECONCILED**
- أي شي حساس (مال، مصادقة، migration، أسرار) → **HOLD للإنسان**

زيادة تفاصيل الأمر لا تتجاوز البوابة — هي ثابتة بغض النظر عن الهدف.

---

## المصدر

جزء من **CCM v3.18.0 "Compression & Lean"**:

[claude-code-methodology](https://github.com/AribSudia/claude-code-methodology) — `.claude/skills/arib-build/SKILL.md`

---

*آخر تحديث: يونيو 2026 | CCM v3.18.0*
