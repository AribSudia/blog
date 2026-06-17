---
title: "دليل MCP الشامل — Model Context Protocol"
date: 2026-06-17
draft: false
tags: ["MCP", "Model Context Protocol", "Anthropic", "AI Agents", "ذكاء اصطناعي", "دليل", "API"]
description: "الدليل الشامل لـ Model Context Protocol — المعيار المفتوح الذي يُتيح لنماذج الذكاء الاصطناعي التفاعل مع الأدوات والخدمات الخارجية"
weight: 1
---

> بروتوكول السياق للنماذج

---

## ما هو MCP؟

**Model Context Protocol** هو معيار مفتوح المصدر أطلقته شركة **Anthropic** عام 2024، يُعرّف لغة اتصال موحّدة تُتيح لنماذج الذكاء الاصطناعي التفاعل مع الأدوات والخدمات الخارجية.

### المشكلة التي يحلّها

قبل MCP، كان كل نموذج ذكاء اصطناعي يحتاج إلى integration مخصصة مع كل أداة:

```
N نماذج × M أداة = N×M integration مختلفة
```

**مع MCP:**

```
N نماذج + M أداة = N+M فقط (عبر بروتوكول موحّد)
```

---

## المبادئ الجوهرية

| المبدأ | الشرح |
|--------|-------|
| **معيار مفتوح** | بروتوكول اتصال مثل HTTP — أي server يتبعه يعمل مع أي client |
| **Server-side Logic** | كل منطق العمل يبقى في الـ server، النموذج يسأل فقط |
| **Composability** | عشرات الـ servers في نفس الجلسة في آنٍ واحد |
| **Security by Design** | الـ server هو الحارس — Claude لا يصل مباشرة للبيانات |

---

## الأطراف الثلاثة

```
┌─────────────────────────────────────────────────────┐
│                        HOST                         │
│                                                     │
│  ┌─────────────┐  JSON-RPC  ┌─────────────────┐    │
│  │             │ ◄────────► │                 │    │
│  │  MCP Client │            │   MCP Server    │    │
│  │  (Claude)   │            │                 │    │
│  │             │            │  ┌───────────┐  │    │
│  └─────────────┘            │  │ Database  │  │    │
│                             │  │ APIs      │  │    │
│                             │  │ Files     │  │    │
│                             │  └───────────┘  │    │
│                             └─────────────────┘    │
└─────────────────────────────────────────────────────┘
```

- **MCP Client** — النموذج الذي يطلب البيانات (مثل Claude)
- **MCP Server** — الخدمة التي توفّر البيانات أو تنفّذ الأوامر
- **Host** — البيئة التي تشغّل كل شيء (مثل Claude.ai أو Claude Code)

---

## مراحل الاتصال

### ١. Initialization — التهيئة

عند بدء الجلسة، الـ Client يتصل بالـ Server ويعلن نفسه:

```json
// Client → Server
{
  "jsonrpc": "2.0",
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "clientInfo": { "name": "claude-desktop", "version": "1.0" },
    "capabilities": { "sampling": {}, "roots": { "listChanged": true } }
  }
}

// Server → Client
{
  "jsonrpc": "2.0",
  "result": {
    "serverInfo": { "name": "github-mcp", "version": "1.2.0" },
    "capabilities": { "tools": {}, "resources": { "subscribe": true } }
  }
}
```

### ٢. Discovery — الاكتشاف

Claude يتعلم ما هو متاح من أدوات ومصادر:

```json
// Client → Server
{ "method": "tools/list" }

// Server → Client
{
  "result": {
    "tools": [
      { "name": "search_repos", "description": "يبحث في مستودعات GitHub" },
      { "name": "create_issue", "description": "ينشئ issue جديدة" },
      { "name": "get_pr_status", "description": "يسترجع حالة الـ Pull Request" }
    ]
  }
}
```

### ٣. Execution — التنفيذ

