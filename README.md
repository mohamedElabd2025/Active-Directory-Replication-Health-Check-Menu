تمام… بما إن الجهاز دخل مرحلة “شاهد القبر” في Active Directory 
يبقى خلينا نعمله “عملية قلب مفتوح” ونشوف إزاي بننقذ الموقف أو نتعامل معاه بشكل صح.
احنا عرفنا ان قبل 60 يوم المشكل قليله 
 و من 60 ل 180 مرحلة الخطر و من 180 ده مرحلة الكوارث 
تعالي بقا نحلل الكلام ده ممن يكون في مشكلة عندك في سيرفير شهر او اتنين  و تكون محتاج السيرفير و ممكن يكون اطول شويا و توصل ل 3 او 4 بس استحالة تكون سايب سيرفير اكتر من 180 يوم مقفول و تكون محتاجة يعني تقريبا 6 شهور 
طيب نعمل اية المرحلة الاولي   ننضف الحرج خالص استنب بس مش بتكلم في الطب و الله في ال it   يعني نعمل اية  ندحل نشوف ال 
 “التنضيف الصح” (Forest Cleanup Strategy)
أنت قلت "ندخل ننضف الحرج" — خلينا نحولها لخطة IT حقيقية:
1. تحديد الأجهزة الميتة (Stale Computers)
أول خطوة:
•	نحدد آخر Logon لكل جهاز 
•	نشوف Last Logon Timestamp / LastLogonDate
طيب اجيب المعلومة اذاي 
Get-ADComputer -Filter * -Properties LastLogonDate |
Where-Object {$_.LastLogonDate -lt (Get-Date).AddDays(-60)} |
Select Name, LastLogonDate

تصنيف الأجهزة (مهم جدًا قبل الحذف) ماينفعش تمسح مباشرة 
اعمل 3 مجموعات:
•	أجهزة 0–60 يوم → Monitor 
•	أجهزة 60–120 → Disable 
•	أجهزة 120–180 → Quarantine 
•	أجهزة +180 → Candidate for deletion

3. Disable قبل Delete (أفضل Practice)
Disable-ADAccount -Identity "PC-Name"
ليه؟
•	عشان تتأكد إنه مش مستخدم فعليًا 
•	وتدي فرصة rollback

4. تنظيف Metadata لو الجهاز/DC قديم
لو عندك DC أو سيرفر قديم:
هنا بنستخدم:
•	ntdsutil 
•	أو أدوات زي: 
o	Active Directory Sites and Services 
o	repadmin

طيب انت بتعمل كل ده في الاول ليه بص ده امان ليك قبل اي حاجة انت مفيش حاجة مسحتها بس كمان مش انت بس اللي في القسم 

المرحلة ده خلصت انت عندك list  بتقولك كل المعلومات و دلواقتي انت محتاج تعرف و تتاكد من كام حاجة تانية الجهاز اللي في العمليات ده 
الوقت و التاريخ تمام  طيب ال ip  مظبوط طيب هو شايف ال domain طيب هو الفترة بتعتو قد اية 60 و لا اقل من 180 
كده معلومات كاملة تعالي بقا ندخل على كور العملية 
بعد ما تجمع البيانات دي… تعمل إيه؟
هنا بقى “Decision Engine” 
حالة 1: جهاز حي لكن ناسي يدخل
→ Monitor فقط
حالة 2: الجهاز مش بيرد لكن أقل من 120 يوم
→ Disable + Tag
حالة 3: 120–180 + مش بيرد + Secure channel broken
→ Quarantine OU
حالة 4: +180 يوم + ميت بكل المقاييس
→ Candidate Delete

