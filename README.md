# CODEX<>PRO

تطبيق معرفة ومستندات مبني بواجهة Vanilla JavaScript + Vite ويُغلّف كتطبيق Android باستخدام Capacitor.

## حالة المشروع بوضوح

الواجهة الموجودة حاليًا هي أساس المشروع، لكنها ما زالت تستخدم بيانات محلية تجريبية (`localStorage`) وتحتوي على تبديل أدوار للمعاينة. لم يكتمل بعد تسجيل الدخول الحقيقي أو ربط المستندات والتحديثات بقاعدة Supabase. لا تستخدمها لحفظ بيانات حساسة قبل إكمال مراحل المصادقة وRLS.

## المتطلبات الحالية

- Node.js 22 أو أحدث.
- Capacitor 8.
- Android SDK Platform 36 وBuild Tools 36.0.0.
- Java 21 في بيئة GitHub Actions.
- Android Studio 2025.2.1 أو أحدث للتطوير المحلي.

## التشغيل محليًا

1. انسخ `.env.example` إلى `.env.local`.
2. املأ `VITE_SUPABASE_URL` و`VITE_SUPABASE_ANON_KEY` بعد إنشاء مشروع Supabase.
3. ثبّت الاعتماديات وشغّل الواجهة:

   ```bash
   npm install
   npm run dev
   ```

## بناء Android محليًا

```bash
npm install
npm run build
npx cap add android   # نفّذها مرة واحدة فقط إذا لم يوجد مجلد android
npx cap sync android
npx cap open android
```

مجلد `android/` مستثنى من Git عمدًا لأن Workflow ينشئه تلقائيًا من Capacitor عند عدم وجوده. إذا أضفت تعديلات Native خاصة بك، لا تحذف المجلد ولا تستبدله تلقائيًا.

## بناء Debug APK باستخدام GitHub Actions

1. ارفع محتويات هذا المجلد إلى جذر المستودع، بحيث يكون `package.json` و`index.html` في الجذر.
2. افتح تبويب **Actions** في GitHub.
3. اختر **Build CODEX<>PRO Android APK** ثم **Run workflow**.
4. يجب أن تنتهي المهمة باللون الأخضر. فشل بناء APK أو عدم وجود الملف يجعل المهمة حمراء.
5. افتح نتيجة التشغيل ونزّل Artifact باسم `CODEX-PRO-Android-APK`.
6. نزّل Artifact المساعد `CODEX-PRO-NPM-Lockfile` وأضف ملف `package-lock.json` إلى جذر المستودع؛ بعد ذلك تصبح عمليات البناء التالية أكثر ثباتًا وقابلية للتكرار.

لا يستخدم Workflow npm cache أو Gradle cache. ولأن الأرشيف الأصلي لم يتضمن `package-lock.json`، ينشئ Workflow ملف القفل على Runner عند الحاجة ثم ينفذ `npm ci`، ويرفع ملف القفل كـArtifact مساعد.

## GitHub Secrets

أضف هذه القيم من **Settings → Secrets and variables → Actions → New repository secret**:

| Secret name | الغرض | مطلوب الآن؟ |
|---|---|---|
| `SUPABASE_URL` | عنوان مشروع Supabase، يُمرر إلى Vite وقت البناء | عند ربط Supabase |
| `SUPABASE_ANON_KEY` | مفتاح العميل العام (anon/publishable) | عند ربط Supabase |
| `ANDROID_KEYSTORE_BASE64` | ملف keystore مشفّر Base64 لتوقيع إصدار Release | اختياري، لاحقًا |
| `ANDROID_KEYSTORE_PASSWORD` | كلمة مرور keystore | اختياري، لاحقًا |
| `ANDROID_KEY_ALIAS` | اسم المفتاح داخل keystore | اختياري، لاحقًا |
| `ANDROID_KEY_PASSWORD` | كلمة مرور المفتاح | اختياري، لاحقًا |

لا تضع أي قيمة سرية في ملفات المشروع. مفتاح Supabase anon/publishable مخصص للعميل وسيظهر ضمن APK بعد البناء؛ أمان البيانات يجب أن يعتمد على سياسات RLS. يمنع منعًا باتًا وضع أي مفتاح ذي صلاحيات خادم داخل الواجهة أو Workflow أو APK.

## المراحل الوظيفية التالية

- Supabase schema نهائي مع RLS واختبارات السماح والمنع.
- Bootstrap للمالك مع قيد وجود Owner واحد فقط.
- تسجيل ودخول وخروج واستمرار الجلسة والحظر.
- جلب المستندات والكتل والتحديثات من Supabase بدل بيانات Local Storage.
- رفع الملفات بقيود الحجم والنوع والتحقق من المحتوى.
- لوحات منفصلة للمستخدم والمدير والمالك، مع فرض الصلاحيات في قاعدة البيانات.

لا يُعد التطبيق جاهزًا للاستخدام الحقيقي أو الإنتاجي حتى تنجح اختبارات الأمان والوظائف السابقة على Android.
