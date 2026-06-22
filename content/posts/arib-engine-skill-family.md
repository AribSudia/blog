---
title: "عائلة ARIB Skills — من المحرك إلى التخصص"
date: 2026-06-22
draft: false
tags: ["Claude Code", "AI Agents", "arib-engine", "arib-build", "arib-nestjs", "arib-postgres", "CCM", "ذكاء اصطناعي", "أدوات"]
description: "أربع أدوات متكاملة: محرك مستقل، مدير فريق، معمار NestJS، ومحسّن PostgreSQL — كل منها بنُطاق دقيق وحجم يعكس فلسفته."
weight: 1
---

بدأ الأمر بـ `/arib-engine` — محرك يكتشف عمله بنفسه ويُنهيه بالدليل.

ثم جاء `/arib-build` — مدير فريق يأخذ هدفاً محدداً ويُوزّعه على متخصصين بالتوازي.

والآن العائلة اكتملت بطبقة **Stack Skills** — تخصصات عميقة تُزرع في أي مشروع وتعمل مع الأدوات الكبيرة أو بمفردها.

---

## خريطة العائلة

```
┌────────────────────────────────────────────────────────────┐
│                    ARIB Skill Family                        │
├─────────────────────┬──────────────────────────────────────┤
│  Engine Skills       │  Stack Skills (opt-in)              │
│                     │                                      │
│  /arib-engine       │  /arib-nestjs                        │
│  المحرك المستقل     │  معمار NestJS + مراجعة               │
│  discover→verify    │                                      │
│  300 سطر            │  /arib-postgres                      │
│                     │  تحسين PostgreSQL + أمان migrations  │
│  /arib-build        │  85 سطر                              │
│  مدير الفريق        │                                      │
│  decompose→dispatch │                                      │
│  70 سطر             │                                      │
└─────────────────────┴──────────────────────────────────────┘
```

**Engine Skills** — يعمل أياً منهما على أي مشروع بدون تكوين مسبق.

**Stack Skills** — تخصصات عميقة، opt-in صارم، تُفعَّل حين يحتاجها الهدف.

---

## Engine Skills — الأساس

### `/arib-engine` — المحرك المستقل

> أعطه ملكية هدف — يكتشف العمل، يشحنه عبر PRs صغيرة، يتحقق، ويتوقف حين يقول **الدليل** إنه انتهى.

```bash
/loop /arib-engine harden the codebase
# يُجري مسح adversarial: find → refute → confirm
# يُنتج PR صغير قابل للتراجع لكل مشكلة مُتحقَّق منها
# يتوقف لما يُستنفد الـ backlog + trunk أخضر
```

**الفلسفة:** لا يُجيب على "هل انتهينا؟" بسؤال — يُجيب بالدليل.

**الحجم:** 300 سطر — يحمل آلية الاكتشاف الكاملة وبروتوكول الإغلاق.

---

### `/arib-build` — مدير الفريق الهندسي

> أعطه هدفاً محدداً — يُحدّد أصغر وضع تنفيذ مناسب، يُفكّك، يوزّع على متخصصين، يجمع، يُمرّر بوابة الدمج.

```bash
# Inline — هدف محدود
/arib-build add a soft-delete flag to the Patient entity

# Workflow — هدف واسع
/arib-build migrate all 40 controllers to the new auth guard

# /loop — حملة مستمرة
/loop /arib-build harden every money-math path
```

**الفلسفة:** "أصغر آلية تُناسب" — لا Workflow لملف واحد، لا `/loop` لهدف ينتهي في دقيقتين.

**الحجم:** 70 سطر — مُركّز على التوزيع والتجميع فقط.

---

## Stack Skills — التخصص العميق

### `/arib-nestjs` — معمار NestJS + مراجعة

**ما تحلّه:** تكوين الـ modules يتكرر من مشروع لآخر. أنماط الـ DI غير المثلى، DTOs بدون validation صارم، middleware مرتبة خطأ — أخطاء شائعة لا يمسكها الـ type checker.

**ما يغطيه:**

| الجانب | ما يفعله |
|--------|---------|
| **Modules & DI** | بنية صحيحة، circular deps, lazy loading |
| **DTOs + Validation** | class-validator الصحيح، whitelist, transform |
| **Guards / Interceptors / Pipes / Filters** | ترتيب التنفيذ الصحيح |
| **Config & Lifecycle** | ConfigModule، OnModuleInit، graceful shutdown |
| **Data Access** | TypeORM patterns، N+1 الشائع |
| **Testing** | Test module، mock providers |
| **Security Pitfalls** | mass assignment، injection، headers |

