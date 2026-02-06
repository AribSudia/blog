---
title: "كيف تحصل على OAuth Token لـ OpenClaw"
date: 2026-02-06T23:40:00+03:00
draft: false
tags: ["OpenClaw", "OAuth", "دليل", "تثبيت"]
categories: ["دليل OpenClaw"]
description: "شرح تفصيلي لاستخراج OAuth Token من Claude Code لاستخدامه في OpenClaw"
---

## ما هو OAuth Token؟

OAuth Token هو طريقة للمصادقة مع Anthropic **بدون الحاجة لشراء رصيد API**. إذا عندك اشتراك Claude Pro أو Max، تقدر تستخدم نفس الاشتراك مع OpenClaw عبر OAuth.

---

## المتطلبات

1. **حساب Anthropic** (مجاني أو مدفوع)
2. **Claude Code CLI** مثبت على جهازك
3. **متصفح** لتسجيل الدخول

---

## الخطوة 1: تثبيت Claude Code CLI

إذا ما ثبتته، افتح التيرمنال واكتب:

```bash
npm install -g @anthropic-ai/claude-code
```

تأكد إنه مثبت:

```bash
claude --version
```

> **ملاحظة**: ما تحتاج تثبت تطبيق Claude Desktop — الـ CLI يكفي!

---

## الخطوة 2: تسجيل الدخول عبر OAuth

اكتب في التيرمنال:

```bash
claude login
```

راح يطلع لك خيارين:

```
? How would you like to authenticate?
❯ Login with Anthropic (OAuth)
  Use API key
```

**اختر الخيار الأول**: `Login with Anthropic (OAuth)`

استخدم الأسهم ↑↓ للتنقل واضغط Enter.

---

## الخطوة 3: تسجيل الدخول في المتصفح

1. المتصفح يفتح تلقائياً على صفحة Anthropic
2. سجل دخول بإيميلك (أو أنشئ حساب جديد)
3. اضغط **Authorize** للموافقة على الصلاحيات
4. ارجع للتيرمنال

المفروض يظهر:
```
✓ Successfully logged in
```

> **إذا المتصفح ما فتح تلقائي**: شوف التيرمنال، راح يعطيك رابط — انسخه وافتحه يدوي.

---

## الخطوة 4: استخراج التوكن

التوكنات تتحفظ في ملف مخفي. اكتب:

```bash
cat ~/.claude/.credentials.json
```

راح يطلع لك شي زي كذا:

```json
{
  "oauth": {
    "accessToken": "ant-oa-xxxxxxxxxxxxxxxxxxxx",
    "refreshToken": "ant-rt-xxxxxxxxxxxxxxxxxxxx",
    "expiresAt": "2026-02-07T12:00:00.000Z"
  }
}
```

**انسخ الـ `accessToken`** — هذا اللي تحتاجه!

---

## الخطوة 5: إضافة التوكن لـ OpenClaw

### الطريقة 1: في ملف الإعدادات

افتح ملف `~/.openclaw/config.yaml` وأضف:

```yaml
anthropic:
  oauthToken: "ant-oa-xxxxxxxxxxxxxxxxxxxx"
```

### الطريقة 2: كـ Environment Variable

```bash
export ANTHROPIC_OAUTH_TOKEN="ant-oa-xxxxxxxxxxxxxxxxxxxx"
```

لحفظه دائماً:

```bash
echo 'export ANTHROPIC_OAUTH_TOKEN="ant-oa-xxxxxxxxxxxxxxxxxxxx"' >> ~/.zshrc
source ~/.zshrc
```

---

## ملاحظات مهمة

### ⏰ التوكن له صلاحية
التوكن ينتهي بعد فترة (شوف `expiresAt`). لما ينتهي، سوِّ `claude login` من جديد.

### 🔄 الـ Refresh Token
بعض إعدادات OpenClaw تطلب `refreshToken` أيضاً:

```yaml
anthropic:
  oauthAccessToken: "ant-oa-xxxxxxxxxxxxxxxxxxxx"
  oauthRefreshToken: "ant-rt-xxxxxxxxxxxxxxxxxxxx"
```

### 🔐 لا تشارك التوكن
التوكن مثل كلمة السر — لا تشاركه مع أحد!

---

## ملخص سريع

```bash
# 1. ثبت Claude Code CLI
npm install -g @anthropic-ai/claude-code

# 2. سجل دخول
claude login

# 3. استخرج التوكن
cat ~/.claude/.credentials.json

# 4. انسخ accessToken وحطه في OpenClaw
```

---

## الخطوة التالية

الحين عندك التوكن! ارجع لـ [الدليل الشامل لـ OpenClaw](/posts/openclaw-complete-guide/) وكمّل باقي الإعدادات.

---

> **واجهتك مشكلة؟** تواصل معنا على [ai@arib.sa](mailto:ai@arib.sa)
