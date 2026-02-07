---
title: "كيف تعرض OAuth Token الحالي في OpenClaw"
date: 2026-02-07T23:45:00+03:00
draft: false
tags: ["OpenClaw", "OAuth", "Token", "دليل"]
categories: ["دليل OpenClaw"]
description: "طريقة سريعة لعرض التوكن المستخدم حالياً في OpenClaw"
---

هل تريد معرفة التوكن المستخدم حالياً في OpenClaw؟ إليك الطريقة.

---

## الطريقة السريعة

افتح التيرمنال واكتب:

```bash
cat ~/.openclaw/agents/main/agent/auth-profiles.json | jq -r '.profiles["anthropic:default"].token'
```

ستظهر لك نتيجة مشابهة لهذا:

```
sk-ant-oat01-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## إذا لم يكن jq مثبتاً

يمكنك استخدام `grep` بدلاً منه:

```bash
grep -o '"token": "[^"]*"' ~/.openclaw/agents/main/agent/auth-profiles.json
```

أو عرض الملف كاملاً:

```bash
cat ~/.openclaw/agents/main/agent/auth-profiles.json
```

---

## مسار ملف التوكن

التوكن محفوظ في:

```
~/.openclaw/agents/main/agent/auth-profiles.json
```

---

## أنواع التوكنات

| النوع | البداية | المصدر |
|-------|---------|--------|
| OAuth Token | `sk-ant-oat01-` | `claude setup-token` (اشتراك Pro/Max) |
| API Key | `sk-ant-api-` | Anthropic Console |

---

## متى تحتاج التوكن؟

- **نقل الإعدادات** لجهاز آخر
- **استكشاف مشاكل المصادقة**
- **التحقق من نوع التوكن** المستخدم

---

## ⚠️ تحذير أمني

- **لا تشارك التوكن مع أحد** — يعمل مثل كلمة السر
- **لا تنشره في مكان عام** (GitHub, Twitter, إلخ)
- إذا تسرب، جدده فوراً عبر `claude login` ثم `claude setup-token`

---

*للمزيد: [كيف تحصل على OAuth Token](/posts/openclaw-oauth-token/)*
