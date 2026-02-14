<div dir="rtl">

# 📊 ماژول مانیتورینگ Go + Prometheus

این ماژول یک سرویس ساده Go را با استک مانیتورینگ کامل اجرا می‌کند تا بتوانید متریک‌های اپلیکیشن و سیستم را جمع‌آوری، مشاهده و تحلیل کنید.

## 🎯 هدف ماژول

- ارائه یک API ساده در `main.go`
- اکسپورت متریک‌ها در مسیر `/metrics`
- جمع‌آوری متریک‌ها با Prometheus
- آماده‌سازی زیرساخت نمایش و هشدار با Grafana و Alertmanager

## 🧱 اجزای پروژه

- `main.go`: سرویس Go + تعریف و ثبت متریک‌ها
- `prometheus/prometheus.yml`: تنظیمات scrape و alerting
- `alertmanager/alertmanager.yml`: مسیر دریافت alert
- `docker-compose.yml`: بالا آوردن سرویس‌های مانیتورینگ

## ⚙️ پیش‌نیازها

- Go `1.25+`
- Docker + Docker Compose

## 🚀 اجرای سریع

1. اجرای سرویس Go:

```bash
go run main.go
```

2. اجرای سرویس‌های مانیتورینگ:

```bash
docker compose up -d
```

3. دسترسی به سرویس‌ها:

- اپلیکیشن Go: `http://localhost:8080/`
- متریک اپلیکیشن: `http://localhost:8080/metrics`
- Prometheus: `http://localhost:9090`
- Alertmanager: `http://localhost:9093`
- Grafana: `http://localhost:3000` (user: `admin`, pass: `super_secure_password`)

## 🧪 متریک‌های فعلی در `main.go`

در فایل `main.go` دو متریک اصلی تعریف شده‌اند:

1. `http_requests_total` (نوع `CounterVec`)
- شمارش کل درخواست‌های HTTP
- برچسب‌ها: `method`, `endpoint`, `status`

2. `http_request_duration_seconds` (نوع `HistogramVec`)
- اندازه‌گیری زمان پاسخ‌گویی درخواست‌ها
- برچسب‌ها: `method`, `endpoint`

این متریک‌ها در `metricsMiddleware` مقداردهی می‌شوند و در تابع `main` با `prometheus.MustRegister(...)` ثبت شده‌اند.

## ➕ Add Custom Metric (افزودن متریک سفارشی)

برای افزودن متریک سفارشی در `main.go` این الگو را دنبال کنید:

1. تعریف متریک در بخش Metrics
2. ثبت متریک داخل `main`
3. به‌روزرسانی متریک در Handler یا Middleware
4. بررسی خروجی در `/metrics`

### نمونه عملی: شمارش درخواست‌های endpoint اصلی

این نمونه یک متریک جدید برای شمارش فراخوانی‌های هندلر `/` اضافه می‌کند:

<div dir="ltr">

```go
var helloRequestsTotal = prometheus.NewCounter(
	prometheus.CounterOpts{
		Name: "hello_requests_total",
		Help: "Total requests to hello handler",
	},
)
```
<div dir="rtl">

ثبت در `main`:

<div dir="ltr">

```go
prometheus.MustRegister(helloRequestsTotal)
```
<div dir="rtl">

استفاده در `helloHandler`:

<div dir="ltr">

```go
func helloHandler(w http.ResponseWriter, r *http.Request) {
	helloRequestsTotal.Inc()
	time.Sleep(100 * time.Millisecond)
	w.Write([]byte("Hello Prometheus"))
}
```
<div dir="rtl">

پس از اجرا، مقدار این متریک را در آدرس زیر می‌بینید:

`http://localhost:8080/metrics`

## 🧠 نکات حرفه‌ای برای Custom Metric

- تعداد labelها را کنترل کنید (Cardinality بالا باعث مصرف زیاد حافظه می‌شود).
- از نام‌گذاری استاندارد استفاده کنید:
- Counter: پسوند `_total`
- Histogram: پسوند `_seconds`
- برای هر متریک `Help` واضح و کوتاه بنویسید.
- اگر متریک باید کاهش و افزایش داشته باشد، از `Gauge` استفاده کنید.

## 🔍 عیب‌یابی

- اگر در Prometheus وضعیت `go_app` خطا بود:
- ابتدا `go run main.go` را اجرا کنید.
- مقدار target در `prometheus.yml` روی `172.17.0.1:8080` تنظیم شده است؛ در صورت تفاوت شبکه Docker، IP را اصلاح کنید.
- برای بررسی IP بریج Docker می‌توانید از `ip addr show docker0` استفاده کنید.

## ✅ جمع‌بندی

این ماژول یک نقطه شروع استاندارد برای Observability در Go است و ساختار آن طوری چیده شده که بتوانید متریک‌های سفارشی را سریع، امن و قابل نگهداری به `main.go` اضافه کنید.
