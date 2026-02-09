---
title: "التغريد بدون API — كيف تتجاوز قيود تويتر"
date: 2026-02-09
draft: false
tags: ["twitter", "automation", "javascript", "graphql"]
description: "طريقة للتغريد برمجياً بدون الحاجة لـ API الرسمي أو مكتبات خارجية"
---

## المشكلة

تويتر (X) يضع قيوداً صارمة على الـ API الرسمي:
- **17 تغريدة/يوم** في الباقة المجانية
- **مكتبات مثل twikit**: محظورة من Cloudflare
- **أدوات CLI مثل bird**: تُكتشف كـ automation

حتى لو استخدمت الكوكيز الصحيحة مع مكتبات HTTP مثل `httpx` أو `requests`، تويتر يكتشفها عبر **TLS Fingerprinting** — يعرف إن الطلب مو من متصفح حقيقي.

## الحل: fetch() من داخل المتصفح

بدل ما ترسل طلبات HTTP من كود خارجي، خلّي **المتصفح نفسه** يرسل الطلب!

### لماذا تنجح؟

1. ✅ **TLS Fingerprint حقيقي** — المتصفح يستخدم نفس التوقيع اللي يتوقعه تويتر
2. ✅ **الكوكيز تلقائية** — `credentials: 'include'` يرسل كل الكوكيز
3. ✅ **Headers صحيحة** — المتصفح يضيف headers إضافية تلقائياً
4. ✅ **لا حاجة لـ API Key** — تستخدم جلستك العادية

## الكود

### الخطوة 1: احصل على queryId

تويتر يغير الـ `queryId` مع كل تحديث. هذا الكود يجيبه تلقائياً:

```javascript
async function getQueryId() {
  const html = await fetch('https://x.com/home').then(r => r.text());
  const match = html.match(/main\.[a-f0-9]+\.js/);
  if (!match) throw new Error('main.js not found');
  
  const jsUrl = 'https://abs.twimg.com/responsive-web/client-web/' + match[0];
  const js = await fetch(jsUrl).then(r => r.text());
  const m = js.match(/queryId:"([^"]+)",operationName:"CreateTweet"/);
  
  return m ? m[1] : null;
}
```

### الخطوة 2: أرسل التغريدة

```javascript
async function tweet(text) {
  // احصل على CSRF token من الكوكيز
  const ct0 = document.cookie
    .split('; ')
    .find(c => c.startsWith('ct0='))
    ?.split('=')[1];
  
  const qid = await getQueryId();
  
  const features = {
    communities_web_enable_tweet_community_results_fetch: true,
    c9s_tweet_anatomy_moderator_badge_enabled: true,
    tweetypie_unmention_optimization_enabled: true,
    responsive_web_edit_tweet_api_enabled: true,
    graphql_is_translatable_rweb_tweet_is_translatable_enabled: true,
    view_counts_everywhere_api_enabled: true,
    longform_notetweets_consumption_enabled: true,
    responsive_web_twitter_article_tweet_consumption_enabled: true,
    tweet_awards_web_tipping_enabled: false,
    creator_subscriptions_quote_tweet_preview_enabled: false,
    longform_notetweets_rich_text_read_enabled: true,
    longform_notetweets_inline_media_enabled: true,
    articles_preview_enabled: true,
    rweb_video_timestamps_enabled: true,
    rweb_tipjar_consumption_enabled: true,
    responsive_web_graphql_exclude_directive_enabled: true,
    verified_phone_label_enabled: false,
    freedom_of_speech_not_reach_fetch_enabled: true,
    standardized_nudges_misinfo: true,
    tweet_with_visibility_results_prefer_gql_limited_actions_policy_enabled: true,
    responsive_web_graphql_skip_user_profile_image_extensions_enabled: false,
    responsive_web_graphql_timeline_navigation_enabled: true,
    responsive_web_enhance_cards_enabled: false,
    post_ctas_fetch_enabled: true,
    responsive_web_grok_community_note_auto_translation_is_enabled: false,
    responsive_web_grok_share_attachment_enabled: false,
    responsive_web_grok_analyze_post_followups_enabled: false,
    responsive_web_grok_analysis_button_from_backend: false,
    premium_content_api_read_enabled: false,
    responsive_web_grok_analyze_button_fetch_trends_enabled: false,
    responsive_web_grok_image_annotation_enabled: false,
    responsive_web_profile_redirect_enabled: false,
    responsive_web_grok_annotations_enabled: false,
    responsive_web_grok_show_grok_translated_post: false
  };
  
  const payload = {
    variables: {
      tweet_text: text,
      dark_request: false,
      media: { media_entities: [], possibly_sensitive: false },
      semantic_annotation_ids: []
    },
    features,
    queryId: qid
  };
  
  const resp = await fetch(`https://x.com/i/api/graphql/${qid}/CreateTweet`, {
    method: 'POST',
    headers: {
      'content-type': 'application/json',
      'x-csrf-token': ct0,
      'x-twitter-auth-type': 'OAuth2Session',
      'x-twitter-active-user': 'yes',
      'authorization': 'Bearer AAAAAAAAAAAAAAAAAAAAANRILgAAAAAAnNwIzUejRCOuH5E6I8xnZz4puTs%3D1Zv7ttfk8LF81IUq16cHjhLTvJu4FA33AGWWjCpTnA'
    },
    credentials: 'include',
    body: JSON.stringify(payload)
  });
  
  const data = await resp.json();
  const tweetId = data?.data?.create_tweet?.tweet_results?.result?.rest_id;
  
  return tweetId 
    ? `https://x.com/i/status/${tweetId}`
    : data;
}

// استخدام
tweet('مرحباً من JavaScript! 🚀');
```

## كيف تستخدمها؟

### الطريقة 1: Developer Console
1. افتح x.com وسجل دخول
2. افتح Developer Tools (F12)
3. الصق الكود في Console
4. نادِ `tweet('نص التغريدة')`

### الطريقة 2: Browser Extension
اصنع extension يحقن الكود في صفحات x.com

### الطريقة 3: Puppeteer / Playwright
```javascript
// افتح المتصفح
const browser = await puppeteer.launch({ headless: false });
const page = await browser.newPage();
await page.goto('https://x.com/home');

// انتظر تسجيل الدخول يدوياً أو استخدم كوكيز محفوظة

// نفذ الكود
const result = await page.evaluate(async (text) => {
  // الصق كود tweet() هنا
  return await tweet(text);
}, 'تغريدة من Puppeteer!');
```

### الطريقة 4: مع OpenClaw
إذا تستخدم [OpenClaw](https://openclaw.ai)، المتصفح المدمج يدعم `evaluate`:

```
browser → act → evaluate → الكود
```

## ملاحظات مهمة

⚠️ **لا تسيء الاستخدام** — هذا للأتمتة الشخصية، مو للسبام

⚠️ **الـ queryId يتغير** — لازم تجيبه ديناميكياً

⚠️ **الـ features تتغير** — تويتر يضيف features جديدة أحياناً (راقب الأخطاء)

⚠️ **Rate Limits** — تويتر عنده حدود حتى للمستخدمين العاديين

## الخلاصة

المتصفح هو أفضل "مكتبة HTTP" للتعامل مع المواقع اللي تستخدم anti-bot measures. بدل ما تحاول تحاكي المتصفح، **استخدم المتصفح نفسه**.

---

*هل جربت الطريقة؟ شاركني تجربتك!*