```json
// المستخدم يطلب: "ما حالة الـ PR الأخير؟"

// Claude → Server
{
  "method": "tools/call",
  "params": {
    "name": "get_pr_status",
    "arguments": { "repo": "facebook/react", "pr_number": 31542 }
  }
}

// Server → Claude
{
  "result": {
    "content": [{
      "type": "text",
      "text": "PR #31542 — مفتوح، 3 reviewers، جميع checks نجحت"
    }]
  }
}
```

### ٤. Context Integration — دمج السياق

نتيجة الـ tool تُضاف لـ context المحادثة، ويُكمل Claude إجابته بشكل طبيعي.

---

## المكوّنات الثلاثة

### 🔧 Tools — الأدوات

وظائف ينفذها الـ server بناءً على طلب النموذج.

**مثال — Shopify MCP Server:**

```json
{
  "name": "get_order_details",
  "description": "يسترجع تفاصيل طلب شراء بناءً على رقمه",
  "inputSchema": {
    "type": "object",
    "properties": {
      "order_id": {
        "type": "string",
        "description": "رقم الطلب"
      },
      "include_shipping": {
        "type": "boolean",
        "description": "هل تشمل بيانات الشحن؟",
        "default": true
      }
    },
    "required": ["order_id"]
  }
}
```

**كيف يُستخدم:**

```
المستخدم: "أين طلبي رقم #ORDER-8821؟"

Claude → tools/call: get_order_details({ order_id: "ORDER-8821" })
Server → Claude: { status: "قيد الشحن", eta: "غداً", carrier: "Aramex" }
Claude → المستخدم: "طلبك #ORDER-8821 في الطريق إليك، يصل غداً عبر Aramex."
```

---

### 📦 Resources — المصادر

بيانات ثابتة أو شبه ثابتة يقرأها النموذج كـ context مستمر.

**مثال — Slack MCP Server:**

```
// URI scheme للمصادر
"slack://channels/list"          → قائمة كل القنوات
"slack://channel/general"        → محتوى قناة #general
"slack://user/profile/U08XY"     → بيانات موظف معين
```

**الفرق بين Tools و Resources:**

|            | Tools                | Resources              |
|------------|----------------------|------------------------|
| الاستخدام | تنفيذ أوامر وكتابة  | قراءة فقط              |
| متى يُستدعى | عند طلب محدد       | في بداية السياق        |
| مثال       | إرسال رسالة Slack    | قائمة القنوات المتاحة |

---

### 💬 Prompts — القوالب

قوالب محادثة جاهزة يُعرّفها الـ server لحالات الاستخدام المتكررة.

**مثال — Jira MCP Server:**

```json
{
  "name": "sprint_review",
  "description": "يُنشئ تقرير مراجعة sprint",
  "arguments": [
    { "name": "sprint_id", "required": true },
    { "name": "team_name", "required": false },
    { "name": "include_velocity", "required": false }
  ]
}
```

---

## بروتوكول النقل

### stdio — للبيئات المحلية

الـ server يعمل على نفس الجهاز، التواصل عبر stdin/stdout.

```json
// claude_desktop_config.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/username/Documents"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_xxxxxxxxxxxx" }
    }
  }
}
```

**مثال استخدام — VS Code + GitHub Copilot:**

```
المطوّر: "راجع الـ PR هذا وأخبرني بأي مشاكل أمنية"
Claude (عبر GitHub MCP):
  → get_pr_diff({ pr: 142 })
  → get_file_content({ path: "auth/middleware.js" })
  → create_review_comment({ body: "SQL injection محتملة في السطر 47" })
```

---

### SSE / HTTP — للبيئات البعيدة

الـ server يعمل عبر الإنترنت، التواصل عبر HTTP مع Server-Sent Events.

```json
// في claude.ai أو تطبيقات الويب
{
  "type": "url",
  "url": "https://mcp.notion.com/mcp",
  "name": "notion"
}
```

**مثال استخدام — Notion كـ knowledge base:**

```
المستخدم: "أنشئ صفحة توثيق لميزة تسجيل الدخول الجديدة"
Claude (عبر Notion MCP):
  → search_pages({ query: "authentication template" })
  → create_page({
      parent: "Engineering Docs",
      title: "Login Feature — v2.1",
      content: "## الوصف\n## كيفية الاستخدام\n## API Reference"
    })
```

