# استخدامات متقدمة لدوال العرض والتوجيه

<img src='../images/views.png'/>

## مُزخرِفات العرض (View decorators)

مزخرفات بايثون هي دوال تُستخدم لتغيير هيئة (وظيفة أو عمل) دوال أخرى. عندما يتم تنفيذ الدالة المُزخرَفِة، يُنَفَذ المُزخرِف بدلاً منها. يتمكن المُزخرِف من تنفيذ عمليات، أو تعديل المُعطيات (arguments)، أو إيقاف التنفيذ، أو استدعاء الدالة الأصلية (المُزخرَفِة). يمكننا استخدام المُزخرِفات للإحاطة بدوال العرض (الدوال التي تُستخدم لعرض المحتوى) بشيفرة برمجية نود تشغيلها قبل تنفيذ دوال العرض هذه.

```python
@decorator_function
def decorated():
    pass
```

إذا قمت باتباع دورة فلاسك الرسميّة فقد تبدو لك صيغة الشيفرة السابقة مألوفة. يُستخدم المُزخرِف `app.route@` لتحديد عناوين دوال العرض في تطبيقات فلاسك.

دعونا نأخذ نظرة على بعض المُزخرِفات الأخرى التي يمكننا استخدامها في تطبيقات فلاسك.

### المصادقة

تجعل إضافة Flask-Login من السهل إضافة نظام تسجيل إلى تطبيقك. بالإضافة إلى أنها تُمكِنُك من التعامل مع تفاصيل مصادقة المستخدمين. توفر لنا هذه الإضافة مُزخرِف يقوم بحصر وصول المستخدمين المسجلين فقط إلى دوال عرض معينة وهو المُزخرِف `login_required@`.

```python
# app.py

from flask import render_template
from flask_login import login_required, current_user


@app.route('/')
def index():
    return render_template("index.html")

@app.route('/dashboard')
@login_required
def account():
    return render_template("account.html")


```

<blockquote>
<b>تحذير</b><br/>
يجب وضع المُزخرِف <code>app.route@</code> في مقدمة المُزخرِفات دائماً (أول مُزخرِف عند استدعاء المُزخرِفات).
</blockquote>

