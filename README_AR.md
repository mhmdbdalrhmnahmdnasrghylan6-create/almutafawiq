# المتفوق – المرحلة الثالثة (طبقة Firebase)

هذه المرحلة تضيف طبقة الربط الأساسية إلى مشروع Flutter: Firebase Authentication، وخدمات قراءة المواد والدروس، وقواعد Firestore، ووظيفة خادمية لتفعيل أكواد الاشتراك.

## 1) ما الذي أُضيف؟
- `lib/services/auth_service.dart`: إنشاء حساب طالب بالبريد وكلمة المرور، إنشاء ملف الطالب، تسجيل الدخول والخروج.
- `lib/services/content_service.dart`: بث المواد والدروس من Firestore وطلب تفعيل الاشتراك من Cloud Function.
- `firestore.rules`: قواعد أولية تمنع الطالب من تعديل دوره أو الاشتراكات أو أكواد التفعيل.
- `functions/src/index.ts`: تفعيل الكود مرة واحدة داخل معاملة خادمية، مع تخزين SHA-256 للكود بدل النص الأصلي.
- `SEED_DATA.json`: أسماء المواد الست وبنية نموذج الدرس.

## 2) متطلبات الربط (تحتاج تنفيذها في حساب Firebase الخاص بك)
1. أنشئ مشروعاً في Firebase Console.
2. أضف تطبيق Android باستخدام اسم الحزمة الموجود في مشروع Flutter (تحقق منه في `android/app/build.gradle` بعد إنشاء مجلدات Android إن لم تكن موجودة).
3. فعّل Authentication > Email/Password.
4. أنشئ قاعدة Cloud Firestore.
5. ثبّت FlutterFire CLI ثم شغّل من مجلد المشروع:
   ```bash
   dart pub global activate flutterfire_cli
   flutterfire configure
   flutter pub get
   ```
   سينشئ الأمر `lib/firebase_options.dart` بحسب مشروعك. لا تضع مفاتيح أو ملفات Firebase الخاصة بك في محادثة عامة.
6. أضف تهيئة Firebase إلى `main()` قبل `runApp`:
   ```dart
   await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
   ```
   مع استيراد `firebase_core.dart` و`firebase_options.dart` وجعل `main` دالة `async` واستدعاء `WidgetsFlutterBinding.ensureInitialized()`.
7. انشر القواعد والوظيفة:
   ```bash
   firebase login
   firebase use --add
   firebase deploy --only firestore:rules,firestore:indexes,functions
   ```

## 3) بنية Firestore المقترحة
- `users/{uid}`: الاسم، البريد، الدور `student`، الصف، المنهج.
- `subjects/{subjectId}`: العنوان، الترتيب، `active`.
- `subjects/{subjectId}/lessons/{lessonId}`: العنوان، الملخص، المحتوى، رابط الفيديو، `isFree`، `published`، `order`.
- `subscriptions/{uid}`: الحالة وتاريخ الانتهاء. الكتابة ممنوعة من التطبيق وتتم عبر الخادم فقط.
- `subscriptionCodes/{sha256}`: `durationDays` و`usedAt` و`usedBy` و`disabled`. لا تضع الكود الخام في Firestore.

## 4) إنشاء كود اشتراك
أنشئ الأكواد من بيئة إدارية موثوقة خارج تطبيق الطالب. احسب SHA-256 للكود بعد تحويله إلى أحرف كبيرة وإزالة المسافات، ثم أنشئ مستنداً في `subscriptionCodes` بمعرّف يساوي قيمة الهاش، وحقول مثل:
```json
{"durationDays":30,"disabled":false}
```
لا تفتح صلاحية قراءة أو كتابة هذا المسار للطلاب. لا تضع أداة توليد الأكواد أو صلاحيات الإدارة داخل تطبيق الطالب.

## 5) تنبيهات مهمة قبل التشغيل
- هذه طبقة تكامل أولية؛ واجهة تسجيل الدخول في المرحلة الثانية ما زالت واجهة تجريبية ولم يتم استبدالها أو ربطها تلقائياً بهذه الخدمات بعد.
- يجب تهيئة Firebase وإنشاء `firebase_options.dart` على جهازك؛ لا يمكن إنشاء ملف الإعداد الحقيقي دون بيانات مشروعك.
- قواعد Firestore تحمي قراءة مستندات الدروس، لكن روابط ملفات الفيديو/الـPDF الخارجية تحتاج حماية مستقلة (مثل Firebase Storage Rules أو روابط موقعة قصيرة العمر). لا تجعل ملفات مدفوعة متاحة عبر روابط عامة.
- اجعل حساب الإدارة الوحيد موثوقاً، واضبط `role: admin` يدوياً من Firebase Console بعد التحقق من هويته. لا تسمح للطالب بتغيير دوره.
- استخدم كتباً وملخصات وفيديوهات تملك حقوق نشرها أو لديك إذن باستخدامها.

## 6) التشغيل بعد إعداد Android وFirebase
```bash
flutter pub get
flutter run
```
هذه الحزمة ليست APK جاهزاً للنشر. إصدار APK يأتي بعد ربط Firebase واختبار الحسابات والصلاحيات والاشتراكات على جهاز فعلي.