---

## نموذج الأمان

```
المستخدم
    ↓
Claude (Client) — يطلب tools محددة فقط
    ↓
MCP Server — الحارس
    ├── يتحقق من الهوية (Authentication)
    ├── يتحقق من الصلاحيات (Authorization)
    ├── يُفلتر البيانات الحساسة
    └── يُسجّل كل طلب (Audit Log)
    ↓
قاعدة البيانات / الـ API
```

### Capability Negotiation

كل طرف يُعلن ما يدعمه فقط — إذا كانت هناك ميزة غير مدعومة، الاتصال يفشل بأمان:

```json
// Client يُعلن قدراته
{
  "capabilities": {
    "sampling": {},
    "roots": { "listChanged": true }
  }
}

// Server يُعلن قدراته
{
  "capabilities": {
    "tools": {},
    "resources": { "subscribe": true },
    "prompts": {}
  }
}
```

---

## أمثلة عملية من شركات حقيقية

### مثال ١ — GitHub MCP

**السيناريو:** مطوّر يريد مراجعة الـ codebase وإنشاء issues تلقائياً.

```python
# GitHub MCP Server (Python)
from mcp.server.fastmcp import FastMCP
from github import Github

mcp = FastMCP("github-server")
gh = Github(os.environ["GITHUB_TOKEN"])

@mcp.tool()
def search_code(query: str, repo: str) -> dict:
    """يبحث في كود مستودع GitHub"""
    results = gh.search_code(f"{query} repo:{repo}")
    return {
        "count": results.totalCount,
        "files": [{"path": r.path, "url": r.html_url} for r in results[:10]]
    }

@mcp.tool()
def create_issue(repo: str, title: str, body: str, labels: list = []) -> dict:
    """ينشئ issue جديدة في مستودع"""
    repository = gh.get_repo(repo)
    issue = repository.create_issue(title=title, body=body, labels=labels)
    return {"number": issue.number, "url": issue.html_url}

@mcp.resource("github://repos/list")
def list_repos() -> str:
    """قائمة المستودعات المتاحة"""
    repos = gh.get_user().get_repos()
    return "\n".join([f"- {r.full_name}" for r in repos])
```

**محادثة مثال:**

```
المستخدم: "ابحث عن كل الأماكن التي نستخدم فيها fetch() بدون error handling
           في مستودع myorg/backend، وأنشئ issue لكل واحدة"

Claude:
  1. search_code("fetch(" repo:"myorg/backend") → وجد 12 ملف
  2. تحليل كل ملف
  3. create_issue("myorg/backend", "Missing error handling in api/users.js", ...)
  4. create_issue("myorg/backend", "Missing error handling in services/payment.js", ...)
  ...
  "أنشأت 8 issues للأماكن التي تفتقر لـ error handling"
```

---

### مثال ٢ — Stripe MCP

**السيناريو:** فريق دعم العملاء يريد الاستعلام عن الفواتير وإصدار استردادات.

```typescript
// Stripe MCP Server (TypeScript)
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import Stripe from "stripe";
import { z } from "zod";

const server = new McpServer({ name: "stripe-support", version: "1.0.0" });
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

server.tool(
  "get_customer",
  "يسترجع بيانات عميل بناءً على الإيميل",
  { email: z.string().email() },
  async ({ email }) => {
    const customers = await stripe.customers.list({ email, limit: 1 });
    if (!customers.data.length) {
      return { content: [{ type: "text", text: "العميل غير موجود" }] };
    }
    const customer = customers.data[0];
    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          id: customer.id,
          name: customer.name,
          created: new Date(customer.created * 1000).toLocaleDateString("ar"),
          balance: `${customer.balance / 100} USD`
        })
      }]
    };
  }
);

server.tool(
  "create_refund",
  "يُصدر استرداداً لعملية دفع",
  {
    payment_intent_id: z.string(),
    amount: z.number().optional(),
    reason: z.enum(["duplicate", "fraudulent", "requested_by_customer"])
  },
  async ({ payment_intent_id, amount, reason }) => {
    const refund = await stripe.refunds.create({
      payment_intent: payment_intent_id,
      amount: amount ? amount * 100 : undefined,
      reason
    });
    return {
      content: [{
        type: "text",
        text: `تم الاسترداد بنجاح — رقم: ${refund.id} — المبلغ: ${(refund.amount / 100)} USD`
      }]
    };
  }
);
```

