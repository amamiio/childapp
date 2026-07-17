# Child App — Parental Control (پایه پروژه)

## به‌روزرسانی نسخه‌ها (جولای ۲۰۲۶)

نسخه‌های Gradle/Kotlin/AGP/Hilt/Room این پروژه به آخرین نسخه‌های پایدار و
سازگار با هم به‌روزرسانی شدند (AGP 8.13.0، Kotlin 2.2.20، Hilt 2.57.2، Room 2.8.0).
از Kotlin 2.0 به بعد، Compose Compiler یک پلاگین Gradle جداگانه شده
(`org.jetbrains.kotlin.plugin.compose`) که به پروژه اضافه شده است.


این پروژه اسکلت اولیه و حرفه‌ای اپلیکیشن Child است؛ مطابق درخواست، با Kotlin،
Clean Architecture، MVVM، Hilt، Retrofit/OkHttp، Room، WorkManager، Coroutines/Flow
و Jetpack Compose (Material 3).

## نکته‌ی مهم درباره‌ی «مخفی کردن آیکون»

آیکون از طریق یک `activity-alias` رسمی اندروید مخفی/آشکار می‌شود
(`IconVisibilityManager.setLauncherIconVisible`)، **نه** با ترفندهای غیررسمی.
این یعنی:

- برنامه همیشه از مسیر **تنظیمات ← برنامه‌ها** قابل مشاهده، توقف و حذف است.
- کاربر دستگاه (در لحظه‌ی پیکربندی، معمولاً همان کسی که گوشی را در دست دارد)
  با یک صفحه‌ی صریح (`IconVisibilityScreen`) از پنهان شدن آیکون مطلع و آن را
  تأیید می‌کند — این اتفاق پنهانی رخ نمی‌دهد.
- در هر زمان می‌توان از همان تنظیمات، برنامه را حذف کرد؛ Device Admin فقط یک
  گام تأیید اضافه می‌کند، نه مانع واقعی.

اگر تصمیم گرفتید حتی همین سطح از مخفی‌سازی را هم نخواهید، کافیست فراخوانی
`IconVisibilityManager.setLauncherIconVisible(..., visible = false)` را در
`ChildNavGraph.kt` حذف و مستقیماً به `DONE` هدایت کنید.

## ساختار پوشه‌ها

```
ui/            صفحات Compose (welcome, permissions, connect, iconvisibility, lock, done) + navigation
domain/        model + repository interfaces (لایه‌ی مستقل از پیاده‌سازی)
data/
  remote/      Retrofit ApiService + DTOها + AuthInterceptor
  local/       Room (AppDatabase, DAOها, entityها) + SessionStore (DataStore)
  repository/  پیاده‌سازی ریپازیتوری‌ها
  workers/     SyncWorker (WorkManager + Hilt)
services/      Foreground Service، NotificationListenerService،
               AccessibilityService، DeviceAdminReceiver، BootReceiver
di/            ماژول‌های Hilt (Network, Database, Repository)
utils/         Constants، PermissionUtils، IconVisibilityManager
```

## قبل از build گرفتن در Android Studio

1. پروژه را در Android Studio باز کنید تا Gradle Wrapper به‌صورت خودکار ساخته شود
   (یا `gradle wrapper` را دستی اجرا کنید).
2. آیکون واقعی برنامه را با ابزار **Image Asset** جایگزین
   `drawable/ic_launcher_placeholder.xml` کنید.
3. مقدار `Constants.BASE_URL` را به آدرس API واقعی سرور خودتان تغییر دهید.
4. اندپوینت‌های سرور (`ChildApiService`) باید مطابق API واقعی پیاده‌سازی/تطبیق
   داده شوند — در این مرحله فقط قرارداد (contract) کلاینت نوشته شده است.

## چیزهایی که در این مرحله پیاده‌سازی شده

- روند onboarding کامل: خوش‌آمدگویی → مجوزها → اتصال (کد یا QR) → تصمیم درباره‌ی آیکون → شروع سرویس.
- درخواست ترتیبی مجوزها با توضیح متنی برای هرکدام.
- اتصال Child به Parent صرفاً از طریق سرور (کد pairing → توکن).
- Foreground Service با نوتیفیکیشن دائمی و همیشه-قابل‌مشاهده (الزام خود اندروید).
- ذخیره‌ی آفلاین در Room برای موقعیت مکانی و اعلان‌ها، با پرچم `synced`.
- SyncWorker دوره‌ای (هر ۱۵ دقیقه) که داده‌های سینک‌نشده را ارسال و دستورات
  والد (مثل لیست قفل) را دریافت می‌کند.
- قفل برنامه‌ها با AccessibilityService (فقط رویداد تغییر پنجره؛ محتوای صفحه خوانده نمی‌شود).
- BOOT_COMPLETED فقط در صورت pair بودن دستگاه، سرویس را دوباره راه‌اندازی می‌کند.

## مراحل بعدی پیشنهادی (طبق درخواست شما)

- Geofence
- کنترل زمان استفاده (Screen Time budgets)
- داشبورد والد (اپ جداگانه‌ی Parent)
- اعلان‌های لحظه‌ای (Push / FCM)
- رمزنگاری اضافی برای داده‌های حساس ذخیره‌شده در Room (مثلاً SQLCipher)