فقط المستخدمين المسجلين (المصادقين) سيتمكنون من الولوج إلى الرابط *dashboard/*. يمكننا إعداد الإضافة لتوجه المستخدمين الغير مسجلين إلى صفحة التسجيل، أو لإرجاع رمز الحالة 401، أو للقيام بأي شيء آخر نريده.

<blockquote>
<b>ملاحظة</b><br/>
اقرأ المزيد حول استخدام إضافة Flask-Login في <a href='http://flask-login.readthedocs.org/en/latest/'>التوثيقات الرسمية</a> لها.
</blockquote>

### التخزين المؤقت

تخيّل أنَّ مقالة ما ذكرت تطبيقاً لك ونُشرِت على موقع CNN وبعض المواقع الإخبارية الأخرى. ستحصل على آلاف الزيارات في كل ثانية. وبما أن الصفحة الرئيسية في الموقع تقوم بعدة استعلامات في قاعدة البيانات في كل زيارة، سيؤدي هذا إلى إبطاء الموقع. فكيف يمكننا تسريع الأشياء عندها حتى لا يترك كل هؤلاء الزوار موقعنا؟

يوجد العديد من الإجابات الجيدة، ولكن هذا القسم هو حول التخزين المؤقت، لذا دعنا نتحدث عنه. سنستخدم إضافة <a href='http://pythonhosted.org/Flask-Cache/'>Flask-Cache</a> للقيام بذلك. توفر لنا هذه الإضافة مُزخرِف يُمكن استخدامه على صفحتنا الرئيسية لتخزين الاستجابة لفترة من الزمن.

يُمكن أن يتم إعداد هذه الإضافة مع باقة من قواعد بيانات التخزين المؤقت المختلفة. الخيار الشعبي هو <a href='http://redis.io/'>Redis</a>، والذي هو سهل الاستخدام والتثبيت. لنفترض أن إضافة Flask-Cache تم إعدادها بالفعل مع قاعدة البيانات المُختارة. تُظهِر التعليمات البرمجية أدناه كيف ستبدو دالة العرض المُزخرَفِة خاصتنا.

```python
# app.py

from flask_cache import Cache
from flask import Flask

app = Flask()

# سنقوم هنا بإضافة الإعدادات التي نريدها خلال الاستدعاء
cache = Cache(app)

@app.route('/')
@cache.cached(timeout=60)
def index():
    [...] # هنا يتم القيام ببعض عمليات الاستعلام التي نحتاجها
    return render_template(
        'index.html',
        latest_posts=latest_posts,
        recent_users=recent_users,
        recent_photos=recent_photos
    )
```

الآن ستعمل الدالة فقط كل ستين ثانية (عندما تنتهي مدة التخزين). سيتم حفظ الرد في ذاكرة التخزين المؤقت وعرضه عندما تطرأ أية طلبات (زيارات للصفحة).

<blockquote>
<b>ملاحظة</b><br/>
توفر لنا هذه الإضافة أيضاً خاصية لحفظ الدوال، أو بمعنى آخر، تخزين نتيجة دالة يتم استدعاءها بمعاملات محددة. يمكنك حتى تخزين قوالب جينجا.
</blockquote>

## مُزخرِفات مُخصصة

دعنا نتخيل أنك تملك تطبيق يفرض رسوماً على المستخدمين كل شهر. إذا أنتهت صلاحية حساب المستخدم سنقوم بتوجيهه إلى صفحة الدفع ونعلمه أنه عليك ترقية حسابه.

```python
# myapp/util.py

from functools import wraps
from datetime import datetime

from flask import flash, redirect, url_for

from flask_login import current_user

def check_expired(func):
    @wraps(func)
    def decorated_function(*args, **kwargs):
        if datetime.utcnow() > current_user.account_expires:
            flash("Your account has expired. Update your billing info.")
            return redirect(url_for('account_billing'))
        return func(*args, **kwargs)

    return decorated_function
```

| السطر | الشرح |
| ----- | ----- |
| 10 | عندما يتم زخرفة دالة باستخدام `check_expired@`، سيتم استدعاء `()check_expired` وستمرر الدالة المُزخرَفِة كمعامل. |
| 11 | يوجد في هذا السطر مُزخرِف (`warp@`) يقوم ببعض أعمال المراقبة حتى تبدو الدالة `()decorated_function` كالدالة `()func` ﻷغراض توثيقية وتنقيحية. هذا يجعل من سلوك الدالة أكثر طبيعيةً بقليل. |
| 12 | ستقوم الدالة `decorated_function` بجمع كل المعاملات المفتاحية (kwargs) والمعاملات الصفيّة (args) المُمررة لدالة العرض الأصليّة `()func`. في هذا السطر سنقوم بالتأكد إذا كان حساب المستخدم مُنتهي الصلاحية. إذا كان كذلك، سنرسل رسالة له ونقوم بإعادة توجيهه إلى صفحة الدفع. |

عندما نرص (نُكدِس) المُزخرِفات، المُزخرِف الأعلى (الذي في المقدمة) هو من يعمل أولاً، ومن ثم يقوم باستدعاء الذي يليه بالسطر، سواءً كان دالة عرض أو مُزخرِف آخر. الصيغة الكتابية للمُزخرِفات تحوي القليل من التجميل اللغوي (اقرأ المزيد عن <a href='https://ar.wikipedia.org/wiki/التجميل_اللغوي'>التجميل اللغوي</a>).

```python
# هذه التعليمات البرمجية:
@foo
@bar
def one():
    pass

r1 = one()

# هل هذه نفس التعليمة:
def two():
    pass

two = foo(bar(two))
r2 = two()

r1 == r2 # النتيجة ستكون صحيحة منطقياً (True)
```

الشيفرة أدناه تعرض مثال حول استخدام المُزخرِف المخصص الذي قمنا بإنشائه مع المُزخرِف `login_required@` من الإضافة Flask-Login. نستطيع استخدام كلا المُزخرِفين عن طريق رصهم (تكديسهم) أسفل بعضهم.

```python
# myapp/views.py

from flask import render_template

from flask_login import login_required

from . import app
from .util import check_expired

@app.route('/use_app')
@login_required
@check_expired
def use_app():
    """Use our amazing app."""
    # [...]
    return render_template('use_app.html')

@app.route('/account/billing')
@login_required
def account_billing():
    """Update your billing info."""
    # [...]
    return render_template('account/billing.html')
```

عندما يحاول المستخدم اﻵن الدخول إلى *use_app/*، سيتأكد المُزخرِف `()check_expired` قبل تشغيل دالة العرض أن حساب هذا المستخدم غير منتهي الصلاحية.

<blockquote>
<b>ملاحظة</b><br/>
اقرأ المزيد حول ما تقوم به الدالة <code>()warps</code> <a href='http://docs.python.org/2/library/functools.html#functools.wraps'>في توثيقات بايثون</a>.
</blockquote>

## محولات الروابط

### المحولات القياسية (المبنية في الإطار)

عندما تقوم بتعريف موجه بفلاسك، يمكنك تحديد أجزاء من الرابط ليتم تحويلها إلى متغيرات وتمريرها كمعاملات إلى دالة العرض.

```python
@app.route('/user/<username>')
def profile(username):
    pass
```

أي شيء سيتم كتابته مكان الجزء في العنوان المسمى ب `<username>` سيتم تمريره إلى دالة العرض كمعامل يسمى `username`. يمكنك أيضاً تحديد محول لتصفية (تحويل قيمة) المتغير قبل تمريره إلى دالة العرض.

```python
@app.route('/user/id/<int:user_id>')
def profile(user_id):
    pass
```

في الشيفرة أعلاه، الدخول إلى العنوان *http://myapp.com/user/id/Q29kZUxlc3NvbiEh* سيعيد رمز الحالة 404 (أي الصفحة غير موجودة). هذا لأن الجزء الأخير من الرابط الذي كان من المفترض أن يكون رقماً صحيحاً تم إدخال مكانه سلسلة نصية.

يمكننا أيضاً تعريف دالة عرض أخرى تستخدم مُنقِح يبحث عن سلسلة نصيّة.

يظهر الجدول أدناه محولات الروابط القياسية الموجودة في إطار فلاسك.

| المحول | الشرح |
| ------ | ----- |
| string | يقبل أي مُدخل نصي بدون مائلة (المحول الافتراضي). |
| int | يقبل الأعداد الصحيحة. |
| float | مثل المحول int ولكنه يقبل القيم ذات الفاصلة العشرية. |
| path | مثل المحول string ولكنه يقبل المائلة. |

### محولات مخصصة

يمكننا أيضاً صنع محولات خاصة لتناسب احتياجاتنا. على موقع ريديت (موقع مشهور لمشاركة الروابط) يصنع المستخدمون والمدراء مجتمعات للمناقشة ومشاركة الروابط بناءً على الموضوع. بعض الأمثلة حول روابط المجتمعات: *r/python/* و *r/flask/*. يوجد ميزة مثيرة في هذا الموقع وهي أنه يمكنك عرض منشورات من مجتمعين مختلفين عن طريق الفصل بين أسماء المجتمعات بإشارة مساواة في الرابط، على سبيل المثال: *reddit.com/r/python+flask*.

يمكننا استخدام محول مخصص لإضافة هذه الميزة إلى تطبيقنا. سنأخذ عدداً عشوائياً من العناصر المفصولة بعلامة زائد، وسنقوم بتحويلها إلى قائمة باستخدام الصنف `ListConverter` وسنمرر هذه القائمة إلى دالة العرض.

```python
# myapp/util.py

from werkzeug.routing import BaseConverter

class ListConverter(BaseConverter):

    def to_python(self, value):
        return value.split('+')

    def to_url(self, values):
        return '+'.join(BaseConverter.to_url(value)
                        for value in values)
```

في المثال أعلاه، قمنا بتعريف دالتين `()to_python` و `()to_url`. كما يشير الاسم، تُستخدم الدالة `()to_python` لتحويل الرابط إلى كائن بايثون ليتم تمريره إلى دالة العرض، وتُستخدم الدالة `()to_url` من قبل `()url_for` لتحويل المعاملات إلى رابط.

لاستخدم المحول `ListConverter`، علينا بدايةً إخبار فلاسك أنه موجود.

```python
# /myapp/__init__.py

from flask import Flask

app = Flask(__name__)

from .util import ListConverter

app.url_map.converters['list'] = ListConverter
```

<blockquote>
<b>ملاحظة</b><br/>
هناك احتمالية للوقوع ببعض مشاكل الاستيراد الحلقية إذا احتوى ملف <code>util</code> خاصتك على السطر <code>from . import app</code>. لهذا السبب انتظرت حتى يتم استهلال التطبيق حتى أقوم باستيراد <code>ListConverter</code>.
</blockquote>

اﻵن يمكننا استخدام المحول الذي قمنا بإنشائه بالطريقة نفسها التي نستخدم بها المحولات القياسية (المبنية في الإطار). قمنا بتحديد نوع المفتاح في القاموس على أنه "قائمة" لذلك المثال أدناه يوضح كيف يمكننا استخدام هذا المحول في المُزخرِف `()app.route@`.

```python
# myapp/views.py

from . import app

@app.route('/r/<list:subreddits>')
def subreddit_home(subreddits):
    """Show all of the posts for the given subreddits."""
    posts = []
    for subreddit in subreddits:
        posts.extend(subreddit.posts)

    return render_template('/r/index.html', posts=posts)
```

يجب أن يعمل المثال أعلاه كما في موقع ريديت. يُمكن استخدام نفس الطريقة لصنع أي محول روابط نريده.

## الخلاصة

<ul style='list-style-type: disc; list-style-position: inside;'>
  <li>يساعدنا المُزخرِف <code>login_required@</code> في الإضافة Flask-Login على حصر وصول المستخدمين المسجلين فقط لبعض دوال العرض.</li>
  <li>توفر الإضافة Flask-Cache مجموعة من المُزخرِفات لتنفيذ طرق متنوعة من التخزين المؤقت.</li>
  <li>يُمكننا تطوير مُزخرِفات عرض مُخصصة لتساعدنا على تنظيم الشيفرة البرمجية وللالتزام بمبدأ "<a href='https://ar.wikipedia.org/wiki/%D9%84%D8%A7_%D8%AA%D9%83%D8%B1%D8%B1_%D9%86%D9%81%D8%B3%D9%83'>لا تكرر نفسك</a>".</li>
  <li>يُمكن أن تكون محولات الروابط المخصصة طريقة رائعة لتنفيذ ميزات إبداعية متعلقة بالروابط.</li>
</ul>