**محادثة مثال:**

```
موظف دعم: "العميل john@example.com يقول إنه دُفع مرتين لنفس الطلب"

Claude:
  1. get_customer({ email: "john@example.com" })
     → { id: "cus_Xyz123", balance: "$0" }
  2. get_invoices({ customer_id: "cus_Xyz123", limit: 5 })
     → فاتورتان بتاريخ متقارب بنفس المبلغ $49.99
  3. create_refund({
       payment_intent_id: "pi_duplicate123",
       amount: 49.99,
       reason: "duplicate"
     })
     → "تم الاسترداد بنجاح — رقم: re_abc123"

"تم إصدار استرداد بمبلغ $49.99 للعميل john@example.com
 بسبب الدفع المزدوج. رقم الاسترداد: re_abc123"
```

---

### مثال ٣ — Salesforce MCP

**السيناريو:** فريق مبيعات يريد تحديث سجلات العملاء وإنشاء التقارير.

```python
# Salesforce MCP Server
from mcp.server.fastmcp import FastMCP
from simple_salesforce import Salesforce

mcp = FastMCP("salesforce-crm")
sf = Salesforce(
    username=os.environ["SF_USERNAME"],
    password=os.environ["SF_PASSWORD"],
    security_token=os.environ["SF_TOKEN"]
)

@mcp.tool()
def search_leads(company: str = None, industry: str = None) -> dict:
    """يبحث عن leads بناءً على معايير"""
    query = "SELECT Id, FirstName, LastName, Company, Email, Status FROM Lead WHERE "
    conditions = []
    if company:
        conditions.append(f"Company LIKE '%{company}%'")
    if industry:
        conditions.append(f"Industry = '{industry}'")
    query += " AND ".join(conditions) if conditions else "Status = 'Open'"
    results = sf.query(query)
    return {"total": results["totalSize"], "leads": results["records"]}

@mcp.tool()
def update_opportunity_stage(opportunity_id: str, stage: str, close_date: str = None) -> dict:
    """يُحدّث مرحلة الصفقة"""
    update_data = {"StageName": stage}
    if close_date:
        update_data["CloseDate"] = close_date
    sf.Opportunity.update(opportunity_id, update_data)
    return {"success": True, "opportunity_id": opportunity_id, "new_stage": stage}

@mcp.resource("salesforce://pipeline/summary")
def get_pipeline_summary() -> str:
    """ملخص pipeline المبيعات الحالي"""
    result = sf.query("""
        SELECT StageName, COUNT(Id) count, SUM(Amount) total
        FROM Opportunity
        WHERE IsClosed = false
        GROUP BY StageName
    """)
    return "\n".join([
        f"- {r['StageName']}: {r['count']} صفقة، إجمالي ${r['total']:,.0f}"
        for r in result["records"]
    ])
```

**محادثة مثال:**

```
مدير المبيعات: "كيف حال pipeline هذا الشهر؟ وأي leads من قطاع التكنولوجيا
               لم يُتواصل معهم بعد؟"

Claude:
  1. resources/read("salesforce://pipeline/summary")
     → Prospecting: 23 صفقة، $1.2M
       Negotiation: 8 صفقات، $890K
       Proposal: 15 صفقة، $2.1M
  2. search_leads({ industry: "Technology" })
     → 14 lead جديد لم يُتواصل معهم

"Pipeline الحالي: 46 صفقة بقيمة إجمالية $4.19M
 أبرز المراحل: Proposal ($2.1M) و Prospecting ($1.2M)

 14 lead من قطاع التكنولوجيا لم يُتواصل معهم بعد:
 • TechCorp Inc — مدير: Sarah Johnson
 • CloudSystems — مدير: Ahmed Al-Rashid
 ..."
```