```bash
# مراجعة module محدد
/arib-nestjs review apps/api/src/modules/payments

# معمار feature جديدة
/arib-nestjs feature reservations-module

# مراجعة كاملة للمشروع
/arib-nestjs review
```

**الحجم:** 90 سطر — مُكثَّف بدون حشو.

---

### `/arib-postgres` — تحسين PostgreSQL + أمان Migrations

**ما تحلّه:** استعلامات بطيئة بدون سبب واضح، migrations تُقفل جداول في الإنتاج، JSONB يُستخدم بدون إدراك التبعات — كلها مشاكل تظهر بعد فوات الأوان.

**ما يغطيه:**

| الجانب | ما يفعله |
|--------|---------|
| **Indexing** | متى تُنشئ، متى لا، index types الصحيح |
| **EXPLAIN Plans** | قراءة الـ plan، Seq Scan vs Index Scan |
| **N+1 Detection** | يمسك الأنماط في Prisma/TypeORM |
| **Safe Migrations** | lock-aware، CONCURRENTLY، no-downtime |
| **Pooling** | pgBouncer modes، connection limits |
| **JSONB** | متى يُستخدم، متى ينزّل إلى columns |
| **RLS Multi-tenancy** | Row Level Security بشكل صحيح |

```bash
# تحسين استعلام
/arib-postgres tune "SELECT * FROM reservations WHERE..."

# مراجعة migration قبل تطبيقها
/arib-postgres review migration 20260622_add_guests_index

# فحص كامل للـ schema
/arib-postgres index
```

**الحجم:** 85 سطر — يُركّز على الأمان والأداء، لا التوثيق.

---

## كيف تعمل معاً

**سيناريو: تسليم feature جديدة في HotelOS**

```bash
# 1. بيلد مع تخصصات Stack
/arib-build "add smart lock integration to reservation flow"
# ← الـ engineer-manager يُطلق arib-nestjs تلقائياً للـ modules
# ← يُطلق arib-postgres للـ schema changes

# 2. مراجعة مُعمّقة بعد التسليم
/arib-nestjs review apps/api/src/modules/locks
/arib-postgres review migration add_lock_models
```

**سيناريو: حملة تحسين مستمرة**

```bash
# الـ engine يقود، Stack Skills أدواته
/loop /arib-engine --with-arib-family harden the billing module
# يستدعي arib-nestjs لمراجعة كل controller يمسّه
# يستدعي arib-postgres لكل استعلام مالي يُعدّله
```

---

## مبدأ التصميم خلف العائلة

كل skill في العائلة يُجيب على سؤال واحد فقط:

| Skill | سؤاله |
|-------|-------|
| `/arib-engine` | **ماذا** يحتاج العمل؟ |
| `/arib-build` | **كيف** تُسلّمه بالتوازي؟ |
| `/arib-nestjs` | **هل** هذا الـ NestJS صحيح؟ |
| `/arib-postgres` | **هل** هذا الـ SQL آمن وسريع؟ |

لا تداخل في المسؤوليات. كل حجم يعكس نطاق السؤال — لا أقل ولا أكثر.

---

## Stack Skills: قواعد الاستخدام

**opt-in صارم:**
- لا يُفعَّل Stack Skill إلا إذا طُلب صراحةً أو عبر `--with-arib-family`
- `/arib-build` الافتراضي لا يمتد لهم

**مستقلة بالكامل:**
- تعمل بدون Engine Skills
- يمكن استدعاؤها مباشرة على أي module أو migration أو query

**مؤلَّفة بالـ MIT:**
- مكتوبة من الصفر، لا اعتماد على مكتبات خارجية في منطق الـ skill نفسه

---

## المصدر

جزء من **CCM (Claude Code Methodology)**:

[AribSudia/claude-code-methodology](https://github.com/AribSudia/claude-code-methodology)

```
.claude/skills/
├── arib-engine/    SKILL.md  (300 سطر)
├── arib-build/     SKILL.md  (70 سطر)
├── arib-nestjs/    SKILL.md  (90 سطر)
└── arib-postgres/  SKILL.md  (85 سطر)
```

---

*نُشر يونيو 2026*