🔹 KCC (Knowledge Consistency Checker)
خدمة داخل Active Directory مسؤولة عن إنشاء وإدارة مسارات النسخ المتماثل (Replication Topology) بين الـ Domain Controllers تلقائيًا.
وظيفته:
•	يحدد أي Domain Controller يتبادل البيانات مع الآخر. 
•	ينشئ Connection Objects تلقائيًا. 
•	يضمن وصول تحديثات AD إلى جميع الـ DCs بأقل تكلفة وأعلى كفاءة. 
مثال:
إذا كان لديك 10 Domain Controllers موزعين على عدة فروع، فإن KCC يبني شبكة Replication مناسبة دون الحاجة لتكوين يدوي.
________________________________________
🔹 ISTG (Inter-Site Topology Generator)
هو دور خاص داخل كل Site في Active Directory.
وظيفته:
•	اختيار وإنشاء اتصالات Replication بين الـ Sites المختلفة. 
•	التعاون مع KCC لبناء Replication بين المواقع الجغرافية. 
الفرق بين KCC و ISTG:
KCC	ISTG
يدير Replication داخل الموقع (Intra-Site)	يدير Replication بين المواقع (Inter-Site)
موجود على كل DC	يوجد DC واحد فقط في كل Site يحمل هذا الدور
مثال:
لديك:
•	Riyadh Site 
•	Jeddah Site 
•	Dammam Site 
يقوم ISTG بتحديد المسارات بين هذه المواقع لتبادل بيانات Active Directory.
________________________________________
🔹 Site Links
هي تعريفات في Active Directory تمثل الروابط الشبكية بين الـ Sites.
تحتوي على:
•	Cost (تكلفة الرابط) 
•	Replication Schedule (جدول التكرار) 
•	Replication Frequency (عدد مرات التكرار) 
الهدف:
إخبار Active Directory بكيفية الوصول بين المواقع المختلفة.
مثال:
Riyadh ↔ Jeddah
Cost = 100

Riyadh ↔ Dammam
Cost = 50
سيُفضّل AD الرابط الأقل تكلفة.
أوامر مفيدة:
من:
Active Directory Sites and Services
يمكنك إنشاء وتعديل Site Links.
________________________________________
🔹 SPN (Service Principal Name) & Kerberos
أولاً: Kerberos
بروتوكول المصادقة الأساسي في Active Directory.
المكونات:
•	Client 
•	Service 
•	KDC (Key Distribution Center) 
والـ KDC موجود داخل الـ Domain Controller.
طريقة العمل:
1.	المستخدم يسجل الدخول. 
2.	يحصل على TGT (Ticket Granting Ticket). 
3.	يطلب Ticket للخدمة المطلوبة. 
4.	يصل للخدمة دون إرسال كلمة المرور. 
المميزات:
•	أسرع. 
•	أكثر أمانًا. 
•	يدعم Single Sign-On (SSO). 
________________________________________
ثانياً: SPN
اختصار:
Service Principal Name
هو اسم فريد يربط خدمة معينة بحساب في Active Directory.
أمثلة:
MSSQLSvc/sql01.company.com:1433

HTTP/web01.company.com

CIFS/fileserver.company.com
لماذا مهم؟
عندما يريد Kerberos الوصول إلى خدمة:
•	يبحث عن SPN الخاص بها. 
•	يصدر Ticket للخدمة الصحيحة. 
مشاكل شائعة:
Duplicate SPN
وجود نفس SPN على حسابين مختلفين.
يسبب:
•	Kerberos Authentication Failure 
•	أخطاء SQL Server 
•	مشاكل IIS 
أوامر مهمة:
عرض SPN:
setspn -L accountname
البحث عن SPN مكرر:
setspn -X
إضافة SPN:
setspn -S HTTP/web01 accountname
________________________________________
🔹 Secure Channel
هو العلاقة الآمنة بين جهاز الدومين وDomain Controller.
عندما يتم ضم جهاز إلى الدومين:
•	يُنشأ Computer Account في AD. 
•	تُنشأ كلمة مرور سرية بين الجهاز والـ DC. 
•	تُستخدم لإنشاء Secure Channel. 
استخداماته:
•	تسجيل دخول المستخدمين. 
•	تطبيق Group Policy. 
•	Kerberos Authentication. 
•	الوصول لموارد الدومين. 
________________________________________
مشكلة مشهورة
رسالة:
The trust relationship between this workstation and the primary domain failed
تعني غالبًا أن Secure Channel مكسور.
الأسباب:
•	استعادة Snapshot قديم. 
•	عدم تزامن كلمة مرور الكمبيوتر. 
•	مشاكل Replication. 
•	إعادة بناء DC. 
________________________________________
فحص Secure Channel
PowerShell:
Test-ComputerSecureChannel
إصلاحه:
Test-ComputerSecureChannel -Repair
أو:
Reset-ComputerMachinePassword
________________________________________
الخلاصة السريعة
المصطلح	الوظيفة
KCC	إنشاء Topology للـ Replication تلقائيًا
ISTG	إدارة Replication بين الـ Sites
Site Links	تحديد المسارات والتكلفة بين المواقع
Kerberos	بروتوكول المصادقة الرئيسي في AD
SPN	تعريف الخدمة التي يستخدمها Kerberos
Secure Channel	القناة الآمنة بين الجهاز وDomain Controller
تسلسل منطقي للحفظ:
Site Links → ISTG → KCC → Replication
ثم
SPN → Kerberos Authentication
ثم
Secure Channel → Trust بين الجهاز والدومين.