---

### مثال ٤ — AWS MCP

**السيناريو:** مهندس DevOps يريد مراقبة البنية التحتية وإدارة الـ resources.

```python
# AWS MCP Server
import boto3
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("aws-devops")

@mcp.tool()
def get_ec2_status(region: str = "us-east-1", instance_ids: list = None) -> dict:
    """يسترجع حالة EC2 instances"""
    ec2 = boto3.client("ec2", region_name=region)
    filters = []
    if instance_ids:
        filters.append({"Name": "instance-id", "Values": instance_ids})
    response = ec2.describe_instances(Filters=filters)
    instances = []
    for reservation in response["Reservations"]:
        for inst in reservation["Instances"]:
            name = next((t["Value"] for t in inst.get("Tags", [])
                        if t["Key"] == "Name"), "بدون اسم")
            instances.append({
                "id": inst["InstanceId"],
                "name": name,
                "state": inst["State"]["Name"],
                "type": inst["InstanceType"],
                "public_ip": inst.get("PublicIpAddress", "لا يوجد")
            })
    return {"count": len(instances), "instances": instances}

@mcp.tool()
def get_cloudwatch_alarms(state: str = "ALARM") -> dict:
    """يسترجع CloudWatch alarms النشطة"""
    cw = boto3.client("cloudwatch")
    response = cw.describe_alarms(StateValue=state)
    alarms = [{
        "name": a["AlarmName"],
        "metric": a["MetricName"],
        "threshold": a["Threshold"],
        "state_reason": a["StateReason"]
    } for a in response["MetricAlarms"]]
    return {"active_alarms": len(alarms), "alarms": alarms}

@mcp.tool()
def scale_ecs_service(cluster: str, service: str, desired_count: int) -> dict:
    """يُعدّل عدد instances لـ ECS service"""
    ecs = boto3.client("ecs")
    ecs.update_service(
        cluster=cluster,
        service=service,
        desiredCount=desired_count
    )
    return {"success": True, "cluster": cluster, "service": service, "new_count": desired_count}
```

**محادثة مثال:**

```
مهندس DevOps: "في الـ production هل في أي مشاكل الحين؟
              وإذا في load عالي scale up الـ API service"

Claude:
  1. get_cloudwatch_alarms({ state: "ALARM" })
     → alarm واحد: "API-CPU-High" — CPU > 85% لآخر 15 دقيقة
  2. get_ec2_status({ region: "us-east-1" })
     → 3 instances تعمل، جميعها healthy
  3. scale_ecs_service("production", "api-service", 6)
     → نجح — من 3 إلى 6 instances

"تقرير الـ Production:
 ⚠️ تنبيه نشط: CPU عالي على خدمة الـ API (85%+) منذ 15 دقيقة
 ✅ تم scale up خدمة الـ API من 3 إلى 6 instances
 ✅ باقي الـ infrastructure: لا مشاكل"
```

---

### مثال ٥ — Figma MCP

**السيناريو:** مصمم يريد استخراج tokens التصميم وتحويلها لكود.

```typescript
// Figma MCP Server
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import axios from "axios";
import { z } from "zod";

const server = new McpServer({ name: "figma-design", version: "1.0.0" });
const FIGMA_TOKEN = process.env.FIGMA_ACCESS_TOKEN;

server.tool(
  "get_design_tokens",
  "يستخرج design tokens من ملف Figma",
  { file_key: z.string() },
  async ({ file_key }) => {
    const { data } = await axios.get(
      `https://api.figma.com/v1/files/${file_key}/variables/local`,
      { headers: { "X-Figma-Token": FIGMA_TOKEN } }
    );
    const tokens: Record<string, any> = {};
    for (const variable of Object.values(data.meta.variables) as any[]) {
      tokens[variable.name] = {
        type: variable.resolvedType,
        value: variable.valuesByMode
      };
    }
    return { content: [{ type: "text", text: JSON.stringify(tokens, null, 2) }] };
  }
);

server.tool(
  "get_component_props",
  "يسترجع props مكوّن Figma معين",
  { file_key: z.string(), node_id: z.string() },
  async ({ file_key, node_id }) => {
    const { data } = await axios.get(
      `https://api.figma.com/v1/files/${file_key}/nodes?ids=${node_id}`,
      { headers: { "X-Figma-Token": FIGMA_TOKEN } }
    );
    const node = data.nodes[node_id].document;
    return {
      content: [{
        type: "text",
        text: JSON.stringify({
          name: node.name,
          type: node.type,
          fills: node.fills,
          effects: node.effects,
          componentProperties: node.componentProperties
        }, null, 2)
      }]
    };
  }
);
```

**محادثة مثال:**

```
مطوّر Frontend: "حوّل design tokens من ملف Figma
               هذا إلى CSS variables و Tailwind config"

Claude:
  1. get_design_tokens({ file_key: "abc123xyz" })
     → colors, typography, spacing, shadows...
  2. يُولّد CSS:
     :root {
       --color-primary: #0066FF;
       --color-secondary: #6B7280;
       --font-heading: 'Inter', sans-serif;
       --spacing-sm: 8px;
       ...
     }
  3. يُولّد tailwind.config.js:
     module.exports = {
       theme: {
         extend: {
           colors: { primary: '#0066FF', secondary: '#6B7280' },
           spacing: { sm: '8px' }
         }
       }
     }
```

---

## بناء MCP Server — الدليل الكامل

### Python — باستخدام FastMCP

```python
from mcp.server.fastmcp import FastMCP
from typing import Optional
import httpx

# ١. إنشاء الـ server
mcp = FastMCP(
    name="my-service",
    version="1.0.0",
    description="وصف الـ server"
)

# ٢. تعريف Tool بسيط
@mcp.tool()
def get_weather(city: str, unit: str = "celsius") -> dict:
    """يسترجع الطقس لمدينة معينة"""
    return {"city": city, "temperature": 28, "unit": unit, "condition": "مشمس"}

# ٣. Tool مع معالجة أخطاء
@mcp.tool()
async def fetch_data(url: str, timeout: int = 30) -> dict:
    """يجلب بيانات من URL خارجي"""
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(url, timeout=timeout)
            response.raise_for_status()
            return {"status": "success", "data": response.json()}
    except httpx.TimeoutException:
        return {"status": "error", "message": "انتهت مهلة الطلب"}
    except Exception as e:
        return {"status": "error", "message": str(e)}

# ٤. Resource
@mcp.resource("config://settings")
def get_settings() -> str:
    """إعدادات النظام الحالية"""
    return """
    - البيئة: Production
    - المنطقة: us-east-1
    - النسخة: 2.1.0
    """

# ٥. Prompt template
@mcp.prompt()
def analysis_report(topic: str, depth: str = "standard") -> str:
    """قالب تقرير تحليلي"""
    return f"""حلّل الموضوع التالي بعمق {depth}:

الموضوع: {topic}

الرجاء تغطية:
1. الملخص التنفيذي
2. النقاط الرئيسية
3. التوصيات
"""

# ٦. تشغيل الـ server
if __name__ == "__main__":
    mcp.run(transport="stdio")  # أو transport="sse" للبيئات البعيدة
```

### TypeScript — باستخدام MCP SDK

```typescript
import { McpServer, ResourceTemplate } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "my-service",
  version: "1.0.0"
});

// Tool بسيط
server.tool(
  "calculate",
  "يُجري عمليات حسابية",
  {
    operation: z.enum(["add", "subtract", "multiply", "divide"]),
    a: z.number(),
    b: z.number()
  },
  async ({ operation, a, b }) => {
    const results: Record<string, number> = {
      add: a + b,
      subtract: a - b,
      multiply: a * b,
      divide: b !== 0 ? a / b : NaN
    };
    const result = results[operation];
    if (isNaN(result)) {
      return { content: [{ type: "text", text: "خطأ: لا يمكن القسمة على صفر" }], isError: true };
    }
    return { content: [{ type: "text", text: `النتيجة: ${result}` }] };
  }
);

// Resource مع URI template
server.resource(
  "data",
  new ResourceTemplate("data://{category}/{id}", { list: undefined }),
  async (uri, { category, id }) => ({
    contents: [{
      uri: uri.href,
      mimeType: "application/json",
      text: JSON.stringify({ category, id, data: "..." })
    }]
  })
);

// تشغيل
const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## استراتيجية التبني — ثلاثة مستويات

### المستوى الأول: Consumer — الاستهلاك

استخدم الـ servers الجاهزة من المجتمع:

```bash
# servers رسمية من Anthropic
npx @modelcontextprotocol/server-filesystem
npx @modelcontextprotocol/server-github
npx @modelcontextprotocol/server-google-drive
npx @modelcontextprotocol/server-slack
npx @modelcontextprotocol/server-postgres

# servers من المجتمع
npx @modelcontextprotocol/server-brave-search
npx mcp-server-stripe
npx mcp-server-aws
```

**متى تختار هذا المستوى:**
- يوجد server جاهز يلبي احتياجك
- لا تريد الوقت في بناء البنية التحتية
- تريد تجربة MCP بسرعة

---

### المستوى الثاني: Builder — البناء

ابنِ server خاص يُعرّض أنظمتك الداخلية:

```python
# مثال: server لنظام ERP داخلي
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("company-erp")

@mcp.tool()
def get_inventory(product_code: str = None, category: str = None) -> dict:
    """يسترجع بيانات المخزون"""
    return erp_api.get_inventory(product_code=product_code, category=category)

@mcp.tool()
def create_purchase_order(supplier_id: str, items: list, notes: str = "") -> dict:
    """يُنشئ أمر شراء"""
    return erp_api.create_po(supplier_id=supplier_id, items=items, notes=notes)
```

**متى تختار هذا المستوى:**
- بياناتك داخلية ولا يوجد server جاهز
- تريد التحكم الكامل في ما يُكشف لـ Claude
- لديك API داخلي تريد ربطه بالذكاء الاصطناعي

---

### المستوى الثالث: Integrator — الدمج

اجعل منتجك يتحدث مع نماذج الذكاء الاصطناعي عبر MCP:

```python
# مثال: منصة SaaS تُعرّض نفسها كـ MCP server
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-saas-platform")

@mcp.tool()
def get_analytics(metric: str, date_range: str, segment: str = "all") -> dict:
    """يسترجع تحليلات المنصة"""
    return analytics.query(metric=metric, range=date_range, segment=segment)
```

**متى تختار هذا المستوى:**
- تبني منتجاً SaaS وتريد دعم AI agents
- عملاؤك يستخدمون Claude أو أدوات AI
- تريد أن يكون منتجك جزءاً من نظام AI البيئي

---

## خارطة الطريق

```
الأسبوع ١–٢: تعلّم وتجريب
  ✓ استخدم servers جاهزة (filesystem, github)
  ✓ جرّب MCP في Claude Desktop
  ✓ افهم تدفق الرسائج

الأسبوع ٣–٤: أوّل server خاص
  → ابنِ server بسيط لـ API داخلي
  → اختبره محلياً مع Claude Code
  → أضف error handling و logging

الشهر الثاني: تعمّق
  → أضف Resources و Prompts
  → انشر الـ server على cloud
  → ابدأ دمجه في workflow اليومي

الشهر الثالث+: دمج في المنتجات
  → MCP server لكل منتج رئيسي
  → دعم AI agents الخارجية
  → مراقبة الأداء والأمان
```

---

## موارد ومراجع

| المصدر | الرابط |
|--------|--------|
| المواصفة الرسمية | modelcontextprotocol.io |
| MCP SDK (Python) | github.com/modelcontextprotocol/python-sdk |
| MCP SDK (TypeScript) | github.com/modelcontextprotocol/typescript-sdk |
| قائمة الـ servers الجاهزة | github.com/modelcontextprotocol/servers |
| وثائق Anthropic | docs.anthropic.com |

---

*آخر تحديث: يونيو 2026 — إعداد للأغراض التعليمية*
